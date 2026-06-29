---
zip: 420
title: Cognitive Consensus — LLM-Verified Block Attestations
description: Validators emit verifiable reasoning traces (CognitiveAttestations) during block production, verified by a tiered scheme that never requires universal LLM re-execution.
author: Pattermesh (KCOLBCHAIN Research)
status: Draft
type: Standards Track
category: AI
created: 2026-06-30
requires: 410, 419
---

## Abstract

Cognitive Consensus extends Proof of AI (ZIP-0419) so that each block carries a verifiable **reasoning trace** produced by a protocol-embedded, frozen Large Language Model (LLM). The block proposer runs deterministic inference over committed network state, emits a structured `CognitiveAttestation`, and commits it alongside the block. Other validators do **not** re-run the LLM by default. Instead, a three-tier verification scheme — structural checks (all validators, O(1)), semantic sampling by a random committee (O(k)), and full deterministic re-execution on dispute (O(n)) — bounds the cost of agreement while preserving slashable accountability. The determinism required for verification is supplied by the Hamiltonian LLM (HLLM) framework: models are frozen and behavior is steered by context injection, so `(model_hash, context_hash, seed) → output` is reproducible across heterogeneous hardware up to a bounded numerical tolerance. This ZIP specifies the attestation data structure, the trace schema, the verification pipeline, the consensus integration points against the Quasar finality driver, and the slashing conditions.

## Motivation

Existing AI-native blockchain work treats the model as the *payload* of the chain: PoAI (ZIP-0419) rewards validators for useful ML work, DSO (ZIP-0410) governs a knowledge commons, and ZIP-0407 coordinates decentralized training. In all of these the consensus state machine remains a conventional deterministic executor; the LLM never participates in producing or validating a block.

Cognitive Consensus closes that gap. Concretely it enables:

1. **In-band protocol reasoning.** Per block, an embedded model performs anomaly detection, mempool prioritization, fee/MEV analysis, and risk flagging, and commits its conclusions as first-class, queryable chain data rather than off-chain operator tooling.
2. **A verifiable substrate for SAIPs (ZIP-0421).** Per-block reasoning traces are the raw telemetry that the Self-Authoring Improvement Proposal pipeline aggregates across epochs.
3. **Compute reuse.** PoAI validators already operate GPUs and already run inference; the marginal cost of emitting a small reasoning trace per block is low (see Rationale).

The central engineering problem is **verification without universal re-execution**. Re-running an LLM at every validator for every block is economically and latency-prohibitive. The contribution of this ZIP is a tiered scheme that makes the *common case* cheap (structural + sampled), keeps the *adversarial case* fully accountable (re-execution + slashing), and relies on HLLM determinism so that "did this validator run the agreed model on the agreed context" is a decidable question.

## Specification

All multi-byte integers are big-endian. All hashes are BLAKE3-256 unless stated otherwise. "Canonical encoding" means RLP over the field order given below. Embeddings reuse the DSO 7680-dimensional space (ZIP-0410).

### 1. Roles and parameters

| Symbol | Meaning | Default |
|--------|---------|---------|
| `M` | Active cognitive model, identified by `model_hash` | governed by ZIP-0410 |
| `S_seed` | Per-block deterministic seed = `H(block_parent_hash ‖ proposer_id ‖ height)` | derived |
| `k` | Semantic-sampling committee size | 8 |
| `τ_sem` | Cosine-similarity acceptance threshold (Tier 2) | 0.92 |
| `τ_struct` | Min calibrated confidence accepted without flag | 0.50 |
| `ε_num` | Allowed per-logit numerical tolerance (cross-hardware) | 2⁻¹⁰ |
| `Δ_cog` | Max wall-clock budget for proposer inference | 250 ms |
| `B_trace` | Max serialized trace size | 16 KiB |

`M` is **frozen**: its parameters are fixed for the epoch and addressed by `model_hash`. Upgrading `M` is a DSO/governance action (ZIP-0410), never an in-place fine-tune. This is what makes inference reproducible (see §6).

### 2. Data structure: `CognitiveAttestation`

