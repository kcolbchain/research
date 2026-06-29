---
zip: 422
title: Compute Settlement Layers — Triple-Use GPU Economics
description: A per-node GPU scheduler and atomic on-chain settlement that lets the same GPU serve consensus, decentralized training, and inference-as-a-service, amortizing training cost across all three into a self-funding compute cycle.
author: Pattermesh (KCOLBCHAIN Research)
status: Draft
type: Standards Track
category: AI
created: 2026-06-30
requires: 407, 419, 420
---

## Abstract

Compute Settlement Layers unify three GPU functions that today are operated as separate economic activities — **consensus** (securing the chain, via Cognitive Consensus, ZIP-0420), **training** (improving the chain, via decentralized training, ZIP-0407), and **inference-as-a-service** (generating external revenue) — into a single per-node primitive. A priority-preemptive GPU scheduler arbitrates a node's device time across the three duties under hard economic invariants, and a single atomic on-chain `SettlementReceipt` per epoch per node combines block reward, training reward, and inference fees into one transaction. The key economic claim is that **training cost is amortized across all three functions**: external inference revenue funds the GPU, the GPU's idle cycles produce training gradients that improve the served models, and the improved models raise inference quality and demand — turning the security budget from pure waste (PoW) or pure capital lockup (PoS) into a productive, self-funding R&D budget. This ZIP specifies the scheduler, the duty-cycle and accounting data structures, the atomic settlement, the conservation invariants, and the anti-centralization mechanism.

## Motivation

A GPU node in existing designs does one thing at a time: mine, stake, or serve inference. PoAI (ZIP-0419) already redirects security compute toward useful ML, but still treats consensus, training, and inference as distinct activities with distinct accounting. The consequences:

- **Idle waste.** Consensus inference (ZIP-0420) is bursty (~100–250 ms per block); between proposals/committee duties the GPU is underutilized.
- **Training as a cost center.** ZIP-0407 training is funded as if it were overhead, rather than as an investment that improves a revenue-producing product (inference).
- **Settlement overhead.** Three reward streams mean up to three on-chain settlement paths per node, with no atomicity guarantee across them.

Compute Settlement Layers address all three by (1) scheduling the three duties on one device with consensus-preemptive priority, (2) defining hard invariants so no duty starves another and no node operates at a loss, and (3) settling all three streams atomically once per epoch. The result is the "self-funding compute cycle" described in the source draft, made concrete enough to implement.

## Specification

All amounts are in ZOO base units. An *epoch* is the settlement period (a fixed number of blocks, governance-tunable). A *node* is a single settlement identity bound to one or more GPUs under one operator key.

### 1. Duties and the slot model

Device time within an epoch is partitioned into **slots**. Three duty types compete for slots:

| Duty | Source ZIP | Earns | Typical cost |
|------|-----------|-------|--------------|
| **CONSENSUS** | ZIP-0420 | block reward + tx fees + cognitive reward | ~100–250 ms / proposal or committee call |
| **TRAINING** | ZIP-0407 | training reward (quality-gated; up to 3× per ZIP-0419) | continuous, preemptible |
| **INFERENCE** | this ZIP | per-request fee, settled on-chain | on-demand, preemptible |

### 2. The GPU scheduler (per-node daemon)

A node runs a local scheduler with a **strict priority preemptive** discipline:

```
priority(CONSENSUS) > priority(TRAINING) > priority(INFERENCE)
```

- **Consensus always wins.** A consensus obligation (this node is the slot proposer per ZIP-0420, or a VRF-selected Tier-2 committee member) **preempts** any running training or inference job immediately. Preempted training checkpoints to the gradient buffer; preempted inference requests are re-queued or failed-over per §5.
- **Training is the default background duty.** When no consensus obligation is pending, the scheduler runs training unless an inference request is enqueued *and* the node's training duty-cycle floor for the epoch is already met (invariant I2).
- **Inference fills the rest** and is the only externally-triggered, latency-sensitive duty; the scheduler may reserve a bounded inference lane (charter param) so paid requests are not perpetually starved by training.

Scheduler state, per epoch:

