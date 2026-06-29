# Thinking Chains — Prototype Specifications (Delta Components)

**Companion to:** `thinking-chains-draft.md`, `03-red-team.md`
**Scope:** Reference-implementation skeletons for the four *new* components (the "delta" in draft §5.2). These are **consensus-layer systems** — interface-level pseudocode, data structures, and state-transition rules. Not full implementations; enough to build, test, and measure.
**Design constraint:** Written to the **safe subset** from `03-red-team.md §6`. Anything that the red team found consensus-unsafe is present but **gated behind an explicit advisory/non-binding flag** (`BINDING=false` default) so the system can run and be measured without putting finality at risk. Promotions to binding require the listed preconditions to be met.
**Target stack:** Lux Quasar (pluggable finality drivers), Zoo L2 EVM (precompiles + state), PoAI TEE attestation, DSO 7680-dim embeddings, Zen-Eco-4B / Zen-Nano-0.6B inference. Language conventions: Rust-flavored pseudocode for node/consensus code, Solidity-flavored for on-chain contracts, JSON Schema for wire formats.
**Date:** 2026-06-30 · **Status:** DRAFT spec

---

## 0. Component map and delta

| # | Component (draft §5.2) | This doc | Binding default | Precondition to make binding |
|---|---|---|---|---|
| C1 | `CognitiveAttestation` tx type | §1 | `false` (advisory) | Decidable action-oracle (RT-1) + determinism solved |
| C2 | Reasoning-trace schema + 3-tier verification | §2 | Tier-1 binding; Tier-2/3 advisory | Canonical integer-kernel inference |
| C3 | SAIP generation pipeline | §3 | n/a (output is a draft, never auto-applied) | Named human sponsor signs |
| C4 | Triple-use GPU scheduler | §4 | scheduling is local; settlement binding | per-slot commitment scheme live |

Shared types are defined once in §5 (Common Types) and referenced throughout.

---

## 1. C1 — `CognitiveAttestation` transaction type

### 1.1 Purpose and safety posture

A `CognitiveAttestation` (CA) carries a validator's reasoning trace for a block. **Per `03-red-team.md`, the prose/coherence of a CA is NOT a consensus predicate.** What *is* checkable — and the only thing that can ever be binding — are the **assertions**: structured, decidable claims the trace commits to (e.g., "ordered tx A before tx B", "flagged address 0x… as anomalous"). Assertions are verified against the *actual produced block*, which is a decidable fact. The free-text reasoning is stored as advisory commentary and as the input to SAIP telemetry.

### 1.2 EVM transaction encoding

New typed-transaction envelope (EIP-2718 style), `TxType = 0x54` ("T" for Thinking).

```
CognitiveAttestation := RLP([
  chain_id:        uint256,
  block_ref:       bytes32,      // hash of the block this attestation is about
  proposer:        address,      // validator key that produced it
  model_id:        ModelId,      // governance-pinned model (see §5.1) — NOT a freely chosen model
  context_root:    bytes32,      // Merkle root of the exact context fed to the model (§5.2)
  trace_commit:    bytes32,      // commitment to the full trace blob (§2.1); blob stored off-call-data
  trace_ptr:       BlobPointer,  // EIP-4844 blob ref or DSO content-id where full trace lives
  assertions:      Assertion[],  // the ONLY consensus-relevant payload (§1.3)
  trace_embedding: bytes,        // 7680 * fp16 = 15360 bytes, DSO-space embedding of the trace
  confidence:      uint16,       // calibrated confidence, basis points (see §2.2)
  binding_flag:    bool,         // MUST be false unless governance has flipped the binding switch
  sig:             Signature
])
```

`assertions`, `trace_embedding`, and `confidence` go in calldata (cheap, fixed-size). The full reasoning prose goes in a blob / DSO object referenced by `trace_ptr` and committed by `trace_commit` — keeps calldata bounded and defeats the CC-5 "context bomb" from inflating calldata gas.