A `CognitiveAttestation` is a typed transaction (new tx type `0x07`) committed in the block it describes. Canonical field order:

```
CognitiveAttestation {
    version:        u16              // = 1
    height:         u64              // block height this attests to
    parent_hash:    bytes32          // block_parent_hash
    proposer_id:    bytes20          // validator address
    model_hash:     bytes32          // frozen M identity (ZIP-0410 registry key)
    context_root:   bytes32          // Merkle root over the inference context (§3)
    seed:           bytes32          // S_seed, recomputable by verifiers
    trace_root:     bytes32          // Merkle root over ReasoningTrace nodes (§4)
    trace_blob_cid: bytes            // IPFS/Experience-Ledger CID of full trace
    output_commit:  bytes32          // H(canonical(trace)) — binds blob to root
    embedding:      bytes            // 7680-dim fp16 trace embedding (DSO space)
    confidence:     u16              // calibrated, fixed-point /10000
    flags:          u32              // bitfield: ANOMALY, MEV_RISK, FEE_SPIKE, ...
    runtime_proof:  bytes            // TEE quote OR re-exec receipt (ZIP-0419)
    sig:            bytes65          // proposer signature over all prior fields
}
```

`trace_blob_cid` points at the full human/machine-readable trace; only `trace_root`, `output_commit`, and `embedding` are needed for on-chain verification, keeping per-block state small.

### 3. Inference context and `context_root`

The proposer's inference input MUST be a deterministic function of committed state, so any verifier can reconstruct it bit-for-bit:

```
Context = encode(
    header_summary,        // height, parent_hash, timestamp, gas_used, base_fee
    state_digests,         // {balances_root, mempool_root@cutoff, fee_history[N]}
    experience_refs,       // ordered list of Experience-Ledger CIDs (ZIP-0410)
    prompt_template_hash   // governed prompt program, frozen per epoch
)
context_root = MerkleRoot(Context fields)
```

`experience_refs` MUST be content-addressed and resolvable from the Experience Ledger. The `prompt_template_hash` binds the exact instruction program; changing it is a governance action. No wall-clock, no local entropy, no node-local config may enter `Context`.

### 4. `ReasoningTrace` schema

The trace is a directed acyclic graph of typed nodes. Tier-1 structural verification (§5.1) checks conformance to this schema:

```
ReasoningTrace {
    nodes: [ Node ]            // topologically ordered
}
Node {
    id:          u32
    kind:        enum { PREMISE, EVIDENCE, INFERENCE, CONCLUSION }
    parents:     [u32]         // must reference lower ids (DAG, acyclic)
    statement:   string        // bounded length
    evidence_ref: optional {   // required when kind == EVIDENCE
        path:  StateMerklePath // must verify against header state roots
        value: bytes
    }
    confidence:  u16           // /10000
}
```

Well-formedness rules (all enforced in Tier 1):

- Exactly ≥1 `PREMISE` with no parents and ≥1 `CONCLUSION` that transitively descends from premises.
- Every `EVIDENCE` node MUST carry an `evidence_ref` whose `StateMerklePath` verifies against the block's committed state roots. **Fabricated evidence is detectable in O(1) without the LLM.**
- DAG acyclicity; parent ids strictly less than child id.
- Per-node `confidence` within `[0,10000]`; root `confidence` equals the attestation `confidence`.

### 5. Verification pipeline

#### 5.1 Tier 1 — Structural (every validator, O(1) per block)

Performed during normal block validation:

1. `seed == H(parent_hash ‖ proposer_id ‖ height)`.
2. `output_commit == H(canonical(trace))` after fetching `trace_blob_cid` (or accept on `trace_root` Merkle proofs for fields needed; full blob fetch may be lazy).
3. `ReasoningTrace` is schema-valid (§4) and every `EVIDENCE.evidence_ref` Merkle-verifies against committed state roots.
4. `confidence` is within historical calibration bounds for `proposer_id` (a rolling Brier-score gate, §7); attestations with confidence above the proposer's demonstrated calibration are flagged, not rejected.
5. `model_hash` is the epoch's governance-active model.