```
NodeDutyLedger {
    node_id:          bytes20
    epoch:            u64
    consensus_us:     u64        // microseconds spent on consensus duties
    training_us:      u64
    inference_us:     u64
    consensus_events: { proposals, committee_calls, challenges_served }
    training_output:  { gradient_submissions:[GradId], accepted, quality_sum }
    inference_output: { requests_served, fee_accrued }
    preemptions:      { training_preempted, inference_preempted_or_failed }
    duty_root:        bytes32     // Merkle root of the above, signed by node
}
```

The scheduler is **node-local and untrusted**; correctness is enforced not by trusting `NodeDutyLedger` but by the verifiable artifacts each duty already produces (signed proposals/attestations for consensus, quality-gated accepted gradients for training, signed request receipts for inference). Settlement (§4) pays only for *verified* output, so a lying duty ledger cannot inflate rewards.

### 3. Reward components

Per node per epoch, three independently-verifiable components are computed:

1. **Consensus reward** `R_c` — from ZIP-0420 finalized attestations and base block/fee rewards attributable to this node, scaled by the node's calibration standing (ZIP-0420 §7). Disputed attestations under challenge are escrowed, not paid, until the challenge window closes.
2. **Training reward** `R_t` — from ZIP-0407 gradient submissions that pass PoAI quality scoring (ZIP-0419), with the up-to-3× multiplier applied to *accepted* gradients only. Rejected gradients earn nothing (invariant I4).
3. **Inference fee** `R_i` — sum of per-request fees for requests with a valid settlement receipt (§5).

### 4. Atomic settlement: `SettlementReceipt`

Exactly **one** settlement transaction per node per epoch:

```
SettlementReceipt {
    node_id:        bytes20
    epoch:          u64
    duty_root:      bytes32           // commits the NodeDutyLedger
    consensus:      { reward: u256, attestation_refs:[bytes32] }   // ZIP-0420
    training:       { reward: u256, accepted_grad_refs:[GradId] }  // ZIP-0407/0419
    inference:      { fee: u256, request_merkle_root: bytes32 }    // §5
    operating_cost: u256              // self-reported, used for I3 throttle only
    net:            u256              // = consensus.reward + training.reward + inference.fee
    sig:            bytes65
}
```

Settlement is **atomic**: the three components credit (or escrow) together in one state transition. A receipt that references unverifiable attestations, unaccepted gradients, or unsigned inference requests is rejected wholesale — there is no partial payout from a malformed receipt. This collapses up-to-three settlement paths into one and guarantees cross-component atomicity.

### 5. Inference-as-a-service settlement path

External inference is metered and settled trustlessly:

1. A client opens a **payment channel** funded in ZOO (or pays per-request via a prepaid voucher).
2. Each served request returns a signed `RequestReceipt{ request_hash, model_hash, node_id, units, client_sig }`. `model_hash` ties the served model to the ZIP-0410 registry, so clients verify they received the model they paid for.
3. The node accumulates receipts into `request_merkle_root`; at settlement, fees redeem against channel balances. Disputed receipts are resolved by the channel's adjudication (standard L2 channel mechanics, ZIP-0015 settlement layer).
4. Preempted inference (because consensus won the GPU) either fails over to a peer node in the same service set or returns a no-charge error; a preempted request MUST NOT be billed.

### 6. Economic invariants (enforced at settlement)