### 1.3 Assertion — the decidable payload

```rust
enum AssertionKind {
    Ordering    { tx_a: H256, tx_b: H256 },          // claims a executed before b
    Flag        { subject: Address, label: FlagLabel }, // claims subject is anomalous/sanctioned/etc
    FeeBound    { tx: H256, min_fee: U256 },          // claims tx paid >= min_fee
    Inclusion   { tx: H256, included: bool },         // claims tx is/ isn't in block
    MempoolRank { tx: H256, rank: u32 },              // claims relative priority assigned
}

struct Assertion {
    kind:     AssertionKind,
    // each assertion is checkable against the produced block in O(1)..O(log n)
    // with NO model re-execution. This is what makes it consensus-safe.
}
```

`verify_assertion(a, block) -> bool` is a pure function over committed block state. No GPU, no embeddings, no committee. This is the firewall the red team demanded between "decidable" and "vibes".

### 1.4 Precompile / state interface

```solidity
// Precompile at 0x0...CA1 — verifies a CA's *structural* validity (Tier-1) cheaply on-chain.
// Does NOT verify reasoning. Returns the count of assertions that are *self-consistent*
// (well-formed); cross-checking against block is done by the consensus driver, not here.
interface ICognitiveAttestationPrecompile {
    function verifyStructure(bytes calldata ca) external view returns (
        bool   schemaOk,
        bytes32 contextRoot,
        bytes32 traceCommit,
        uint16  assertionCount
    );
}

// State: a ring buffer keyed by epoch, NOT permanent storage (traces are bulky).
mapping(uint64 epoch => mapping(address proposer => CARecord)) caByEpoch;

struct CARecord {
    bytes32 traceCommit;
    bytes   embedding;       // retained for Tier-2 sampling + SAIP telemetry
    uint16  confidence;
    uint8   assertionsTotal;
    uint8   assertionsVerified;   // filled by consensus driver post-block
    bool    binding;
    Verdict verdict;              // Pending | Coherent | Divergent | Disputed
}
```

### 1.5 Consensus-driver hook (Quasar pluggable finality)

```rust
// Runs inside the Quasar finality driver. The CA is attached to a proposed block but
// CANNOT block finality unless binding_flag && governance.binding_enabled.
impl FinalityDriver for CognitiveDriver {
    fn on_block_proposed(&mut self, blk: &Block, ca: &CognitiveAttestation) -> DriverOutcome {
        // Tier 1 (all validators, O(1)) — ALWAYS RUN, may be binding:
        if !structural_ok(ca) { return reject_if_binding(ca, Reason::Schema); }
        if ca.model_id != self.governance_pinned_model { return reject_if_binding(ca, Reason::Model); }
        if !context_root_matches(ca.context_root, blk) { return reject_if_binding(ca, Reason::Context); }

        // Assertion check (O(n_assertions), decidable, NO model) — may be binding:
        let verified = ca.assertions.iter().filter(|a| verify_assertion(a, blk)).count();
        if ca.binding_flag && verified < ca.assertions.len() {
            // a binding CA that asserted a false decidable claim => slashable, deterministic
            return DriverOutcome::Slash(ca.proposer, Offense::FalseAssertion);
        }

        // Tier 2 (committee, O(n)) — ADVISORY ONLY (red-team CC-2/CC-3): never blocks finality.
        self.schedule_committee_sample(ca);   // result lands async, used for telemetry + reputation

        // Record + accept. Finality proceeds on base-layer validity regardless of trace prose.
        self.record(ca, Verdict::Pending);
        DriverOutcome::Accept
    }
}
```

Key safety property: **finality depends only on base-layer validity + (optionally) decidable assertion truth.** Reasoning prose and embedding similarity never gate finality. This is the architectural fix for CC-1/CC-2/CC-3.

---

## 2. C2 — Reasoning-trace schema + 3-tier verification

### 2.1 Trace blob schema (off-calldata, committed by `trace_commit`)

