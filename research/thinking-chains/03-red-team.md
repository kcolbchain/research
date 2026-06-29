# Thinking Chains — Red Team Analysis

**Companion to:** `thinking-chains-draft.md` (Cognitive Consensus, SAIPs, Compute Settlement Layers)
**Role:** Adversarial review. Assume the protocol is live, mainnet-valuable, and under attack by economically rational and ideologically motivated adversaries.
**Stance:** Skeptic. The job here is to break the design before an attacker does. Each primitive is rated for readiness on a 0–5 scale (0 = conceptual, 5 = production-hardened).
**Date:** 2026-06-30
**Status:** DRAFT — for Zoo Labs Foundation / Lux Network internal review

---

## 0. Executive summary

The draft is intellectually coherent and the "compose existing primitives" framing is its strongest asset. But three of the load-bearing claims do not survive contact with an adversary:

1. **"Reasoning coherence" is not a verifiable consensus predicate.** Cognitive Consensus conflates *semantic plausibility* (cheap, gameable, non-deterministic across hardware) with *correctness* (the thing consensus must actually agree on). Tier-2 embedding similarity does not detect a coherent-but-wrong trace, and the determinism claim breaks on real GPU floating-point.
2. **SAIPs move the trust boundary, they don't remove it.** Autonomous authorship + human voting creates a *new* attack surface — adversarial-but-plausible proposals weaponizing voter inattention and reviewer-model collusion — without a corresponding new defense. The Experience-Ledger feedback loop is a reward-hackable objective.
3. **The "self-funding cycle" is a demand-side assumption dressed as an accounting identity.** The triple-use scheduler is sound as engineering, but the economic invariants are individually defensible and *jointly inconsistent* under stress (invariant 3 can force invariant 2 to be violated; invariant 1's preemption is a covert-channel and a griefing vector).

| Primitive | Readiness | One-line verdict |
|---|---|---|
| Cognitive Consensus | **2 / 5** | Sound as an *advisory* sidecar; unsound as a *consensus-binding* mechanism without a hard correctness oracle. |
| SAIPs | **2 / 5** | Useful as a drafting assistant; dangerous if its output is given any privileged path to deployment. Governance is the real attack surface and it is under-specified. |
| Compute Settlement | **3 / 5** | The scheduler is buildable and largely correct; the economics have a circular-demand assumption and the invariants conflict under load. |
| 5 economic invariants | **Partial** | 2 hold, 1 is gameable, 2 are jointly unsatisfiable under stress. See §4. |

**Bottom line:** The architecture is publishable as a research direction and shippable as a *non-binding intelligence layer* on top of an existing safe consensus. It is **not** ready to have reasoning traces gate finality, nor to give SAIPs any auto-merge path. The draft's own "open questions" §7 propose defenses that, on inspection, do not close the holes (see §5).

---

## 1. Cognitive Consensus — adversarial analysis

### 1.1 The category error at the core

Consensus must agree on a **decidable predicate** over a **canonical input**: given block B, is it valid — yes or no — and will every honest node reach the *same* answer? The draft asks validators to verify "reasoning coherence" in addition to "computational correctness" (§2.1, §2.2). These are different kinds of object:

- *Computational correctness* (did transactions apply correctly to state) is already decidable and already handled by the base execution layer. Cognitive Consensus adds nothing here.
- *Reasoning coherence* (is the chain-of-thought a "good" explanation) is **not a decidable predicate**. There is no ground-truth function `coherent(trace) → {0,1}` that all honest nodes compute identically. The draft substitutes embedding cosine similarity for this function, which is a similarity metric, not a correctness oracle.

The consequence: a trace can be **coherent and wrong**, or **incoherent and right**, and the verification scheme cannot tell the difference. An attacker exploits the first case; the protocol punishes honest nodes in the second.

> **Failure mode CC-1 (coherent-but-wrong).** Proposer emits a fluent, schema-valid trace whose stated conclusion ("mempool tx 0x… is low MEV-risk, prioritize") is *semantically plausible* and *embeds near* the honest committee's traces, but is factually wrong because the proposer privately knows the tx is a sandwich setup. Tier 1 passes (schema OK, on-chain refs correct). Tier 2 passes (high cosine similarity to a plausible honest answer — there are many plausible answers, and "low risk" is one of them). Tier 3 never triggers because there is no dispute — the answer *looked* right. The proposer has laundered a self-serving decision through "consensus."

### 1.2 The determinism claim is false on real hardware

§2.3 leans entirely on: "the inference function is deterministic for a given (model, context, seed) tuple … reasoning traces are reproducible." This is the keystone of Tier-3 dispute resolution and slashing. **It does not hold across heterogeneous GPUs**, which is exactly the network ZIP-0407 / PoAI assume.

- Transformer inference is **not bitwise-reproducible across** different GPU architectures, driver/cuDNN versions, batch sizes, tensor-parallel shardings, or kernel autotuning choices. Floating-point non-associativity in reduction order alone produces logit differences that, after a long autoregressive decode, diverge into *different tokens* and thus *different traces*.
- "Fixed random seed" controls sampling, not the floating-point path. Even greedy (temperature 0) decoding diverges across hardware once two candidate tokens have near-equal logits — a routine occurrence over hundreds of decode steps.

> **Failure mode CC-2 (honest divergence → false slashing).** An honest validator on an AMD MI300 produces a trace that diverges token-for-token from the honest reference on an NVIDIA H100. Tier-3 "deterministic re-execution" finds divergence and slashes an honest node. Now honest operators are incentivized to *homogenize hardware* (centralization pressure) or *not participate* in Tier-3-eligible duty. This directly contradicts the heterogeneous-GPU premise of the underlying stack.

The only way to recover determinism is to (a) mandate identical hardware + identical kernel builds (kills decentralization), or (b) do inference in fixed-point / integer-only arithmetic with a canonical kernel spec (enormous engineering, large quality hit, and still must be re-derived per model), or (c) treat the trace as **advisory and non-slashable** — which collapses Cognitive Consensus into a non-binding sidecar (which is, in fact, the only safe version — see §6).

### 1.3 Reasoning-trace gaming (the proposer's game)

Even granting determinism, the proposer chooses the trace. The scheme creates a metric (`coherence`) and then asks the metric-producer to also be the metric-target — textbook Goodhart.

> **Failure mode CC-3 (embedding-space mimicry).** The proposer doesn't need to reason; it needs to emit text that *embeds* where honest reasoning embeds. Since the committee compares in the 7680-dim DSO space with threshold τ, the proposer can run a cheap local copy of the committee check and *optimize the trace to maximize cosine similarity to the modal answer* while smuggling its preferred conclusion. Embedding spaces are smooth: you can move the conclusion token's contribution without moving the aggregate embedding much. This is an adversarial-example attack against the verifier, and verifier robustness against adversarial text is an open ML problem, not a solved one.

> **Failure mode CC-4 (template collapse / coherence cartel).** Validators converge on a canonical trace template that always passes Tier-2 (because everyone embeds near everyone else by construction). The "reasoning" becomes boilerplate that carries zero information but maximizes pass rate and minimizes compute. The protocol pays a real inference cost for a trace that has been optimized to be *verifiable*, not *useful*. Anomaly detection (§3.2) built on top of this signal now reads cartel boilerplate as "normal."

### 1.4 Threshold τ is an unforced governance hole

Tier-2 fires Tier-3 when cosine similarity < τ. Who sets τ? 
- τ too high → honest hardware divergence (CC-2) constantly trips disputes; the chain spends O(n²) re-execution and slashes honest nodes. DoS-by-strictness.
- τ too low → CC-1/CC-3 always pass; the mechanism is decorative.
- There is **no value of τ that simultaneously rejects adversarial mimicry and accepts honest divergence**, because adversarial traces (CC-3) are *designed to embed closer than honest hardware noise*. The attacker's similarity is higher than the honest false-divergence rate. The ROC curve is adversarially controlled.

### 1.5 Liveness and the committee

Tier-2 is "a committee of k validators re-runs inference, O(n)." Re-running 4B–8B inference on potentially adversarial context is a **liveness and griefing surface**:

> **Failure mode CC-5 (context bomb).** Proposer crafts a block whose "current network state as context" maximizes the committee's inference cost (long context, worst-case attention, prompt that induces long generations). Committee members must spend the compute or skip verification. If they skip, CC-1 passes trivially; if they spend, throughput collapses. There is no gas metering described for *verification-side* inference cost — only the proposer pays gas, but the committee bears unpriced cost. This is an asymmetric-work DoS.

### 1.6 MEV leakage through traces

The draft's own §7.4 flags this, but understates it. The reasoning trace is, by design, the proposer's *private analysis of mempool ordering and risk*. Committing it on-chain — even with delay — is publishing alpha.

> **Failure mode CC-6 (trace as MEV oracle).** Other searchers read committed traces to learn the proposer's ordering heuristics, anomaly thresholds, and which addresses it flags. Over time this is a free model-distillation of the network's collective MEV strategy, and a map of which transactions the protocol considers "safe to front-run." Delayed decryption (the proposed fix) only delays the leak; the heuristics are stable across blocks, so yesterday's decrypted traces predict today's behavior. FHE on the *verification* path (committee must compare encrypted traces) is not practical at consensus latency in 2026.

### 1.7 Verdict — Cognitive Consensus

**Readiness 2/5.** The primitive is coherent as an *advisory* layer (a per-block AI commentary that informs but does not bind). As a *consensus-binding* mechanism it has a category error (coherence ≠ correctness), a false determinism assumption (CC-2), an adversarially-controlled verifier (CC-3/CC-4), an unsatisfiable threshold (CC-1.4), an unpriced DoS surface (CC-5), and a structural MEV leak (CC-6). 

**To reach 3/5:** drop "reasoning coherence" as a consensus predicate; bind only on a hard correctness oracle (e.g., the trace must produce *checkable actions* — "I prioritized tx X" is verified against the actual block ordering, a decidable fact — not against a vibe). Make traces advisory + non-slashable. Price verification compute. Then it is a useful, safe sidecar.

---

## 2. Self-Authoring Improvement Proposals (SAIPs) — adversarial analysis

### 2.1 The trust boundary moved; the threat didn't shrink

The draft's safety claim is "the LLM *proposes*; humans *decide*" (§3.1). This is correct as far as it goes and it is the right instinct. But it implies the entire security of SAIPs rests on the **human governance vote**, which the draft treats as a solved, trusted oracle. It is not. SAIPs *industrialize proposal production* and aim at *accelerating the pipeline* (§3.1) — i.e., they increase the **rate** and **plausibility** of proposals hitting a governance process whose throughput and scrutiny are fixed by human attention. That is a denial-of-service on governance review, and a plausibility attack on voters.

> **Failure mode SAIP-1 (proposal flooding / attention exhaustion).** The protocol auto-generates proposals whenever "metrics deviate beyond thresholds" (§3.1). An attacker who can nudge telemetry (even slightly — see SAIP-3) can trigger a flood of plausible SAIPs. Token-holder attention is the bottleneck the draft explicitly wants to relieve; relieving the *supply* side while the *review* side stays human-bound guarantees that review quality per proposal *drops*. Low-scrutiny voting is exactly the condition under which a malicious-but-plausible proposal passes.

### 2.2 Adversarial-but-plausible proposals

§7.2 names this risk and proposes "multi-model adversarial review before governance vote." This defense has two holes.

> **Failure mode SAIP-2 (subtle-vulnerability proposal).** The dangerous SAIP is not the one that says "send all funds to 0xattacker." It's the one that proposes a parameter change that is *individually reasonable* and *jointly catastrophic* with another recently-passed change, or that introduces a vulnerability three steps removed from the diff (e.g., "reduce finality timeout to improve latency" → interacts with a reorg edge case). LLM reviewers are *worse* at multi-step, cross-proposal, economic-equilibrium reasoning than at single-diff code review. The review model and the authoring model share training distribution and failure modes — **correlated blind spots**. An adversarial input that fools the author tends to fool the reviewer (same architecture family, e.g. Zen).

> **Failure mode SAIP-3 (reviewer/author collusion via shared model).** If author and reviewer are the same model family (the draft's whole pitch is the Zen family), "adversarial review" is two draws from one distribution. Real adversarial review needs *independent* models with *uncorrelated* failure modes — i.e., closed-lab models (OpenAI/Anthropic/Google), which the draft's philosophy (§6.3) explicitly rejects ("centralized AI labs … behind closed doors"). The ideology and the security requirement are in direct tension. You cannot get genuine ensemble diversity from a single open-weight family.

### 2.3 The Experience-Ledger feedback loop is a reward-hackable objective

§3.2: "the protocol *learns what kinds of improvements work*" by accumulating which proposals passed and "what outcomes resulted." This defines an implicit objective: *generate proposals that get accepted and produce good outcomes*. Any learned objective is gameable, and this one is gameable by **whoever controls the votes or the outcome metric**.

> **Failure mode SAIP-4 (governance-capture flywheel).** A whale (or cartel) that controls enough vote weight to pass proposals teaches the SAIP generator — via the Experience Ledger's accept/reject signal — *what the whale likes to approve*. The generator, optimizing for "proposals that get accepted," converges to **proposals that serve the whale**, dressed in the house style the ledger rewards. The protocol now auto-authors capture. This is worse than human-only governance, where at least the proposer's intent is legible. Here intent is laundered through "the protocol proposed it."

> **Failure mode SAIP-5 (outcome-metric Goodhart).** "What outcomes resulted" must be measured by some metric (TPS, fees, MEV, §3.1 telemetry). The generator learns to propose changes that *improve the measured metric* even at the cost of unmeasured health. Classic: optimize TPS by relaxing a safety check that doesn't show up in the telemetry until it fails catastrophically. The feedback loop has no term for "harm we didn't measure."

### 2.4 Telemetry is an attacker-influenceable input

SAIP generation is triggered by telemetry deviation (§3.1: TPS, finality, gas, uptime, slashing, economic velocity, MEV patterns). An adversary who can move telemetry can *steer what the protocol proposes*.

> **Failure mode SAIP-6 (telemetry injection → directed proposal).** Attacker spams a transaction pattern that inflates "MEV extraction" telemetry, reliably triggering an auto-SAIP "to mitigate MEV" — and (via SAIP-4) shapes that proposal toward a change the attacker wants (e.g., a sequencer/ordering change that *helps* the attacker). The attacker now has a **remote-trigger for the governance agenda**. This is a new capability that did not exist in human-authored ZIP governance, where setting the agenda requires social effort.

### 2.5 Provenance and accountability vacuum

Human ZIP authorship has accountability: a named author, a reputation, a social cost for proposing garbage. SAIP authorship is anonymous-by-construction ("the protocol proposed it"). This **launders responsibility** and removes the single most effective real-world filter on bad proposals: the proposer's reputation being on the line. The draft does not address who is accountable for a passed SAIP that causes loss.

### 2.6 Verdict — SAIPs

**Readiness 2/5.** As a *drafting assistant* that produces a ZIP skeleton a human author then owns, signs, and is accountable for — useful, low-risk, ship it. As an *autonomous authorship layer with a learning loop and a privileged path to the ballot* — it adds attack surface (SAIP-1, 2, 6), a capturable objective (SAIP-4, 5), correlated-reviewer blind spots (SAIP-3), and an accountability vacuum (§2.5), while its sole defense (multi-model review) is undercut by the project's own open-weight philosophy.

**To reach 3/5:** (a) require a *named human sponsor* who adopts, signs, and is slashable/reputation-bound for every SAIP that reaches the ballot — no anonymous auto-proposals; (b) rate-limit SAIPs hard (per-epoch cap, decoupled from telemetry triggers, to defeat SAIP-1/6); (c) remove the accept/reject *learning loop* on governance outcomes (or make the reward an explicit, audited, harm-inclusive objective — not "what got accepted"); (d) source adversarial review from *genuinely independent* model families even if that means using closed models for the red-team step, and disclose the tension with §6.3.

---

## 3. Compute Settlement Layers — adversarial analysis

### 3.1 The scheduler is the strong part

Credit where due: the per-node priority scheduler (Consensus > Training > Inference, with consensus preemption) is a real, buildable systems design, and "one settlement tx per epoch" is good for fee efficiency. Rated higher than the other two primitives largely on this. The problems are economic and at the seams, not in the core scheduling idea.

### 3.2 The "self-funding cycle" is circular reasoning

§4.2's cycle diagram is an *assumption presented as a mechanism*. Every arrow is fine except the one that's load-bearing: **"better protocol attracts more users → more inference demand."** Nothing in the design produces external inference demand; it is assumed. Strip that arrow and the cycle is just "GPUs cost money to run." 

> **Failure mode CS-1 (demand assumption).** If external inference demand is low (the default state for a new L1/L2 with no consumer product), Function-3 revenue is near zero. Then invariant 3 ("if inference revenue < operating cost, node reduces training duty until profitable") forces training duty toward zero — which kills Function-2, which kills the "improving the chain" leg, which the whole pitch rests on. The "self-funding R&D budget" only funds itself *if it already has paying customers*. That is the hard part, and it's outside the protocol. The draft's strongest rhetorical claim (§4.2, "training is not a cost center") is the least substantiated.

### 3.3 GPU-slot monopolization

§7.3 flags this and proposes "diminishing returns per additional GPU per operator." This defense is **trivially Sybil-defeated**.

> **Failure mode CS-2 (Sybil around per-operator caps).** "Per operator" requires an identity. GPU operators are pseudonymous keys. A large operator splits one farm into N pseudonymous "operators," each below the diminishing-returns knee. Per-operator diminishing returns provides zero protection without strong, costly identity (KYC, or large stake-per-identity that itself centralizes). The draft has no Sybil-resistance story for the "operator" abstraction. Stake-weighting reintroduces "richest wins"; KYC reintroduces a trusted registrar (centralization the project rejects).

> **Failure mode CS-3 (latency moat → consensus capture).** Consensus has the tightest latency budget (~100ms inference/block, §4.1). The lowest-latency operators (best GPUs, best colocation, best network) win the most consensus slots → most block rewards → buy more/better GPUs → tighter latency moat. This is the standard PoW/PoS centralization spiral, now with a *higher* barrier to entry (you need consensus-grade GPUs, not just stake or hash). Triple-use *raises* the minimum viable node cost, which is centralizing, not democratizing.

> **Failure mode CS-4 (slot-allocation MEV).** The scheduler decides, per node, when to do consensus vs. training vs. inference. A sophisticated operator games *its own* schedule: do consensus exactly when block rewards (incl. MEV) are high, dump to inference/training when low, and time training submissions to maximize the 3x multiplier. The "single settlement tx per epoch" hides this intra-epoch optimization from on-chain scrutiny. Honest "fair-share" nodes earn less; the spiral feeds CS-3.

### 3.4 The preemption channel

Invariant 1 ("consensus always preempts training") is correct for safety but creates two problems:

> **Failure mode CS-5 (preemption as covert channel / training poison).** Whether and when training is preempted leaks consensus timing/load to the training layer. More seriously, frequent preemption corrupts training: a node that is preempted mid-step submits partial/stale gradients. Invariant 4 ("quality gates training") is supposed to catch bad gradients, but *preemption-induced* staleness is correlated across all nodes (they all get preempted at block boundaries), so it looks like a consistent signal, not noise — quality scoring may *accept* systematically biased gradients. Consensus load thus silently biases the trained model.

> **Failure mode CS-6 (griefing via forced preemption).** An adversary who can inflate consensus work (CC-5 context bombs, or just high tx volume at chosen times) forces honest nodes to preempt training repeatedly, denying them the 3x training reward and the inference revenue, while the adversary (who isn't training) is unaffected. Targeted economic griefing of competitors.

### 3.5 Verdict — Compute Settlement

**Readiness 3/5.** The scheduler and settlement-accounting engineering is the most ready thing in the paper. The economics are the weak point: a circular demand assumption (CS-1), no Sybil-resistant operator identity (CS-2), a centralizing latency moat that triple-use *worsens* (CS-3), intra-epoch slot MEV hidden by epoch-batching (CS-4), and a preemption channel that can poison training (CS-5) and be weaponized for griefing (CS-6).

**To reach 4/5:** (a) drop the "self-funding" claim or back it with a concrete external-demand source and a fallback that doesn't zero out training when demand is low; (b) replace per-operator caps with a Sybil-resistant scheme and state its centralization cost honestly; (c) decouple training-gradient acceptance from block-boundary preemption (e.g., only accept gradients from *uninterrupted* training windows); (d) publish per-slot allocation commitments, not just epoch settlement totals, so CS-4 is auditable.

---

## 4. The five economic invariants — do they hold?

Restating §4.4 and stress-testing each, then checking *joint* consistency (the draft only argues them individually).

| # | Invariant | Holds in isolation? | Breaks under… |
|---|---|---|---|
| 1 | Training never starves consensus (consensus preempts) | **Yes** (by construction) | Creates CS-5 (training poison) and CS-6 (griefing). Safe but has side effects. |
| 2 | Inference never starves training (min 20% training duty) | **Conditionally** | Directly contradicted by invariant 3 under low revenue (see below). |
| 3 | Revenue covers compute (reduce training if unprofitable) | **Yes, but** | This is the invariant that *breaks invariant 2*. |
| 4 | Quality gates training (only quality-scored gradients earn) | **Partially** | Preemption-correlated staleness (CS-5) can pass quality gates; quality scoring is itself an attackable ML model. |
| 5 | Settlement is atomic (one tx/epoch) | **Yes** | But hides intra-epoch slot MEV (CS-4); atomicity ≠ fairness. |

### 4.1 The 2-vs-3 contradiction (the important one)

- Invariant 2: training duty cycle ≥ 20% per epoch.
- Invariant 3: if inference revenue < operating cost, *reduce training duty until profitable*.

Under low inference demand (the default for a young network, CS-1), invariant 3 instructs the node to push training duty **below 20%** — possibly to 0% — to stop bleeding money. That **violates invariant 2**. These two invariants are **jointly unsatisfiable in exactly the economic regime the network starts in.** The draft never reconciles them. Either:
- the network subsidizes training (then it's *not* self-funding, contradicting §4.2), or
- training stops in the bootstrap phase (then the "improving the chain" leg is dead when it's needed most), or
- the 20% floor is not actually a floor (then invariant 2 is marketing, not an invariant).

### 4.2 Invariant 4's quality oracle is itself an attack surface

"Quality scoring (PoAI existing mechanism)" is invoked as if it were a trusted constant. It is an ML-based scorer; it can be gamed (submit gradients optimized to score well rather than to help), it can be poisoned (CS-5), and it has its own false-positive/negative rates that the invariant's binary framing hides.

### 4.3 Verdict — invariants

**2 hold cleanly (1, 5), 1 holds-with-bad-side-effects (3), 1 is partial (4), and the set is jointly inconsistent (2 vs 3) in the bootstrap regime.** "Atomic settlement" (5) is true but oversold — atomicity prevents partial-payment bugs, it does *not* prevent the intra-epoch gaming (CS-4) that the atomic batch conceals.

---

## 5. Do the draft's own "open question" answers hold? (§7)

The draft proposes a fix for each open question. Skeptical audit:

| OQ | Proposed fix | Holds? | Why / why not |
|---|---|---|---|
| 7.1 Model selection (circular dep) | (left open) | n/a | Genuinely open. But note: if the consensus model *is* the trained model, every training step changes the consensus function → re-introduces the determinism break (CC-2) at the protocol level. Must use a *separately versioned, governance-pinned* model. The draft should commit to this. |
| 7.2 Adversarial SAIPs | Multi-model adversarial review | **No** | SAIP-3: shared open-weight family → correlated blind spots. Genuine independence requires closed models, contradicting §6.3. Review is *worse* than authoring at the cross-proposal economic reasoning that matters (SAIP-2). |
| 7.3 Slot monopolization | Diminishing returns per operator | **No** | CS-2: Sybil-trivial without costly identity. "Per operator" is undefined over pseudonymous keys. |
| 7.4 Trace privacy / MEV | FHE-encrypted traces, delayed decryption | **Partially** | CC-6: delay only delays the leak; stable heuristics make old traces predictive. FHE on the *verification/comparison* path (Tier-2 must compare encrypted traces) is not consensus-latency-practical in 2026. The fix addresses storage privacy, not the strategic leak. |
| 7.5 Bootstrapping | Seed Experience Ledger from Zen Agentic Dataset (12B tokens) | **Partially** | Seeding *content* is fine. But it does not bootstrap the thing that matters — the *outcome* feedback (which proposals actually worked on *this* chain). And a public seed dataset is a public attack manual: adversaries read the seed to learn exactly what the protocol's priors are (SAIP-2/6 become easier). |

**Net:** 3 of 5 proposed answers do not hold (7.2, 7.3 fail; 7.4, 7.5 partial), and 7.1 is open with a hidden determinism trap. The "open questions" section is honest about *what* is open but optimistic about whether the one-line fixes close the gaps. They mostly don't.

---

## 6. The safe subset (what survives the red team)

Stripping everything that fails, here is the version that is actually defensible and shippable in 2026:

1. **Cognitive Consensus → Cognitive *Sidecar*.** Validators may emit reasoning traces. Traces are **advisory, non-binding, non-slashable**. Where a trace makes a *checkable* claim ("I ordered tx X before Y," "I flagged address Z"), bind only on the *checkable action* (a decidable fact verified against the actual block), never on the prose's "coherence." No embedding-similarity consensus predicate. This removes CC-1, CC-2, CC-3, CC-4 and prices CC-5 with explicit verification gas. Keeps the useful per-block AI commentary.

2. **SAIPs → AI-Drafted, Human-Owned ZIPs.** The model drafts; a **named human sponsor** adopts and signs (accountability restored, §2.5). Hard per-epoch rate limit decoupled from telemetry triggers (kills SAIP-1, SAIP-6). **No outcome-learning loop** that optimizes for "what gets accepted" (kills SAIP-4, SAIP-5). Adversarial review uses *independent* model families with the §6.3 tension disclosed.

3. **Compute Settlement → Triple-Use Scheduler, honest economics.** Keep the scheduler (it's good). Drop "self-funding" as a claim; state the external-demand dependency (CS-1) explicitly with a no-demand fallback that doesn't zero training. Decouple gradient acceptance from preemption boundaries (CS-5). Publish per-slot commitments (CS-4). Replace per-operator caps with an honest Sybil-resistance discussion (CS-2).

In this subset, all three "primitives" become **useful, non-consensus-binding layers on top of an existing safe consensus (Quasar)**. That is a real, shippable contribution. It is *not* the paper's headline claim that "the LLM becomes the operator of the blockchain." The honest framing is: **the LLM becomes a well-instrumented, accountable advisor to the blockchain.**

---

## 7. Readiness scorecard and recommendation

| Item | Readiness (0–5) | Ship as research? | Ship to mainnet as designed? |
|---|---|---|---|
| Cognitive Consensus (as binding) | 2 | Yes | **No** — category error + determinism break |
| Cognitive Sidecar (advisory) | 4 | Yes | Yes (with verification gas) |
| SAIPs (autonomous + learning loop) | 2 | Yes | **No** — capture flywheel + accountability vacuum |
| AI-drafted human-owned ZIPs | 4 | Yes | Yes (with named sponsor + rate limit) |
| Compute Settlement scheduler | 3 | Yes | Pilot-only — fix CS-1/CS-5/CS-4 first |
| 5 economic invariants | partial | Yes | **No** — invariants 2 & 3 jointly inconsistent at bootstrap |

**Recommendation to Zoo/Lux:** Publish Thinking Chains as a research direction with the headline claim **softened from "operator" to "accountable advisor."** Implement the §6 safe subset. Do **not** let any reasoning trace gate finality and do **not** give any SAIP an auto-merge path until: (i) a decidable correctness oracle replaces "coherence"; (ii) determinism is solved via canonical integer kernels or the trace is explicitly non-slashable; (iii) governance has Sybil-resistant identity and SAIPs have named human accountability; (iv) the 2-vs-3 invariant contradiction is resolved with a concrete bootstrap subsidy model.

The prototype specs in `04-prototype-specs.md` are written to the **safe subset**, with the unsafe-as-designed behaviors gated behind explicit `BINDING=false` / advisory flags so the system can be built and measured without putting consensus at risk.