| # | Invariant | Enforcement |
|---|-----------|-------------|
| **I1** | **Training never starves consensus.** | Scheduler preemption (§2); additionally, a node that *missed* an assigned consensus duty because it was training is penalized (forfeits the epoch's training multiplier), making the preemption economically self-enforcing. |
| **I2** | **Inference never starves training.** | A minimum training duty-cycle floor per epoch (charter default 20%); below the floor, the node's training reward and inference reward for the epoch are both reduced pro-rata. Verified via accepted-gradient evidence, not the self-reported `training_us`. |
| **I3** | **Revenue covers compute.** | If realized `R_i` over a trailing window < self-reported `operating_cost`, the node MAY auto-reduce its training duty toward the I2 floor to stay solvent. This is an operator-side throttle, advisory at the protocol level. |
| **I4** | **Quality gates training pay.** | Only gradients passing PoAI quality scoring (ZIP-0419) earn `R_t`; the 3× multiplier applies to accepted gradients only. |
| **I5** | **Settlement conservation.** | For every epoch, `Σ_nodes net == Σ block_rewards_minted + Σ training_budget_allocated + Σ inference_fees_redeemed`. No receipt can create value; over-claim ⇒ receipt rejected. This is the hard invariant the ZIP-0421 simulation gate checks. |
| **I6** | **Atomicity.** | All three components settle in one transaction (§4); no partial settlement. |

### 7. Anti-centralization (Open Question 3)

To prevent a large operator from monopolizing all three slots across many GPUs and centralizing the network, settlement applies **per-operator diminishing returns**:

- Define an operator's effective reward weight as a concave function of its total verified contribution: `w(x) = x^α`, `0 < α < 1` (charter default `α = 0.8`), applied per-operator across consensus, training, and inference reward components.
- Consensus committee selection (ZIP-0420 VRF) and proposer scheduling already cap per-validator block share; this ZIP extends the same diminishing curve to *training and inference* rewards so that stacking GPUs under one identity yields sub-linear total reward.
- Sybil resistance against splitting one operator into many identities is inherited from the base staking/identity layer (a node identity requires bonded stake); `α` is applied per bonded identity, and stake-bonding cost makes fragmentation non-free.

### 8. The self-funding cycle (made precise)

The amortization claim is the accounting statement that the GPU's fixed operating cost `C` is covered primarily by inference revenue `R_i`, while training is run on cycles that would otherwise be idle:

```
marginal training cost ≈ C · (training_us / epoch_us) − (idle_us amortized)
```

Because training preempts to inference/consensus and runs in otherwise-idle device time (§2), its *marginal* cost approaches zero whenever `R_i` already covers `C`. Training then improves the served models (ZIP-0407 → better `model_hash` in the ZIP-0410 registry), which raises inference quality and demand, raising `R_i`, funding more GPUs and more training capacity. The security budget thus does work three times over — the cycle the source draft describes, expressed as an invariant set (I1–I6) rather than a narrative.

## Rationale

**Why strict priority instead of weighted fair-share?** Consensus has a hard liveness deadline (the block/committee slot); training and inference do not. Strict consensus-preemptive priority is the simplest discipline that guarantees I1, and pairing it with an economic penalty for missed consensus duty makes the priority self-enforcing rather than purely trust-based.

**Why atomic single-receipt settlement?** Three separate settlement paths admit partial-failure states (e.g., training pays but inference channel dispute strands fees) and triple the on-chain footprint. One atomic receipt per node per epoch gives cross-component atomicity (I6), enables the global conservation check (I5) in one place, and minimizes state growth.

**Why pay only for verified output?** The per-node scheduler is untrusted; `NodeDutyLedger` is a hint, not a payment basis. Anchoring every reward component to an artifact that some other party already verifies (finalized attestations, quality-gated gradients, client-signed inference receipts) means a node cannot profit by lying about how it spent its microseconds.

**Why diminishing returns rather than hard caps?** A hard cap on GPUs-per-operator is trivially evaded by splitting identities and is brittle. A concave reward curve (`α < 1`) makes monopolization economically self-limiting while still rewarding genuine scale, and it composes uniformly across all three reward streams (Open Question 3).

**Why is I3 advisory, not enforced?** Solvency is an operator concern; the protocol should not refuse to pay a node that chooses to run training at a loss. I3 gives operators a safe default throttle without the protocol policing private economics.

## Backwards Compatibility

- The scheduler is **off-chain, node-local**; adopting it requires no consensus change. Existing PoAI validators already produce the consensus and training artifacts this ZIP settles against.
- `SettlementReceipt` **supersedes** the separate per-stream settlement paths but can run in parallel during migration: a flag-day epoch switches reward accrual from N receipts to 1 atomic receipt. Until cut-over, the conservation invariant I5 is checked per-stream.
- Networks not running Cognitive Consensus (ZIP-0420) can adopt Compute Settlement with `R_c` limited to base block/fee rewards (no cognitive component), so this ZIP degrades gracefully on chains that adopt training+inference settlement without the cognitive layer.
- The diminishing-returns curve (§7) ships with `α = 1` (linear, i.e., no change) and is moved to `α < 1` only by governance, so existing reward expectations are not retroactively altered without a vote.
- Inference payment channels reuse the existing ZIP-0015 L2 settlement layer; no new channel primitive is introduced.

## Security Considerations

1. **Duty-ledger forgery:** mitigated by paying only for externally-verified artifacts (§2, §6). A node inflating `training_us` or `inference_us` earns nothing extra because rewards bind to accepted gradients / signed receipts, not self-reported time.
2. **Consensus starvation by greedy training (I1):** scheduler preemption plus forfeiture of the training multiplier on any missed consensus duty makes starving consensus strictly unprofitable.
3. **Inference starvation (I2):** the training duty-cycle floor, verified via accepted-gradient evidence, prevents a node from claiming training rewards while actually only serving inference, and vice-versa via the reserved inference lane.
4. **Reward over-claim / value creation (I5):** the global conservation check rejects any receipt set whose `net` exceeds minted + allocated + redeemed value; this is the hard invariant the SAIP simulation gate (ZIP-0421 §6.2) also enforces against proposed parameter changes.
5. **Settlement non-atomicity / griefing:** I6 makes settlement all-or-nothing; a malformed component cannot strand a valid one. Disputed consensus attestations are escrowed (not lost, not prematurely paid) until the ZIP-0420 challenge window closes.
6. **Centralization (Open Question 3):** §7 concave reward curve + per-identity bonded stake + ZIP-0420 VRF committee caps jointly bound any single operator's share sub-linearly; identity-splitting is taxed by bonding cost.
7. **Inference billing abuse:** preempted/failed requests MUST NOT be billed (§5.4); clients sign receipts so a node cannot fabricate served requests; `model_hash` in the receipt prevents bait-and-switch on which model was served.
8. **Quality-gate gaming:** the 3× multiplier applies only to PoAI-accepted gradients (I4, ZIP-0419), so submitting low-quality or replayed gradients earns nothing; gradient ids are content-addressed to prevent replay.
9. **MEV via slot scheduling:** a node could in principle time inference/training to influence its own block proposals; because consensus duty is VRF-assigned and proposer order is not node-chosen, scheduling cannot manufacture additional proposal opportunities — only fulfill or forfeit assigned ones.

## Reference Implementation (sketch)

```python
# Per-node scheduler (off-chain daemon), strict consensus-preemptive priority
class Scheduler:
    def loop(self):
        while True:
            if self.consensus_obligation_pending():      # ZIP-0420 proposer/committee
                self.preempt_running()                    # checkpoint training / requeue inf.
                self.run_consensus()                      # always wins -> I1
            elif self.inference_pending() and self.training_floor_met():
                self.run_inference_request()              # billed via RequestReceipt
            else:
                self.run_training_step()                  # default background -> I2 floor

# On-chain settlement: one atomic receipt per node per epoch
def settle(node_id, epoch, duty, charter) -> SettlementReceipt:
    R_c = consensus_reward(node_id, epoch)                # finalized, escrow disputed (0420)
    R_t = sum(g.reward for g in duty.accepted_gradients)  # quality-gated (0407/0419), I4
    R_i = sum(r.fee for r in duty.signed_requests)        # channel-redeemable, I3 throttle
    # per-operator diminishing returns (I7), concave alpha<1
    R_c, R_t, R_i = apply_concave(operator_of(node_id), R_c, R_t, R_i, charter.alpha)
    rc = SettlementReceipt(node_id, epoch, duty.duty_root,
                           consensus={'reward':R_c, 'attestation_refs':duty.att_refs},
                           training={'reward':R_t, 'accepted_grad_refs':duty.grad_ids},
                           inference={'fee':R_i, 'request_merkle_root':duty.req_root},
                           operating_cost=duty.cost, net=R_c+R_t+R_i, sig=sign(...))
    assert conservation_holds(epoch)                      # I5, global
    return atomic_credit(rc)                              # I6, all-or-nothing
```

Components reused: ZIP-0420 finalized attestations + calibration standing + challenge escrow; ZIP-0407 gradient submission + ZIP-0419 quality scoring and 3× multiplier; ZIP-0410 model registry (`model_hash`); ZIP-0015 L2 settlement / payment channels.

## Copyright

Copyright and related rights waived via CC0.