```jsonc
// JSON Schema (draft 2020-12), canonicalized (RFC 8785 JCS) before hashing -> trace_commit
{
  "$id": "https://kcolbchain/schemas/reasoning-trace/v1",
  "type": "object",
  "required": ["schema_version","model_id","context_root","steps","assertions","confidence"],
  "properties": {
    "schema_version": { "const": "rt/1" },
    "model_id":       { "$ref": "#/$defs/ModelId" },
    "context_root":   { "type": "string", "pattern": "^0x[0-9a-f]{64}$" },
    "steps": {                       // premise -> evidence -> conclusion chain (draft §2.2 Tier-1)
      "type": "array", "minItems": 1, "maxItems": 64,   // maxItems bounds CC-5 verification cost
      "items": {
        "type": "object",
        "required": ["role","text","refs"],
        "properties": {
          "role":  { "enum": ["premise","evidence","inference","conclusion"] },
          "text":  { "type": "string", "maxLength": 2048 },
          "refs":  { "type": "array", "items": { "$ref": "#/$defs/StateRef" } } // must resolve on-chain
        }
      }
    },
    "assertions": { "type": "array", "items": { "$ref": "#/$defs/Assertion" }, "maxItems": 32 },
    "confidence": { "type": "integer", "minimum": 0, "maximum": 10000 }  // basis points
  }
}
```

`StateRef` must point at real, current on-chain state (account, storage slot, tx hash, prior block). Tier-1 resolves every `ref` — a trace that cites non-existent state fails structurally. This is cheap and decidable.

### 2.2 Confidence calibration (binding in Tier-1)

Confidence is not free text; it is scored against the proposer's **historical assertion accuracy**, maintained on-chain.

```rust
// Brier-score-based calibration. A proposer who claims 95% confidence but is right 60%
// of the time gets a degraded reputation and (if binding) reduced reward weight.
struct CalibrationState { hist: RingBuffer<(conf_bps: u16, correct: bool)>; }

fn calibration_penalty(c: &CalibrationState) -> Reward Multiplier {
    let brier = c.hist.iter().map(|(p, y)| {
        let p = *p as f64 / 10_000.0; let y = if *y {1.0} else {0.0};
        (p - y).powi(2)
    }).sum::<f64>() / c.hist.len() as f64;
    // brier in [0,1]; 0 = perfect. Map to multiplier in [0.5, 1.0].
    (1.0 - brier).clamp(0.5, 1.0)
}
```

This makes confidence *costly to inflate* — a partial counter to CC-4 boilerplate (boilerplate that asserts high confidence and is occasionally wrong gets penalized). Operates only over **decidable assertions**, never over prose.

### 2.3 Three-tier verification — explicit binding posture

```rust
enum Tier { Structural, SemanticSample, FullReexec }
enum Binding { Binding, Advisory }

// Tier 1: Structural. BINDING. O(1) per validator.
fn tier1(ca: &CA, blk: &Block) -> Verdict {
    // schema valid? model pinned? refs resolve? assertions self-consistent? calibration in range?
    // + the decidable assertion check from §1.5. Slashable on false binding assertion.
}

// Tier 2: Semantic sampling. ADVISORY (red-team CC-2/CC-3 forbid binding here). O(n) committee.
fn tier2(ca: &CA, committee: &[Validator]) -> SampleResult {
    // committee members re-run inference on the SAME context_root (resolved from chain).
    // compare embeddings in DSO 7680-space.
    let sims: Vec<f64> = committee.map(|v| cosine(v.run_local(ca.context_root).embedding,
                                                  ca.trace_embedding));
    let agree = sims.iter().filter(|s| **s >= TAU_ADVISORY).count();
    // OUTPUT IS NOT A FINALITY GATE. It feeds:
    //   (a) proposer reputation (low agreement -> reputation decay, not slash)
    //   (b) SAIP telemetry (systematic disagreement -> anomaly signal)
    //   (c) dispute trigger ONLY for binding assertions that ALSO failed decidable check
    SampleResult { agree_frac: agree as f64 / committee.len() as f64, sims }
}

// Tier 3: Full re-exec. ADVISORY unless canonical-kernel determinism is live. O(n^2) on dispute.
fn tier3(ca: &CA) -> Verdict {
    // Requires DETERMINISTIC inference. Until canonical integer kernels ship (see §2.4),
    // Tier-3 can confirm ASSERTION truth (decidable, deterministic) but CANNOT slash on
    // trace divergence (non-deterministic across GPUs -> would slash honest nodes, CC-2).
    if !CANONICAL_KERNELS_LIVE { return Verdict::AssertionsOnly(check_assertions(ca)); }
    // ... full deterministic re-exec path, TEE-attested, when available ...
}
```