Tier-1 failure ⇒ block is **invalid** (not merely the attestation). This makes a malformed or evidence-fabricating attestation a consensus fault, not a soft signal.

#### 5.2 Tier 2 — Semantic sampling (committee of `k`, O(k))

For each finalized block, a committee of `k` validators is selected by verifiable random function seeded on `S_seed` (reusing ZIP-0419's VRF committee selection). Each committee member:

1. Reconstructs `Context` from `context_root` (deterministic, §3).
2. Runs `M` locally to produce `trace'` and embedding `e'`.
3. Computes `cos(embedding, e')`.
4. **Accepts** iff `cos ≥ τ_sem`; otherwise emits a `Challenge` (§5.3).

Committee verdicts are aggregated; ≥⌈k/2⌉ accepts finalizes the attestation as *semantically verified*. Because committees are small and random, expected verification cost per block is `O(k)` inferences network-wide, independent of validator count.

Embeddings (not raw text) are compared so that benign paraphrase does not trigger false challenges, while substantive divergence (different conclusion, different flagged risk) pushes cosine below `τ_sem`. `τ_sem` is governance-tunable and SHOULD be calibrated from the false-challenge / missed-divergence ROC on historical traces.

#### 5.3 Tier 3 — Full re-execution (on dispute only, O(n) worst case)

A `Challenge{height, challenger_id, claimed_divergence}` opens an adjudication window. Resolution is **deterministic re-execution**:

1. All adjudicating validators (or a TEE-attested subset per ZIP-0419) reconstruct `Context` and run `M` with `seed`, greedy/temperature-0 decoding, fixed kernel/quantization profile pinned by `model_hash`.
2. Outputs are compared per-logit within `ε_num`; the canonical trace is the deterministic decode.
3. **Adjudication outcomes:**
   - Proposer's `output_commit` matches the deterministic re-decode ⇒ challenger is slashed for a false challenge.
   - Proposer's commit diverges beyond `ε_num` ⇒ proposer is slashed for an invalid attestation; the block's cognitive layer is reverted and re-attested by the next proposer (transaction state is unaffected — see Backwards Compatibility).

### 6. Why determinism holds (HLLM dependency)

Verification reduces to "did the proposer run the agreed function?" That is only decidable if inference is reproducible. The HLLM construction provides this:

- **Frozen weights.** `model_hash` pins parameters; no per-validator fine-tuning drift.
- **Context-injected behavior.** All steering is via `Context` (committed, §3), not weight updates.
- **Pinned numerics.** `model_hash` additionally pins quantization, kernel profile, and decode policy (temperature 0, fixed seed) for Tier-3. Tier-2 tolerates cross-hardware drift by comparing embeddings, not logits; Tier-3 tightens to `ε_num` under a pinned profile.

A conventional continuously-fine-tuned model would make this intractable (each validator diverges); HLLM makes reproducibility a property of the system rather than an aspiration.

### 7. Calibration accounting

Each `proposer_id` carries an on-chain rolling **Brier score** over the last `W` (default 4096) attestations whose flagged predictions later resolved (e.g., a flagged anomaly that did/did not lead to a slashing event or reorg). Confidence claims exceeding demonstrated calibration are down-weighted in reward (ZIP-0422 settlement) and surfaced as a Tier-1 flag. This disincentivizes confidently-wrong attestations without requiring the chain to adjudicate "truth" in real time.

### 8. Consensus integration (Quasar)

Cognitive Consensus is a **pluggable attestation sub-protocol**, not a replacement finality rule. It hooks Quasar at:

- **Block proposal:** proposer runs §3–§4 inference within `Δ_cog` and includes the `CognitiveAttestation` tx. Missing/late attestation ⇒ block proposed without cognitive layer (allowed during bootstrap, see Backwards Compatibility), but forfeits cognitive reward.
- **Pre-vote:** validators run Tier-1 (§5.1) as part of block validity.
- **Post-finality:** committee runs Tier-2 (§5.2) asynchronously; challenges (Tier-3) resolve within the challenge window before cognitive reward is released.

Transaction-state finality is **never gated** on Tier-2/Tier-3. Only the *cognitive reward* and *SAIP eligibility* of the attestation depend on semantic verification, so a dispute cannot stall the chain.

## Rationale

**Why tiered rather than uniform re-execution?** A single Zen-Eco-4B forward pass over a bounded context is on the order of ~100 ms on validator-class GPUs and produces a small trace. Requiring all `n` validators to re-run it per block is `O(n)` inferences/block — unacceptable at scale and on the critical finality path. Tiering pushes the expensive operation off the hot path: O(1) structural checks gate validity, an O(k) random committee gives probabilistic semantic assurance, and O(n) re-execution is reserved for the rare disputed case where slashing pays for the cost.

**Why embeddings in Tier 2?** Text-equality verification would punish benign nondeterminism and paraphrase; logit-equality is infeasible cross-hardware without a pinned profile. The DSO 7680-dim space (ZIP-0410) already exists and already supports semantic gradients, so reusing it for trace comparison adds no new infrastructure and lets `τ_sem` be tuned against real ROC data.

**Why make evidence Merkle-verifiable?** The most dangerous failure is a fluent but fabricated trace. By requiring every `EVIDENCE` node to carry a Merkle path into committed state, the cheapest tier (Tier 1) already catches the cheapest attack (hallucinated facts) deterministically, without any model.

**Why not gate finality on reasoning?** Liveness must not depend on a probabilistic semantic check. Decoupling state-finality from cognitive-verification means the worst case for a model outage or dispute is "blocks finalize without cognitive rewards," never "chain halts."

**Why a separate, small model?** Open Question 1 of the source draft. This ZIP recommends a *dedicated* small reasoning model (Zen-Eco-4B / Zen-Mini-8B) distinct from any model the network trains, to avoid a circular dependency where consensus correctness depends on a model the same consensus is mid-training. Model identity is governed by ZIP-0410.

## Backwards Compatibility

- **New tx type `0x07`** is additive. Nodes that do not understand it treat `CognitiveAttestation` as an opaque typed tx and still reach state consensus; only cognitive reward accounting (ZIP-0422) requires the new logic. This permits a **soft-launch**: a flag-day epoch where attestations are emitted and verified but carry no reward or slashing, used to gather `τ_sem` calibration data.
- **Bootstrap mode.** Before the Experience Ledger is seeded (Open Question 5), proposers MAY emit attestations with an empty `experience_refs` set; Tier-2 still applies. Cognitive reward weight ramps from 0 across the bootstrap epochs.
- **No change to transaction execution semantics.** EVM/state transition is untouched; the cognitive layer is strictly side-car data plus a reward/slashing overlay.
- **Quasar:** integration is via existing pluggable sub-protocol hooks; chains can disable Cognitive Consensus per-subnet without a hard fork.

## Security Considerations

1. **Fabricated evidence:** mitigated at Tier 1 — `EVIDENCE` nodes must Merkle-verify against committed state; fabrication is an O(1) detectable block-validity fault.
2. **Plausible-but-wrong reasoning (adversarial fluency):** mitigated by Tier-2 semantic sampling plus calibration accounting (§7); a proposer that is confidently wrong loses reward and accrues Brier penalty. SAIP-level adversarial review (ZIP-0421) provides a second line for proposals derived from traces.
3. **Committee bribery / collusion (Tier 2):** committee is VRF-selected per block (unpredictable), small, and any single honest member can open a Tier-3 challenge that is adjudicated by deterministic re-execution — so corrupting a Tier-2 committee does not finalize a bad attestation, it only delays its detection until any honest party challenges.
4. **Grinding the seed:** `S_seed` derives from `parent_hash ‖ proposer_id ‖ height`; the proposer cannot grind committee selection without grinding the parent block, which is already cost-bounded by the base consensus.
5. **Nondeterminism / hardware divergence:** Tier-2 tolerates it via embeddings; Tier-3 eliminates it via a pinned numeric profile (`model_hash`-bound quantization/kernel/decode). If a hardware class cannot meet `ε_num` under the pinned profile, it is ineligible to adjudicate (but may still propose).
6. **Trace privacy / MEV leakage (Open Question 4):** reasoning may reveal MEV or strategy. Sensitive flags MAY be committed as `output_commit` only, with the blob FHE-encrypted and time-delayed for decryption, reusing the ZIP-0419 TEE/threshold framework. On-chain verification still proceeds against `trace_root` and `embedding` (which can themselves be computed inside the TEE).
7. **Model-swap / governance capture:** `model_hash` is governance-pinned via ZIP-0410; an attestation referencing a non-active model fails Tier 1. Capture of the model registry is the governance threat boundary, out of scope here and inherited from ZIP-0410.
8. **DoS via oversized traces:** `B_trace` caps serialized size; over-cap traces fail Tier 1. `Δ_cog` caps proposer inference time.
9. **Liveness:** by construction (§8) state-finality never depends on cognitive verification, so neither a model outage nor a flood of challenges can halt the chain — they only suspend cognitive rewards.

## Reference Implementation (sketch)

```python
# Proposer side (block production)
def produce_cognitive_attestation(block, state, M, ledger) -> CognitiveAttestation:
    ctx = build_context(block.header, state, ledger.refs_for(block.height))
    seed = blake3(block.parent_hash + proposer_id + u64(block.height))
    trace = M.infer(ctx, seed=seed, temperature=0.0, deadline_ms=DELTA_COG)  # frozen M
    assert serialized_len(trace) <= B_TRACE
    cid = ledger.put(canonical(trace))            # full blob to Experience Ledger
    emb = M.embed_trace(trace)                    # 7680-dim, DSO space
    att = CognitiveAttestation(
        version=1, height=block.height, parent_hash=block.parent_hash,
        proposer_id=proposer_id, model_hash=M.hash, context_root=merkle(ctx),
        seed=seed, trace_root=merkle(trace.nodes), trace_blob_cid=cid,
        output_commit=blake3(canonical(trace)), embedding=fp16(emb),
        confidence=trace.root_confidence, flags=trace.flags,
        runtime_proof=tee_quote_or_receipt(),     # ZIP-0419
    )
    att.sig = sign(proposer_key, att.signing_bytes())
    return att

# Tier 1 — every validator, O(1)
def verify_structural(att, block, state) -> bool:
    if att.seed != blake3(att.parent_hash + att.proposer_id + u64(att.height)):
        return False
    trace = fetch(att.trace_blob_cid)
    if blake3(canonical(trace)) != att.output_commit: return False
    if not schema_valid(trace): return False
    for n in trace.nodes:
        if n.kind == EVIDENCE and not merkle_verify(n.evidence_ref.path,
                                                    block.state_roots): 
            return False                          # fabricated evidence -> invalid block
    if not within_calibration(att.proposer_id, att.confidence): att.flags |= F_LOWCAL
    return att.model_hash == active_model(epoch_of(att.height))

# Tier 2 — VRF committee of k, O(k) network-wide
def verify_semantic(att) -> Verdict:
    ctx = reconstruct_context(att.context_root)   # deterministic
    trace2 = active_model().infer(ctx, seed=att.seed, temperature=0.0)
    if cosine(att.embedding, embed(trace2)) >= TAU_SEM:
        return ACCEPT
    return challenge(att.height, claimed_divergence=trace2)

# Tier 3 — deterministic adjudication on challenge, O(n) worst case
def adjudicate(challenge) -> Slash:
    ctx = reconstruct_context(challenge.context_root)
    canonical_trace = pinned_profile_infer(ctx, seed=challenge.seed)  # eps_num bound
    if blake3(canonical(canonical_trace)) == challenge.proposer_commit:
        return slash(challenge.challenger_id, FALSE_CHALLENGE)
    return slash(challenge.proposer_id, INVALID_ATTESTATION)
```

Components reused unchanged: ZIP-0419 VRF committee selection + TEE quotes; ZIP-0410 7680-dim embedding service and model registry; Experience Ledger blob store; Quasar pluggable sub-protocol hooks.

## Copyright

Copyright and related rights waived via CC0.
