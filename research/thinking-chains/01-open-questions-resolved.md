# Thinking Chains — Open Questions Resolved

**Companion to:** `thinking-chains-draft.md` (Pattermesh / KCOLBCHAIN Research)
**Status:** Resolution memo — proposed answers, tradeoffs, ZIP/LP integration
**Date:** 2026-06-30
**Grounding:** Every integration claim below is cross-checked in `02-grounding.md` against the real luxfi / zoo-apps / zenlm / hanzoai repositories.

---

## How to read this document

The draft's §7 lists 5 numbered open questions. Two further questions are latent in the body (verification cost economics in §2.2, and the ZIP-numbering collision implied by §5.3). This memo resolves **seven** questions total. For each: a concrete proposed answer, the tradeoff space, and exactly how it wires into the existing ZIP/LP stack.

A hard prerequisite surfaces repeatedly and is stated once here: **the proposed ZIP numbers 0420–0423 in the draft are already allocated** to shipped, Final-status ZIPs (0420 = 7680-Dimensional Embeddings, 0421 = Training-Free Preference Optimization, 0422 = Computer-Use Framework, 0423 = Privacy-Preserving AI Training / FHE). The AI range is populated through ZIP-0434. Thinking Chains must claim **ZIP-0435–0438** (next free block). This is folded into the relevant answers below and flagged hard in `02-grounding.md`.

---

## Q1 — Model selection for Cognitive Consensus

> *Should the consensus LLM be the same model the network trains (circular dependency), or a separate, smaller model dedicated to protocol reasoning?*

### Proposed answer: a **pinned, separate "operator" model**, version-gated by DSO governance — never the live training target.