`TAU_ADVISORY` is a *reputation* threshold, not a consensus threshold — so the red-team's "no value of τ works for consensus" (CC-1.4) is sidestepped: a bad τ only mis-tunes reputation, never finality.

### 2.4 Determinism note (the precondition for binding Tier-2/3)

To ever make Tier-2/3 binding, inference must be bitwise-reproducible across heterogeneous GPUs. Spec'd path (out of scope to implement here, in scope to gate on):
- Canonical **integer-only** kernels (fixed-point GEMM + softmax) with a published reference, per pinned `model_id`.
- Fixed reduction order, fixed tile sizes, no kernel autotuning, no TF32.
- A conformance test vector set per model; nodes must pass before joining binding duty.
Until this exists, `CANONICAL_KERNELS_LIVE = false` and all trace-prose verification is advisory. **This flag is the single switch that the whole "binding consensus" claim hangs on.**

### 2.5 Verification compute metering (closes CC-5)

```rust
// Committee verification compute is PRICED. Proposer pays an inference-gas surcharge
// proportional to context size and max step count, refunded if trace verifies cleanly.
fn inference_gas(ca: &CA) -> Gas {
    let ctx_tokens = ca.context_token_count();         // from context_root manifest
    let max_decode = ca.steps_max_decode_estimate();   // bounded by schema maxItems=64
    BASE_INF_GAS + ctx_tokens * PER_CTX_TOKEN + max_decode * PER_DECODE_TOKEN
}
```

A "context bomb" now costs the proposer quadratically (large context => large gas), removing the asymmetric DoS.

---

## 3. C3 — SAIP generation pipeline

### 3.1 Safety posture (from red-team §2.6)

- SAIP output is **always a draft ZIP**, never an auto-applied change. No code path exists from SAIP to deployment without a human signature.
- Every SAIP that reaches the ballot has a **named human sponsor** (slashable / reputation-bound).
- SAIP emission is **rate-limited per epoch**, decoupled from telemetry magnitude (defeats SAIP-1 flooding, SAIP-6 telemetry-injection steering).
- The pipeline does **not** optimize for "what gets accepted" (no governance-outcome reward loop — defeats SAIP-4/5 capture flywheel). It may learn *drafting quality* from human-edit diffs, an objective that is not vote-coupled.

### 3.2 Pipeline stages

```
[telemetry] -> [anomaly aggregation] -> [debounce + rate-limit] -> [draft generation]
            -> [independent adversarial review] -> [sponsor adoption gate] -> [ballot]
```

