# Thinking Chains

**LLM-native consensus for self-evolving blockchain protocols — a research direction for Zoo Labs Foundation / Lux Network.**

Thinking Chains add a *reasoning layer* on top of Proof of AI (ZIP-0419 / LP-302): an LLM that reasons about each block, drafts protocol improvements for humans to ratify, and shares GPU compute across consensus, training, and inference. The honest framing — after adversarial review — is **not** "the LLM operates the chain" but "the LLM becomes a well-instrumented, accountable *advisor* to the chain," layered on a safe deterministic consensus (Quasar), with every consensus-unsafe behavior gated behind a non-binding flag.

> **Author:** Pattermesh (KCOLBCHAIN Research) · **Status:** DRAFT for Zoo/Lux review (@abhicris) · **Date:** 2026-06-30

---

## The three primitives

1. **Cognitive Consensus** — Validators emit a structured reasoning trace (`CognitiveAttestation`) per block, verified by a three-tier scheme (structural/assertion O(1) → semantic sampling O(k) → dispute re-execution O(n)) that never universally re-runs the model. *Binding only on decidable assertions; the prose is advisory.*
2. **SAIPs (Self-Authoring Improvement Proposals)** — The protocol drafts improvement proposals in standard ZIP format from its own telemetry, gated through heterogeneous adversarial review → Lean/EasyCrypt formal checks → shadow-fork simulation → human governance with a named sponsor. *The machine proposes; humans decide; the machine never self-deploys.*
3. **Compute Settlement Layers** — A per-node GPU scheduler + atomic on-chain settlement so one GPU serves consensus, training, and inference, settling all three reward streams in one transaction per epoch. *"Self-funding" is stated as a demand-dependent target, not a guarantee.*

---

## Read in this order

| # | Document | What it is |
|---|---|---|
| **00** | [`00-thinking-chains.md`](./00-thinking-chains.md) | **The paper (v2).** Polished, grounded, red-teamed. Start here. |
| 01 | [`01-open-questions-resolved.md`](./01-open-questions-resolved.md) | Resolution memo for all 7 open questions (5 from the draft + 2 latent), with tradeoffs and ZIP/LP integration. |
| 02 | [`02-grounding.md`](./02-grounding.md) | Vaporware audit — every component, ZIP/LP number, repo, and number checked against the live `luxfi`/`zoo-apps`/`zenlm`/`hanzoai` orgs. |
| 03 | [`03-red-team.md`](./03-red-team.md) | Adversarial review. Rates each primitive 0–5, finds the load-bearing failures, and defines the **safe subset**. |
| 04 | [`04-prototype-specs.md`](./04-prototype-specs.md) | Reference-implementation skeletons for the four *new* (delta) components, written to the safe subset (`BINDING=false` default). |

## The four ZIPs (working drafts)

Written for internal review as `zip-0420`…`zip-0423`. **These numbers are placeholders** — on upstream submission they must be renumbered to **ZIP-0435–0438** (0420–0423 are already allocated; see `01`/`02` Q7 and the paper §5.3). Canonical ZIP repo is `zoo-apps/ZIPs`, *not* `zoo-labs/zips`.

| Working draft | Title | Upstream number | Type |
|---|---|---|---|
| [`zip-0420-cognitive-consensus.md`](./zip-0420-cognitive-consensus.md) | Cognitive Consensus — LLM-Verified Block Attestations | → ZIP-0435 | Standards Track |
| [`zip-0421-saips.md`](./zip-0421-saips.md) | Self-Authoring Improvement Proposals (SAIPs) | → ZIP-0436 | Meta |
| [`zip-0422-compute-settlement.md`](./zip-0422-compute-settlement.md) | Compute Settlement Layers — Triple-Use GPU Economics | → ZIP-0437 | Standards Track |
| [`zip-0423-reasoning-trace-verification.md`](./zip-0423-reasoning-trace-verification.md) | Reasoning Trace Verification via Semantic Embeddings | → ZIP-0438 | Standards Track |

---

## What's real vs. new (from the grounding audit)

**Already shipped / Final (reused, not built here):** PoAI (ZIP-0419/LP-302), HLLM (ZIP-0418) + Training-Free GRPO (ZIP-0421, externally anchored at arXiv:2510.08191 / 2512.24615), DSO + 7680-dim embeddings (ZIP-0410), Experience Ledger (ZIP-0401/0404/0410), decentralized training / Zoo Gym (ZIP-0407→0434), Quasar (`luxfi/consensus`, incl. a real `ai/` autonomous-decision package), Zen edge models (`zen-eco`/`zen-eco-thinking`/`zen-nano`, Apache-2.0), FHE/TEE (LP-013/LP-302/LP-020, `luxfi/fhe`), Lean + EasyCrypt proofs, Zoo L2 EVM (ZIP-0015).

**Genuinely new (the delta — four components, a composition layer, *not* a new chain):** the `CognitiveAttestation` tx type + trace schema, the three-tier verification driver, the SAIP generation+review pipeline, and the triple-use GPU scheduler + unified settlement.

**Corrections folded in:** Zen Agentic Dataset is agentic/coding context at size class 10B–100B tokens (721M+ per repo description), *not* "12B tokens of blockchain engineering"; consensus correctness is proved in Lean (EasyCrypt covers crypto primitives); Quasar's toggleable layer is the *signing mode* (finality drivers are specifically Ray/Field); cite `zoo-apps/ZIPs`.

## Readiness (from the red team)

| Primitive | As binding consensus | As the safe subset |
|---|---|---|
| Cognitive Consensus | **No** — coherence isn't a decidable predicate; determinism breaks on GPU FP | **4/5** as advisory Sidecar (bind only on decidable assertions) |
| SAIPs | **No** — capture flywheel + accountability vacuum | **4/5** as AI-drafted, human-owned ZIPs (named sponsor, rate-limited) |
| Compute Settlement | Pilot-only | **3/5** with honest economics (no "self-funding" guarantee; bootstrap subsidy) |

---

*All claims are audited in `02-grounding.md` (verification run 2026-06-30). Contact: research@kcolbchain.com · github.com/kcolbchain*