Use a dedicated small reasoning model for consensus (the draft's Zen-Eco 4B Thinking or Zen-Nano variants), **frozen at a governance-pinned content hash**, with the *training* models (the ones the Compute Settlement training slot improves) kept strictly out of the consensus path. Concretely:

1. **Operator model = `zen-eco-thinking` 4B (or `zen-nano` for low-end validators), pinned by digest.** The consensus engine references the model by a content-addressed identifier committed on-chain, exactly as the Experience Ledger content-addresses experiences. A validator running a different digest is simply running a different protocol version and is detectable at Tier 1 verification.
2. **The training target is a different artifact.** The Compute Settlement "training slot" (draft §4) improves general-purpose Zen models for inference-as-a-service. Letting that artifact also be the consensus operator creates the circular dependency the question fears: today's blocks would be validated by a model whose weights are being mutated by today's blocks, destroying the deterministic-replay property that the whole verification scheme (draft §2.2/§2.3) depends on.
3. **Upgrades to the operator model are themselves SAIPs.** A model bump is a protocol change: it ships as a draft ZIP, carries the new digest + eval deltas, and is ratified by governance. Between upgrades the operator is immutable.

### Why this is the right cut

The draft's own §2.3 argument only holds for a **frozen** model: "the inference function is deterministic for a given (model, context, seed) tuple." That determinism is load-bearing for Tier 2 semantic sampling and Tier 3 full re-execution. A continuously-trained consensus model breaks it. The HLLM framework (ZIP-0418, Final) is precisely the escape hatch: model improvement happens in **context space, not parameter space** — so the operator's *behavior* can still improve over time (by accreting Experience-Ledger context) while its *weights* stay pinned and replayable. This is the single most important design decision in the paper and the draft under-commits on it.

### Tradeoffs

| Option | Determinism | Capability ceiling | Centralization risk | Verdict |
|---|---|---|---|---|
| Same model as training target | Broken (weights drift per block) | Highest | Whoever controls training controls consensus | **Reject** |
| Separate frozen operator, context-improved (HLLM) | Preserved | Grows via Experience Ledger | Governance-pinned digest is auditable | **Adopt** |
| Multiple competing operator models, vote on output | Preserved per-model | High | Low | Adopt *only* for Tier 2 committee (see Q2) |

### Integration with existing stack

- **ZIP-0418 (HLLM, Final):** supplies the frozen-weights / context-injection paradigm that makes a pinned operator improvable without drift.
- **ZIP-0410 (DSO, Final):** governs operator-model version pinning the same way it governs experience curation — model digest changes flow through DSO's DAO governance + Merkle-anchored provenance.
- **luxfi/consensus `ai/` package (real Go code):** the `BasicModel interface { Decide(ctx, question, data) (*SimpleDecision, error) }` is already the right seam. The operator model is whatever concrete type implements `BasicModel`; pinning = committing the implementation+weights digest. (See `02-grounding.md` — this interface exists today.)
- **zenlm/zen-eco-thinking (public):** the concrete chain-of-thought model. Determinism in production requires fixing sampler/seed and ideally greedy decode for the committed trace.

---

## Q2 — SAIP adversarial quality control

> *How do you prevent the LLM from generating adversarial proposals that look reasonable but introduce vulnerabilities?* (Draft proposes: multi-model adversarial review before governance vote.)

### Proposed answer: a **four-gate pipeline** — heterogeneous-model adversarial review → formal-spec/property check → simulation on shadow-fork → human governance — with **no SAIP ever reaching a vote on a single model's say-so**, and **machine proposals barred from being self-executing**.

The draft's instinct (multi-model adversarial review) is correct but insufficient on its own. Concretely:

**Gate 0 — Authorship constraint.** A SAIP may only *propose* a change expressible in the existing ZIP schema (problem / root-cause / change / risk / impl sketch). It may **never** emit a change that bypasses governance (e.g. it cannot propose "disable Gate 3"). This is enforced structurally, not by asking the model nicely.

**Gate 1 — Heterogeneous adversarial review.** The proposing operator model is *not* one of the reviewers. A committee of **distinct model families** (e.g. Zen-Eco-Thinking, Zen-Coder, a third non-Zen reference model) independently red-teams the SAIP for: hidden privilege escalation, economic-incentive distortion, backwards-incompat, and "looks-benign-but-isn't" parameter changes. Disagreement among reviewers is surfaced to governance rather than averaged away. Using genuinely different architectures matters: a monoculture of one model family shares blind spots and can be jointly fooled.

**Gate 2 — Formal & property gate.** Any SAIP that touches consensus-critical parameters must come with, or be auto-paired to, a machine-checkable property. Zoo already maintains **Lean 4 proofs** (`zoo-labs/proofs/lean/` — `PoAI.lean`, `Consensus`, `ExperienceLedger.lean`, `Gym.lean`) and Lux maintains **EasyCrypt** proofs for its crypto primitives (`luxfi/crypto`, `luxfi/threshold`, `luxfi/pulsar`, `luxfi/fhe`). A SAIP that would violate a stated safety/liveness invariant (e.g. PoAI's `f < n/3` fault bound, quorum intersection) is auto-rejected before it can reach humans. SAIPs that *change* an invariant must ship the amended proof obligation.

**Gate 3 — Shadow-fork simulation.** The proposed change runs on a forked replica of mainnet state for N epochs. Telemetry deltas (TPS, finality latency, MEV, slashing rate) are measured empirically, not argued rhetorically. A SAIP whose claimed benefit doesn't materialize in simulation is downgraded.

**Gate 4 — Human governance.** Only SAIPs that clear 1–3 reach token-holder vote. The LLM proposes; humans decide; the machine never self-deploys. This is the draft's stated principle (§3.1) and must be a hard architectural invariant, not a convention.

### The key threat the draft underestimates

A sophisticated failure isn't a single bad SAIP — it's **gradual incentive capture**: a sequence of individually-reasonable SAIPs that collectively shift value toward whoever influences the proposal pipeline. Mitigations: (a) the Experience Ledger records *who/what context* shaped each accepted SAIP, making drift auditable after the fact; (b) a "constitutional" set of invariants (fault bounds, anti-centralization caps from Q3, governance-supremacy from Gate 0) that **no SAIP can amend** without a supermajority + cooling-off period; (c) reviewer-model rotation so the proposer can't learn to game a fixed reviewer.

### Tradeoffs

- More gates → slower pipeline, partially defeating the "accelerate proposals" goal. Resolution: gates 1–3 are *automated and parallel*; only gate 4 is human-rate-limited. Net pipeline is still vastly faster than today's researcher→paper→ZIP path.
- Heterogeneous models cost more inference per SAIP. Acceptable: SAIP generation is epoch-frequency, not block-frequency, and runs in idle consensus slots under Compute Settlement (Q3).

### Integration

- **zoo-labs/proofs (Lean 4) + luxfi/{crypto,threshold,pulsar,fhe} (EasyCrypt):** Gate 2's formal backstop. Real and shipped.
- **luxfi/consensus `ai/` package:** already implements *autonomous-upgrade / dispute-resolution / fork-arbitration* decision objects (`SimpleDecision{Action, Confidence, Reasoning}`). SAIP review committee = multiple `BasicModel` instances producing competing `SimpleDecision`s; the package's "cross-chain coordination weighted by votes" is the aggregation seam.
- **ZIP process (zoo-apps/ZIPs):** SAIPs emit standard `zip-04xx` drafts with the existing front-matter (`status: Draft`, `traces-from`, `requires`). No new governance machinery needed.

---

## Q3 — Compute-Settlement anti-centralization & fairness

> *How do you prevent large GPU operators from monopolizing all three slots and centralizing the network?* (Draft proposes: diminishing returns per additional GPU per operator.)

### Proposed answer: **per-operator concave reward + identity-bonded stake + slot-class separation**, with diminishing returns applied to the *consensus* reward specifically (not to honest inference revenue).

Diminishing returns alone (the draft's proposal) is the right primitive but has two holes: (a) Sybil — an operator splits into many identities to dodge the curve; (b) over-penalizing legitimately efficient inference providers. Refined design:

1. **Concave consensus reward per operator-identity.** Block-production/consensus reward scales as a concave function of an operator's attributed compute (e.g. √-stake-style or explicit cap), so the marginal block reward of the 1000th GPU under one operator is far below the 1st. This is the draft's diminishing-returns idea, scoped to the *security* function where centralization actually threatens safety.
2. **Sybil resistance via PoAI's existing identity layer.** PoAI (ZIP-0419 / LP-302) already requires **TEE attestation + on-chain AI identity (ZIP-0403)** for compute attribution. The concave curve is applied per attested identity, and identity carries a bonded stake — so splitting into N identities costs N bonds and N attestations, removing the cheap-Sybil escape. (ZIP-0419 lists Sybil resistance as a first-class design goal; we reuse it rather than reinvent.)
3. **Slot-class separation prevents cross-subsidy monopoly.** Consensus, training, and inference are accounted separately at settlement (draft §4.4 already makes settlement atomic-but-itemized). Apply the anti-centralization curve to *consensus rewards only*. Inference-as-a-service is a competitive market — penalizing efficient inference providers would just push that revenue off-chain. Training rewards already pass PoAI quality gating, which naturally rewards diverse contributions over raw bulk.
4. **Minimum-diversity quorum for Cognitive Consensus.** Tier-2 semantic-sampling committees (draft §2.2) must be drawn from ≥k distinct operators (luxfi/consensus **Photon** already does committee selection via Fisher-Yates with luminance-weighted reputation — extend it to enforce operator-distinctness, not just stake-weight). This stops one operator from populating an entire verification committee even if it controls a large stake share.

### Tradeoffs

- Concave rewards reduce total yield for large efficient operators → mild efficiency loss in exchange for decentralization. Standard, accepted PoS-design tradeoff.
- Identity bonding raises the barrier for small honest operators. Mitigation: bond scales with claimed capacity, so a hobbyist Zen-Nano validator posts a small bond.
- "Consensus reward concave, inference reward linear" creates an arbitrage incentive to misreport work as inference. Closed by PoAI's compute-binding (LP-302's whole point: bind reward to the *computation actually performed*, via Freivalds-style checks + TEE), so misclassification is detectable.

### Integration

- **ZIP-0419 / LP-302 (PoAI, Final):** TEE attestation, quality scoring, Sybil resistance, compute-binding — all reused directly. The anti-centralization curve is a parameter on top of PoAI reward accounting, not new infrastructure.
- **ZIP-0403 (On-Chain AI Identity, Final):** the per-operator identity the concave curve is keyed on.
- **luxfi/consensus `protocol/photon` (real):** committee selection extended for operator-distinctness.
- **luxfi/lens / luxfi/threshold / luxfi/safe-frost (real):** threshold-signature machinery for bonded multi-operator quorums.

---

## Q4 — Reasoning-trace privacy (MEV / strategy leakage)

> *Some reasoning traces may reveal MEV opportunities or trading strategies. Should traces be encrypted (FHE) or delayed?* (Draft proposes: FHE-encrypted traces with delayed decryption per the PoAI TEE framework.)

### Proposed answer: **commit-reveal with timed/threshold decryption as the default; FHE only for the narrow class of traces that must be *verified while still encrypted*.** Don't FHE everything — it's the wrong cost/benefit for most traces.

The draft conflates two different privacy needs. Separate them:

1. **Most reasoning traces don't need to be confidential forever — only until the block is final.** The MEV/strategy leak is a *front-running* problem: the danger is a trace revealing intent *before* execution. The clean fix is **commit-reveal**: the validator commits `hash(trace)` with the block, and reveals the plaintext trace one (or k) epochs later, after the MEV window has closed. Tier-1 structural verification can run on the committed hash + schema; full semantic verification (Tier 2/3) runs post-reveal. This is cheap, deterministic, and reuses the chain's existing ordering. **This should be the default.**
2. **Timed/threshold decryption for stronger guarantees.** Where "reveal after k epochs" must be enforced without trusting the committer to reveal, use **threshold decryption**: the trace is encrypted to a validator threshold key (Lux's FROST/threshold stack — `luxfi/safe-frost`, `luxfi/lens`, `luxfi/threshold`), and decrypts automatically when the committee releases shares at the scheduled height. No single party can early-decrypt; the trace is guaranteed to surface.
3. **FHE only for verify-while-encrypted.** FHE earns its (large) cost *only* if a verifier must check trace properties *without ever seeing the plaintext* — e.g. confidential trading strategies that should never be revealed even after finality. Lux ships real FHE: **LP-013 (TFHE on GPU, F-Chain), `luxfi/fhe` repo, ZIP-0423 (Privacy-Preserving AI Training)**. Route this narrow class to F-Chain; keep the common case on commit-reveal.
4. **TEE as the pragmatic middle.** For traces that need confidentiality *and* low latency, PoAI's existing **TEE attestation** (LP-302; institutional TEE composition in LP-020) lets verification happen inside an enclave: the verifier attests it ran the check correctly without exposing the trace. Cheaper than FHE, weaker trust model.

### Decision matrix

| Trace sensitivity | Mechanism | Cost | Where it lives |
|---|---|---|---|
| Routine (anomaly flags, fee notes) | Plaintext, immediate | ~0 | base chain |
| Front-runnable intent | **Commit-reveal, delayed** | low | base chain ordering |
| Must-reveal-but-not-early | **Threshold/timed decryption** | medium | `luxfi/threshold`, `safe-frost` |
| Never-reveal, verify-encrypted | **FHE** | high | LP-013 F-Chain, `luxfi/fhe` |

### Why this corrects the draft

The draft's "FHE-encrypted traces with delayed decryption per ZIP-0419 TEE framework" muddles three distinct tools (FHE, delay, TEE) into one phrase. They are different mechanisms with different costs and trust models. FHE-everything would make Cognitive Consensus economically nonviable (TFHE bootstrap is orders of magnitude slower than the ~100ms inference budget the draft assumes in §4.1). Commit-reveal solves the *stated* threat (MEV front-running) at near-zero cost; reserve FHE for the genuine never-reveal minority.

### Integration

- **LP-013 (FHE on GPU / F-Chain, Final) + `luxfi/fhe` (real repo):** the encrypted-verification path.
- **ZIP-0423 (Privacy-Preserving AI Training / FHE, Final):** Zoo-side FHE spec; trace-FHE is an application of the same primitives.
- **`luxfi/threshold`, `luxfi/safe-frost`, `luxfi/lens` (real):** threshold/timed decryption.
- **LP-302 / LP-020 (TEE):** enclave-verified traces.
- **NOTE:** the draft attributes the TEE framework to "ZIP-0419" — that is correct (PoAI = ZIP-0419, mirror of LP-302, which specifies the TEE attestation). Verified in `02-grounding.md`.

---

## Q5 — Bootstrapping the Experience Ledger for protocol-level reasoning

> *The first Thinking Chain has no history to learn from. How do you seed the Experience Ledger for protocol-level reasoning?* (Draft proposes: seed from the Zen Agentic Dataset's "12B tokens of blockchain engineering context.")

### Proposed answer: a **three-source cold-start** — (a) the existing ZIP/LP corpus as structured priors, (b) the real Zen Agentic Dataset, and (c) **synthetic shadow-fork experience generation** — then transition to live accretion. **Correct the token figure.**

**Source A — the ZIP/LP corpus is the highest-value seed and the draft omits it.** There are ~35 AI-range ZIPs (0400–0434), the full LP series (luxfi/LPs, hundreds of specs incl. LP-302 PoAI, LP-017/LP-217 Quasar), the Lean/EasyCrypt proof corpus, and 24+ Zoo research papers. This is *already* a structured, governance-grade record of "what protocol changes were proposed, accepted, and why." Ingest it as the initial protocol-reasoning context: every Final ZIP is a worked example of an accepted improvement; every superseded LP (e.g. LP-017 "superseded by LP-217") is a worked example of an *evolution decision*. This is exactly the experience type SAIPs need and it exists today.

**Source B — the Zen Agentic Dataset, with the number corrected.** The draft says "12B tokens." The real artifacts: `zenlm/zen-agentic-dataset` is tagged `size_categories: 10B<n<100B` (so 10B–100B is defensible as a *range*), but the human-readable repo descriptions cite **"721M+ tokens"** (`zen-agentic-dataset`) and **"671M+ tokens"** (`zen-claude-agentic-full`). **Do not state "12B" as fact** — either cite the `10B<n<100B` size class or the 721M figure, with a link. (Flagged in `02-grounding.md`.) This dataset is agentic/coding context, not specifically "blockchain engineering" — useful for the operator model's general reasoning, less so as protocol-governance priors. It complements Source A; it does not replace it.

**Source C — synthetic experience via shadow-fork.** Before mainnet, run the operator model against the **shadow-fork simulator** (the same Gate-3 machinery from Q2) replaying historical/synthetic chain states with injected anomalies (congestion, slashing storms, MEV spikes, fork scenarios). Each handled scenario becomes a seed experience with a *known-good* outcome label. This is the only source that produces *protocol-operation* experience (as opposed to general agentic text) at volume, and it's cheap because it reuses Compute Settlement idle slots.

**Transition:** seed (A+B+C) → boot mainnet in "advisory mode" (Cognitive Consensus traces produced and verified but not yet reward-bearing) for an epoch window → promote to reward-bearing once trace accuracy/calibration crosses a governance-set threshold → live accretion takes over, with the Experience Ledger's content-addressed, Merkle-anchored, DAO-curated pipeline (ZIP-0410, Final) doing the ongoing curation.

### Tradeoffs

- Synthetic seeds risk teaching the operator to handle *simulated* not *real* pathologies → mitigate with the advisory-mode shadow period on live traffic before rewards turn on.
- Ingesting the whole ZIP/LP corpus risks anchoring SAIPs to past framings → mitigate with reviewer-model rotation (Q2) and by labeling superseded specs as such.

### Integration

- **ZIP-0410 (DSO, Final) + Experience Ledger (`experience-ledger-dso.tex`, `ExperienceLedger.lean`):** the content-addressable, Merkle-verified, IPFS/Arweave-stored, DAO-governed store. Real and specified. Seeds are written here.
- **ZIP-0401/0404 (Persistent / Content-Addressable Semantic Memory, Final):** the memory-state + CID substrate the Ledger is built on.
- **zoo-apps/ZIPs + luxfi/LPs (real corpora):** Source A.
- **zenlm/zen-agentic-dataset (real, private mirror in hanzoai):** Source B — cite correctly.
- **Shadow-fork simulator:** shares implementation with Q2 Gate-3; not yet a shipped component (new build, see Q6).

---

## Q6 — (Latent) Verification-cost economics & the "what's actually new" boundary

> The draft's §2.2 assumes ~100ms inference per block and that Tier-2 semantic sampling is affordable, and §5.2 lists five "new" components. Are the economics and the build boundary realistic?

### Proposed answer: the verification scheme is sound **only** if Tier-2/3 are rare; budget the system around making disputes rare, and scope the genuinely-new build to **four components, not a new chain.**

1. **Make Tier-3 (full re-execution, O(n²)) genuinely rare.** It must be dispute-triggered only, with slashing severe enough that provoking it is never profitable. PoAI's probabilistic-audit + TEE model (LP-302) already establishes "verify cheaply, escalate rarely"; Cognitive Consensus inherits this. If Tier-2 fires on a meaningful fraction of blocks, the design is uneconomic — so the Tier-2 cosine-similarity threshold τ must be tuned (in advisory mode, Q5) so the common case is Tier-1-only.
2. **Tier-1 is the workhorse and is genuinely O(1).** Schema conformance + confidence calibration + on-chain-state-reference checks need no inference. This is where ~all blocks should resolve.
3. **The ~100ms/block inference budget is plausible for Zen-Eco 4B / Zen-Nano** on a validator-class GPU with greedy decode on a bounded trace — but is a *per-deployment* parameter, not a guarantee. State it as configurable and gate reward-bearing mode on measured latency staying within the block interval.
4. **The real "delta" is four components, and Thinking Chains is a composition layer, not a new chain** — the draft's framing (§5.1 "no new infrastructure") is largely right. Genuinely new build:
   - `CognitiveAttestation` transaction type + trace schema (medium) — rides on the existing EVM/precompile + Quasar block path.
   - Trace verification driver (Tier 1/2/3) — extends luxfi/consensus `ai/` + `protocol/{photon,wave,focus,ray,field}`.
   - SAIP generation+review pipeline (high) — orchestration over existing `ai/` decision objects, ZIP emission, Lean/EasyCrypt gates.
   - Unified Compute-Settlement accounting + GPU scheduler (high) — smart contract + node daemon over PoAI reward accounting.

   Everything else (inference at validators, TEE, 7680-dim embeddings, Experience Ledger, decentralized training, edge models, ZIP governance, Quasar) is shipped or specified.

### Integration

- **luxfi/consensus (Quasar, real):** sub-protocols Photon/Wave/Focus/Nova/Nebula/Prism/Horizon/Flare/Ray/Field/Quasar. The draft's "pluggable finality drivers" claim is *approximately* right but imprecise: the finality **drivers** are specifically **Ray** (linear) and **Field** (DAG); signing **modes** (BLS / BLS+ML-DSA / BLS+Pulsar) are independently toggleable. Cognitive Consensus attaches as an attestation-verification stage, not by swapping the finality driver. Reword the draft accordingly (see `02-grounding.md`).
- **ZIP-0419 / LP-302 (PoAI):** the cost/audit discipline.
- **ZIP-0015 (Zoo L2 Chain Architecture, Final):** the EVM/settlement layer the draft (mis-implicitly) leans on — note the draft's table cites "ZIP-0015" and that is **correct**.

---

## Q7 — (Latent) ZIP numbering, naming, and provenance hygiene

> The draft proposes ZIP-0420…0423 and cites several specs by number. Are the numbers and names correct, and does the paper claim anything as shipped that isn't?

### Proposed answer: **renumber to ZIP-0435–0438, fix three citation errors, and downgrade the "SAIP authorship is autonomous / self-evolving" rhetoric to match what the `ai/` package actually does today.**

1. **Renumber.** 0420–0423 are taken (0420 embeddings, 0421 TF-preference-optimization, 0422 computer-use, 0423 FHE). Claim the next free block:
   - ZIP-0435 — Cognitive Consensus: LLM-Verified Block Attestations
   - ZIP-0436 — Self-Authoring Improvement Proposals (SAIPs)
   - ZIP-0437 — Compute Settlement Layers: Triple-Use GPU Economics
   - ZIP-0438 — Reasoning-Trace Verification via Semantic Embeddings
2. **Citation fixes** (full detail in `02-grounding.md`): "HLLM-TF-GRPO" should resolve to ZIP-0418 (HLLM) + ZIP-0421 (Training-Free Preference Optimization) + paper `hllm-training-free-grpo` + external anchors arXiv:2510.08191 (Training-Free GRPO) and arXiv:2512.24615 (Youtu-Agent). The "Quasar supports pluggable finality drivers" line needs the Ray/Field precision from Q6. The Zen-dataset "12B tokens" figure must be corrected (Q5).
3. **Calibrate the autonomy claims.** luxfi/consensus `ai/` *does* implement autonomous-upgrade/fork-arbitration/dispute-resolution decision plumbing today (real Go, `SimpleDecision{Action,Confidence,Reasoning}`), which substantiates the *mechanism* behind Cognitive Consensus and SAIPs. But the concrete `BasicModel` is an interface — the production operator-LLM wiring is the new work, and SAIP self-authoring as described (telemetry→proposal→multi-gate→vote) is **not yet a shipped pipeline**. Keep the paper's "the LLM proposes, humans decide" framing; avoid implying the self-evolution loop already runs in production. This keeps the paper credible to Zoo/Lux reviewers who know their own stack.

### Integration

- **zoo-apps/ZIPs (real ZIP repo — note: NOT `zoo-labs/zips`, which is the website and holds only 0001/0100/0200):** numbering authority. Confirmed in `02-grounding.md`.
- **ZIP-0000 / INDEX.md:** AI range is 0400–0499; new ZIPs slot at 0435+.

---

## Summary of resolutions

| Q | Resolution in one line | New build? | Reuses |
|---|---|---|---|
| Q1 Model selection | Pinned separate operator (Zen-Eco-Thinking/Nano), HLLM-context-improved, never the training target | config + governance | ZIP-0418, ZIP-0410, consensus/`ai` |
| Q2 SAIP quality | 4-gate: heterogeneous review → Lean/EasyCrypt → shadow-fork → human vote; no self-execution | pipeline (high) | proofs (Lean+EC), consensus/`ai`, ZIP process |
| Q3 Anti-centralization | Concave consensus reward per bonded identity + slot separation + diverse committees | reward params | ZIP-0419/LP-302, ZIP-0403, photon |
| Q4 Trace privacy | Commit-reveal default; threshold/timed decryption; FHE only for verify-encrypted | accounting | LP-013/luxfi-fhe, threshold/safe-frost, TEE |
| Q5 Bootstrapping | ZIP/LP corpus + Zen dataset (number corrected) + synthetic shadow-fork; advisory mode | seeding + simulator | ZIP-0410/0401/0404, zen-agentic-dataset |
| Q6 Verification economics | Tier-1 workhorse, Tier-3 rare-by-slashing; 4 new components, composition not new chain | 4 components | Quasar, PoAI, ZIP-0015 |
| Q7 Numbering/provenance | Renumber 0435–0438; fix 3 citations; calibrate autonomy claims | doc fixes | zoo-apps/ZIPs |