```rust
struct SaipPipeline {
    telemetry:   TelemetryStore,        // append-only, tamper-evident (Merkle), §3.3
    rate_limit:  EpochBudget,           // e.g. MAX_SAIPS_PER_EPOCH = 2, hard cap
    author:      PinnedModel,           // governance-pinned (likely Zen-Mini 8B)
    reviewers:   Vec<IndependentModel>, // MUST be different families (red-team SAIP-3)
}

fn epoch_tick(p: &mut SaipPipeline, epoch: u64) -> Vec<SaipDraft> {
    let anomalies = aggregate_anomalies(&p.telemetry, epoch);   // §3.3
    // DEBOUNCE: only act on anomalies that persist >= K epochs (defeats single-epoch
    // telemetry injection, SAIP-6). RATE-LIMIT: at most MAX_SAIPS_PER_EPOCH regardless
    // of how many anomalies fire (defeats flooding, SAIP-1).
    let candidates = anomalies.persistent(K).top_n(p.rate_limit.remaining(epoch));
    candidates.into_iter().filter_map(|a| {
        let draft = p.author.generate_zip_draft(&a, evidence_from_chain(&a));
        let review = adversarial_review(&p.reviewers, &draft);   // §3.4
        if review.blocking_findings.is_empty() { Some(draft.with_review(review)) }
        else { p.telemetry.record_rejected(draft, review); None } // logged, NOT sent to ballot
    }).collect()
}
```

### 3.3 Telemetry store (tamper-evident, attacker-influence-bounded)

```rust
struct TelemetrySample {
    epoch: u64, metric: MetricId, value: f64,
    source_attest: TeeAttestation,   // sample provenance, PoAI TEE — not free-floating
}
// Anomaly = persistent deviation, robust to single-actor injection:
fn aggregate_anomalies(t: &TelemetryStore, e: u64) -> Anomalies {
    for m in ALL_METRICS {
        let series = t.window(m, e - W .. e);
        // ROBUST stat (median / MAD), not mean/std — so one whale spamming a metric
        // (SAIP-6) can't move the trigger. Requires deviation across MULTIPLE independent
        // sources (TEE-attested) before flagging.
        if robust_zscore(series) > Z && distinct_sources(series) >= MIN_SOURCES {
            flag(m, severity);
        }
    }
}
```

Metrics (draft §3.1): `tps, finality_latency, gas_util, validator_uptime, slash_freq, econ_velocity, mev_pattern`. Each is TEE-attested at the source so a single actor cannot manufacture an anomaly (raises the bar on SAIP-6).

### 3.4 Independent adversarial review (red-team SAIP-2/SAIP-3)

```rust
struct ReviewFinding { severity: Sev, kind: FindingKind, rationale: String }
enum FindingKind {
    SingleDiffVuln,          // reviewer-model strength
    CrossProposalConflict,   // checked against last N passed proposals (the SAIP-2 gap)
    EconomicEquilibrium,     // simulated, NOT just LLM-judged (see note)
    BackwardsIncompat,
}

fn adversarial_review(reviewers: &[IndependentModel], d: &SaipDraft) -> Review {
    // 1. DIVERSE models (different families) review independently — uncorrelated blind spots.
    let mut findings = reviewers.iter().flat_map(|r| r.review(d)).collect::<Vec<_>>();
    // 2. CROSS-PROPOSAL: replay d against the diff-stack of recently-passed SAIPs in a
    //    simulation fork; LLMs are bad at this (SAIP-2) so it is done by EXECUTION, not judgment.
    findings.extend(simulate_against_recent_passed(d));
    // 3. ECONOMIC: run d through an agent-based market sim, report invariant violations.
    findings.extend(econ_sim(d));
    Review { blocking_findings: findings.into_iter().filter(|f| f.severity >= Sev::High).collect() }
}
```

Note: the cross-proposal and economic checks are deliberately **execution/simulation-based, not LLM-judged**, precisely because the red team showed LLM reviewers are weakest exactly where the danger is (multi-step, cross-proposal, equilibrium). The LLM review is a *first pass*; simulation is the *gate*.

### 3.5 Sponsor adoption gate + on-chain SAIP record

```solidity
struct Saip {
    bytes32  draftCommit;      // commitment to the full draft ZIP text
    uint16   zipNumber;        // assigned in AI range 400-499
    address  sponsor;          // REQUIRED, non-zero. Human-controlled key.
    uint256  sponsorBond;      // slashable bond; lost if SAIP passes and causes measured harm
    bytes32  reviewRoot;       // Merkle root of all review findings (incl. simulation)
    MetricId[] triggeringAnomalies;
    Status   status;           // DraftedByProtocol -> Adopted -> Balloted -> {Passed,Rejected}
}

interface ISaipRegistry {
    // The protocol can only register a DRAFT. It CANNOT advance to Balloted.
    function registerDraft(Saip calldata s) external onlyProtocol;     // status = DraftedByProtocol
    // A human sponsor must adopt + bond before it can be voted. This is the accountability anchor.
    function adopt(uint16 zipNumber) external payable;                 // msg.sender = sponsor, bonds
    function ballot(uint16 zipNumber) external onlyGovernance;         // only after Adopted
}
```

No `adopt()` => the SAIP dies as an unballoted draft. This single requirement neutralizes the accountability vacuum and the auto-merge fear: **the protocol's output is inert until a human stakes their name and bond on it.**

---

## 4. C4 — Triple-use GPU scheduler + settlement

### 4.1 Scheduler core (the buildable, mostly-correct part)

```rust
enum Slot { Consensus, Training, Inference }

struct Scheduler {
    duty_floor:     HashMap<Slot, f64>,   // Training >= 0.20 (invariant 2) — but see §4.3
    budget:         GpuBudget,
    pending:        PriorityQueue<Job>,    // Consensus > Training > Inference
    epoch_ledger:   SlotLedger,            // per-slot time accounting -> settlement + commitments
}

fn schedule_next(s: &mut Scheduler, now: Instant) -> Job {
    // 1. Consensus preempts everything (invariant 1).
    if let Some(c) = s.pending.peek(Slot::Consensus) { return preempt_and_run(s, c); }
    // 2. Enforce training floor UNLESS economic guard trips (see §4.3 — the 2-vs-3 fix).
    if s.training_share(now) < s.duty_floor[&Slot::Training] && !s.econ_guard_tripped() {
        return s.pending.pop(Slot::Training);
    }
    // 3. Otherwise serve inference (revenue), else opportunistic training.
    s.pending.pop(Slot::Inference).or_else(|| s.pending.pop(Slot::Training))
}
```

### 4.2 Preemption that doesn't poison training (red-team CS-5)

```rust
// Gradients from a training window that was PREEMPTED by consensus are tagged and
// EXCLUDED from quality scoring / reward (they're stale/partial and preemption-correlated).
struct TrainingWindow { start: Instant, end: Instant, preempted: bool, grad: Option<Gradient> }

fn finalize_training(w: TrainingWindow) -> Option<ScorableGradient> {
    if w.preempted { metrics::inc("training.preempted_discarded"); return None; }  // CS-5 fix
    Some(ScorableGradient { grad: w.grad?, window: (w.start, w.end) })
}
```

Only uninterrupted windows produce reward-eligible gradients. This breaks the block-boundary-correlated bias the red team identified (invariant 4 was passing systematically-stale gradients).

### 4.3 The 2-vs-3 invariant reconciliation (red-team §4.1)

The draft's invariant 2 (training floor 20%) and invariant 3 (drop training if unprofitable) are jointly unsatisfiable at bootstrap. Explicit resolution:

```rust
// Three regimes, governance-parameterized. Makes the contradiction a STATED policy choice,
// not a hidden inconsistency.
enum EconRegime { Bootstrap, Subsidized, SelfFunding }

fn econ_guard_tripped(s: &Scheduler) -> bool {
    match s.regime {
        // BOOTSTRAP: protocol subsidizes training from block-reward issuance. Floor ENFORCED,
        // invariant 3 SUSPENDED. Honest about: this is NOT self-funding yet.
        EconRegime::Bootstrap  => false,
        // SUBSIDIZED: partial subsidy; floor enforced down to a lower bound; guard trips only
        // below survival margin.
        EconRegime::Subsidized => s.inference_revenue() < s.operating_cost() * SURVIVAL_MARGIN,
        // SELF_FUNDING: invariant 3 active; floor becomes a soft target, guard may push below it.
        EconRegime::SelfFunding => s.inference_revenue() < s.operating_cost(),
    }
}
```

Regime transitions are governance decisions tied to a *measured external inference-demand metric*. The "self-funding cycle" (draft §4.2) is thus reclassified as the **terminal** regime the network *aspires* to, not a property it has on day one. This directly answers the red-team's CS-1.

### 4.4 Per-slot commitments (closes CS-4 intra-epoch MEV hiding)

```rust
// Each node commits to its slot allocation BEFORE the epoch (commit-reveal), then settles.
struct SlotCommit { node: NodeId, epoch: u64, planned: HashMap<Slot, f64>, commit_hash: H256 }
struct SlotReveal { node: NodeId, epoch: u64, actual:  SlotLedger, salt: [u8;32] }

// At settlement, |actual - planned| beyond tolerance => reward penalty. This makes intra-epoch
// "do consensus only when MEV is high, dump otherwise" (CS-4) visible and costly.
fn settle_with_commitment(c: &SlotCommit, r: &SlotReveal) -> Reward {
    assert!(hash(r.actual_plan(), r.salt) == c.commit_hash);
    let drift = slot_drift(&c.planned, &r.actual);
    base_reward(&r.actual) * commitment_fidelity_multiplier(drift)
}
```

### 4.5 Sybil-resistant operator identity (red-team CS-2) — honest spec

The draft's "diminishing returns per operator" is Sybil-trivial. There is **no free lunch**; the spec states the tradeoff explicitly and picks bonded-identity:

```rust
// An "operator" is a bonded identity: a stake-bonded registration that gives a per-identity
// throughput cap. Splitting into N Sybils requires N bonds => capital cost scales with the
// Sybil attack, restoring the diminishing-returns intent. The COST is: this is stake-weighted,
// which has known plutocracy concerns (disclosed). Alternative (KYC registrar) rejected for
// centralization. This is a chosen tradeoff, not a solved problem.
struct Operator { id: OperatorId, bond: U256, gpu_cap: u32 }
fn marginal_reward_multiplier(op: &Operator, active_gpus: u32) -> f64 {
    // diminishing returns within an operator's bonded cap; Sybils each pay full bond.
    (1.0 / (1.0 + (active_gpus as f64 / op.knee()).powi(2))).max(MIN_MULT)
}
```

### 4.6 Atomic settlement transaction (draft invariant 5)

```solidity
// One settlement tx per node per epoch. Atomicity preserved; fairness handled by §4.4 commitments.
struct EpochSettlement {
    uint64  epoch; address node;
    uint256 consensusReward;   // block rewards + fees from Consensus slots
    uint256 trainingReward;    // 3x multiplier, ONLY non-preempted scorable gradients (§4.2)
    uint256 inferenceFees;     // settled per-request totals
    bytes32 slotRevealRoot;    // §4.4 — ties payout to committed allocation
    int256  commitmentPenalty; // negative adjustment for slot drift
}
function settleEpoch(EpochSettlement calldata s) external onlyConsensus { /* atomic credit */ }
```

---

## 5. Common types (referenced above)

### 5.1 ModelId — governance-pinned, separately versioned

```rust
// Resolves red-team OQ-7.1 + the circular-dependency / determinism trap: the consensus model
// is NOT the model the network is currently training. It is a frozen, governance-pinned,
// content-addressed artifact. Training produces CANDIDATES; governance PINS a new ModelId.
struct ModelId {
    family:   ModelFamily,   // Zen-Eco-4B | Zen-Mini-8B | Zen-Nano-0.6B
    weights_root: H256,      // content hash of frozen weights (HLLM: weights are frozen by design)
    kernel_spec: H256,       // hash of the canonical kernel spec (§2.4) — determinism anchor
    version:  u32,
}
```

This is also why HLLM (frozen weights + context-injection learning) matters: pinning is cheap because weights don't drift; "learning" happens in the Experience Ledger context, which is itself content-addressed.

### 5.2 Context manifest + context_root

```rust
// The exact, reproducible input to the model. Merkle-rooted so any verifier can fetch and
// confirm they fed the model the identical context. Bounds CC-5 (token count is in the manifest,
// drives inference_gas §2.5).
struct ContextManifest {
    block_height: u64,
    state_refs:   Vec<StateRef>,        // pinned to a state root
    experience:   Vec<ExperienceId>,    // DSO Experience-Ledger entries (content-addressed)
    token_count:  u32,                  // declared; verified during Tier-2 re-run
    prompt_template_id: H256,           // pinned prompt; not free-form
}
fn context_root(m: &ContextManifest) -> H256 { merkle_root(canonicalize(m)) }
```

### 5.3 Shared verdicts / offenses

```rust
enum Verdict  { Pending, Coherent, Divergent, Disputed, AssertionsOnly(bool) }
enum Offense  { FalseAssertion, FalseStateRef, SlotCommitFraud, /* NEVER: TraceProseDivergence */ }
```

`TraceProseDivergence` is deliberately **not** a slashable offense in v1 — it is the CC-2 honest-divergence trap. Listed here as a comment so future authors don't add it without first shipping canonical kernels (§2.4).

---

## 6. Build / test plan (measurable, non-binding first)

| Milestone | Deliverable | Gate to advance |
|---|---|---|
| M0 | C1 tx type + C2 trace schema + Tier-1 (advisory) on a devnet | Traces stored, assertions checked, 0 finality impact |
| M1 | Tier-2 committee sampling -> reputation only | Measure honest cross-GPU divergence rate (validates/refutes CC-2 empirically) |
| M2 | C4 scheduler + per-slot commitments + atomic settlement | Bootstrap regime; training subsidized; CS-4/CS-5 metrics clean |
| M3 | C3 SAIP pipeline: draft-only, named sponsor required, simulation-gated review | SAIPs reach ballot only with sponsor bond; flooding/injection tests pass |
| M4 | Canonical integer kernels + conformance vectors | Bitwise reproducibility across >=3 GPU archs |
| M5 (gated on M4) | Flip Tier-2/3 to binding; enable `binding_flag` for assertions | Only after M1 divergence rate ~ 0 under canonical kernels |

**Nothing in M0–M3 can affect finality or auto-apply a governance change.** The unsafe-as-designed behaviors from the draft are reachable only after M4/M5 preconditions are independently verified — which is exactly the discipline `03-red-team.md` argues the design currently lacks.

### 6.1 Adversarial test suite (one test per red-team failure mode)

```
CC-1 coherent-but-wrong:   craft self-serving assertion that passes embedding sim -> MUST be caught
                           by decidable assertion check, NOT by similarity.
CC-2 honest divergence:    run identical context on H100 vs MI300 -> MUST NOT slash (advisory).
CC-3 embedding mimicry:    optimize trace to maximize cosine sim while flipping conclusion ->
                           MUST NOT change finality (prose non-binding) and SHOULD lower reputation.
CC-5 context bomb:         max-context block -> inference_gas MUST scale; committee cost bounded.
SAIP-1 flooding:           spam anomaly triggers -> MUST be capped at MAX_SAIPS_PER_EPOCH.
SAIP-6 telemetry inject:   single source spikes a metric -> robust stat MUST NOT flag.
CS-2 Sybil:                split farm into N identities -> each MUST post full bond.
CS-4 slot MEV:             do-consensus-only-at-high-MEV pattern -> commitment penalty MUST bite.
CS-5 training poison:      force block-boundary preemption -> preempted gradients MUST be discarded.
2v3 invariant:             zero inference demand -> Bootstrap regime keeps training floor via subsidy;
                           SelfFunding regime is NOT auto-entered.
```

Every failure mode in `03-red-team.md` maps to a concrete, runnable test here. If a test cannot be written, that failure mode is not yet mitigated and the corresponding component stays advisory.
