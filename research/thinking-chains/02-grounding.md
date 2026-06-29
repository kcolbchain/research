# Thinking Chains — Grounding & Vaporware Audit

**Companion to:** `thinking-chains-draft.md`
**Method:** Every component, ZIP/LP number, repo, and quantitative claim in the draft was checked against the live GitHub orgs **luxfi**, **zoo-labs**, **zoo-apps**, **zenlm**, **hanzoai** (via authenticated `gh`) plus web search for external anchors. Verification run: 2026-06-30.
**Verdict legend:** ✅ Real/shipped · 🟡 Real but mischaracterized / number off / partly stubbed · 🔴 Vaporware or wrong as cited · ⚪ External (third-party) but real.

---

## 0. The single most important finding (read first)

The draft's cited ZIPs are **real and Final**, but they live in the **`zoo-apps/ZIPs`** repository — **not** `zoo-labs/zips`. The `zoo-labs/zips` repo is the *governance website* (Next.js) and contains only `zip-0001`, `zip-0100`, `zip-0200` as samples. Anyone trying to verify the paper by looking at `zoo-labs/zips` will wrongly conclude the ZIPs don't exist. The canonical AI-range ZIPs (0400–0434, all with full front-matter, `status: Final`) are in `zoo-apps/ZIPs/ZIPs/`.

- Canonical ZIP repo: `https://github.com/zoo-apps/ZIPs` → `ZIPs/zip-04xx-*.md`, master `INDEX.md`.
- Website repo (sparse): `https://github.com/zoo-labs/zips`.

**Action for the paper:** cite `zoo-apps/ZIPs` as the source of truth.

---

## 1. Core ZIPs the paper builds on

### ZIP-0419 — Proof of AI (PoAI) — ✅ REAL / Final
- File: `zoo-apps/ZIPs/ZIPs/zip-0419-proof-of-ai-consensus.md`, `status: Final`, authors Antje Worring + Zach Kelling, `created: 2024-06-15`, `amended: 2026-06-24`.
- Canonical cross-org spec: **`luxfi/LPs/LP-302-proof-of-ai-compute-binding.md`** (`status: Final`), which explicitly lists `mirrors: HIP-0901 (Hanzo)` and `references: ZIP-0419 (Zoo mirror)`. So ZIP-0419 ↔ LP-302 ↔ HIP-0901 is a real, three-org mirrored standard.
- Substantiates: validators earn rewards for useful AI work (inference/embedding/training/data-sharing); TEE attestation + probabilistic verification + multi-party quality scoring; `f < n/3` BFT; Sybil resistance; 100-node testnet at **847 TPS / 2.3 s finality / 94% useful-work**.
- Implementations referenced in front-matter: `luxfi/standard contracts/ai`, `luxfi/crypto/poi` (Go verifier), `hanzo/engine poi.rs` (prover). Repo: `github.com/zooai/poai`.
- Papers: `zoo-labs/papers/poai-consensus.tex`, `zoo-poai-consensus.tex`; Lean proof `zoo-labs/proofs/lean/Consensus/PoAI.lean`.
- **Draft's use is accurate.** The draft says PoAI validators "already run GPU inference," "already require validators to run inference," and provide TEE attestation — all confirmed.
- **One nuance:** ZIP-0419's 2026 amendment notes the 2024 proofs "attest to authorship and agreement, not to the computation itself," closed by the LP-302 "Native Compute-Binding" (Freivalds-style + PQ proof). The draft's Q3 reliance on "compute-binding" is therefore well-founded.

### ZIP-0410 — Decentralized Semantic Optimization (DSO) — ✅ REAL / Final
- File: `zoo-apps/ZIPs/ZIPs/zip-0410-decentralized-semantic-optimization.md`, `status: Final`, `originated: 2023-09`. Repo `github.com/zooai/dso-protocol`.
- Substantiates: semantic-gradient sharing, differential privacy (ε,δ), Byzantine-robust aggregation, immutable provenance ledger.
- **7680-dim embedding space — ✅ REAL.** Heavily documented: `zoo-labs/papers/embedding-7680.tex` (Zen Reranker, native 7680-dim, BitDelta 31.87× compression, 68.4 MTEB, 94.7% Recall@5), `zoo-labs/proofs/lean/AI/Embedding.lean`, `experience-ledger-dso.tex` (`e.emb ∈ ℝ^7680`). The draft's "7680-dimensional embedding space (DSO's existing infrastructure)" and "cosine similarity below threshold τ" verification are directly supported.
- **Note:** the 7680-dim work also carries its own ZIP number **ZIP-0420** (`zip-0420-7680-dimensional-embeddings.md`) — relevant to the numbering collision (§5).

### ZIP-0407 — Decentralized AI Training Architecture — ✅ REAL / Final
- File: `zoo-apps/ZIPs/ZIPs/zip-0407-decentralized-ai-training-architecture.md`, `status: Final`, `originated: 2022-11`. Repo `github.com/zooai/gym`.
- Substantiates: heterogeneous-node training, Byzantine fault tolerance, proportional verified-compute rewards; precursor to Zoo Gym.
- **"Gym Protocol" (draft §5.1) — ✅ REAL.** Consolidated in **ZIP-0434** (`zip-0434-decentralized-training-infrastructure.md`): "Zoo Gym … unifies DSO (ZIP-0410), PoAI (ZIP-0419), federated learning (ZIP-0424), and GRPO (ZIP-0421)." Lean proof `zoo-labs/proofs/lean/AI/Gym.lean`. The draft's Compute-Settlement "training slot" maps cleanly onto this.

### ZIP-0418 — Hamiltonian LLMs (HLLM) — ✅ REAL / Final
- File: `zoo-apps/ZIPs/ZIPs/zip-0418-hamiltonian-large-language-models.md`, `status: Final`, `originated: 2024-06`. Repo `github.com/zooai/hllm`.
- The draft cites "HLLM-TF-GRPO." This resolves to **two** real ZIPs + papers, not one citation:
  - ZIP-0418 (HLLM) — the frozen-weights / energy-conserving framework.
  - ZIP-0421 (`zip-0421-training-free-preference-optimization.md`) — the Training-Free GRPO mechanism.
  - Paper `hllm-training-free-grpo.tex` (Oct 2025) listed in `zoo-labs/ngo` Research page.
- **The headline numbers are REAL and externally anchored:**
  - `zoo-labs/papers/PAPERS_README.md`: "99.8% cost reduction ($18 vs $10,000+)", "100× data efficiency (100 vs 10,000+ examples)", AIME24 table: Traditional FT 80.0% / $10,000+ vs **Training-Free GRPO 82.7% / $18 / 100 examples**.
  - These match the **external** literature: ⚪ **arXiv:2510.08191** ("Training-Free Group Relative Policy Optimization") and ⚪ **arXiv:2512.24615** (Tencent "Youtu-Agent") report the identical $18 vs $10,000, +2.7% AIME, 82.7% AIME24 / 73.3% AIME25 on DeepSeek-V3.1-Terminus. So the Zoo framing is built on real, citable third-party results — not fabricated. `zenlm/zen-family/LLM.md` itself cites "Youtu-Agent (arXiv:2512.24615)."
- **Draft's claim "0.2% of the cost" / "frozen models that learn via context" — ✅ accurate** (0.2% ≈ the inverse of the ~99.8% reduction; HLLM/DSO are explicitly context-space not parameter-space).
- 🟡 Minor: the draft also says HLLM achieves "superior results at 0.2% of the cost." The papers say "comparable or superior" and "+2.7% accuracy" — i.e. modestly superior on AIME, comparable elsewhere. Fine to keep but don't overstate.

### Experience Ledger — ✅ REAL (spec + Lean proof)
- `zoo-labs/zoo-papers-site/public/papers/experience-ledger-dso.tex`: content-addressable semantic-advantage library, Merkle verification, IPFS/Arweave storage, DAO governance, "0.2% of the cost" of fine-tuning. Lean: `zoo-labs/proofs/lean/AI/ExperienceLedger.lean` + `lean/Token/ExperienceLedger.lean`. Originated 2021 ("zoo-experience-ledger"), formalized 2025.
- The draft's "Experience Ledger already stores verifiable reasoning artifacts" and the SAIP-learning loop are well-grounded. Built on ZIP-0401 (Persistent Semantic Memory) + ZIP-0404 (Content-Addressable Semantic Memory), both Final.

---

## 2. Lux / Quasar consensus

### Quasar / `luxfi/consensus` — ✅ REAL / actively developed
- Repo `https://github.com/luxfi/consensus` — "Quasar consensus engine: Go implementations…", `pushedAt: 2026-06-29`, Go 1.26, not archived. Public web confirms (lux.link, GitHub).
- Documented sub-protocols (from the live README): **Photon** (committee selection, Fisher-Yates, luminance-weighted reputation), **Wave** (threshold voting + FPC), **Focus** (confidence accumulation → local finality), **Nova** (linear-chain mode), **Nebula** (DAG mode), **Prism**, **Horizon**, **Flare**, **Ray** (linear finality driver), **Field** (DAG finality driver), **Quasar** (BLS + Pulsar + ML-DSA threshold signing).
- Signing modes independently toggleable: BLS-only / BLS+ML-DSA (FIPS-204) / BLS+Pulsar (Ring-LWE). Web sources also describe BLS+Ringtail dual finality, ≤50ms PQ-round attack window. PQ family: Pulsar (ML-DSA), Corona (R-LWE), Magnetar (SLH-DSA) per `luxfi/quasar`.
- 🟡 **Draft mischaracterization to fix:** the draft says "Quasar consensus already supports **pluggable finality drivers**." More precisely, the finality **drivers** are specifically **Ray** (linear) and **Field** (DAG); what's pluggable/toggleable is the **signing mode** (BLS/ML-DSA/Pulsar), and protocols compose via sub-protocol packages. Cognitive Consensus should attach as an **attestation-verification stage** layered on the existing block path — it is *not* a finality-driver swap. Reword for accuracy; the underlying composability claim survives.

### luxfi/consensus `ai/` package — ✅ REAL (and a stronger basis for the paper than the draft realizes) / 🟡 model is an interface
- Directory `luxfi/consensus/ai/` contains real Go: `ai.go`, `agent.go`, `engine.go`, `models.go`, `specialized.go`, `modules.go`, plus extensive `*_test.go`.
- `ai.go` defines `SimpleAgent`, `BasicModel interface { Decide(ctx, question, data) (*SimpleDecision, error) }`, and `SimpleDecision{ Action, Confidence, Reasoning, Data, Timestamp }`, plus `State{ chains, disputes, upgrades, security }` and `Performance{ TPS, Latency, FaultRate, … }`.
- `ai/README.md`: "AI Package — Cross-Chain Decentralized AI Computation … Autonomous Upgrades: Blockchain can upgrade itself based on AI consensus; Fork Arbitration; Dispute Resolution; Security Response; Cross-Chain Coordination."
- **This directly substantiates the *mechanism* behind both Cognitive Consensus and SAIPs** — autonomous-upgrade + reasoning-bearing decision objects exist in shipped Go today. The draft under-cites this; it should reference `luxfi/consensus/ai`.
- 🟡 **Caveat (not vaporware, but be precise):** `BasicModel` is an **interface**. The concrete production operator-LLM (e.g. wiring Zen-Eco-Thinking behind `Decide`) and the full telemetry→SAIP pipeline are the genuinely new work. The plumbing is real; the end-to-end self-evolution loop is not yet a deployed product. Calibrate paper claims accordingly (see resolved-questions Q7).

### Formal verification — ✅ REAL, but attribution needs splitting
- 🟡 **Draft says "EasyCrypt proofs in threshold/consensus."** Reality:
  - **EasyCrypt** proofs are real but live in the **crypto** repos, not `luxfi/consensus`: `luxfi/crypto/mlkem/proofs/easycrypt/` (MLKEM correctness, IND-CCA2, wire-format), `luxfi/threshold/protocols/bls/proofs/easycrypt/`, `luxfi/pulsar/proofs/easycrypt/`, `luxfi/fhe/proofs/easycrypt/` (TFHE), `luxfi/magnetar` (EC smoke checks), plus a `lean-easycrypt-bridge.md`. So "EasyCrypt in **threshold**" ✅; "EasyCrypt in **consensus**" 🔴 (consensus correctness is proved in **Lean**, via `zoo-labs/proofs/lean/Consensus/PoAI.lean` and the Lux side, not EasyCrypt).
  - **Reword:** "EasyCrypt proofs for cryptographic primitives (ML-KEM, BLS-threshold, Pulsar, TFHE) and Lean proofs for consensus/AI correctness." Both are real; the draft conflated them.

---

## 3. Zen model family (`zenlm`)

### ✅ REAL — the edge models the draft names exist
- `zenlm/zen-eco` — "4B efficient models — instruct, thinking, and agent variants" (public). `zen-eco-instruct` README confirms **4B params, 32K context, Apache-2.0**.
- `zenlm/zen-eco-thinking` — "Chain-of-thought reasoning model" (public) — the right operator model for Cognitive Consensus traces (Q1).
- `zenlm/zen-nano` — "Zen Nano v1.0 — Ultra-lightweight edge AI model" (public). Draft cites "Zen-Nano 0.6B" — repo exists; the exact 0.6B size is in the model card (size class consistent with an ultra-light edge model; not contradicted).
- `zenlm/zen4-mini` — "Zen4 Mini 8B" (public, **archived**). 🟡 Draft cites "Zen-Mini 8B" as a consensus-LLM option — the 8B model exists but is **archived**, so prefer `zen-eco` 4B / `zen-nano` for forward-looking text.
- Canonical model registry: `zenlm/models` (`src/models.ts`, `families.ts`, `types.ts`) — "single source of truth." Also `zenlm/zen5` (MoDE, 3.1T params) and `zenlm/grpo`.
- Draft's "edge-deployable open-weight models purpose-built for blockchain contexts" — ✅ the edge models are real and Apache-2.0; "purpose-built for blockchain contexts" is **aspirational framing** (the models are general-purpose; no blockchain-specific Zen variant exists). Soften to "edge-deployable open-weight models suitable for embedding in consensus."

### Zen Agentic Dataset — ✅ REAL / 🔴 token figure wrong
- `zenlm/zen-agentic-dataset` (private), mirror `hanzoai/zen-agentic-dataset-private`. README front-matter: `size_categories: 10B<n<100B`, `license: commercial`, tags agentic/coding/claude.
- Repo **descriptions** state **"721M+ tokens"** (`zen-agentic-dataset`) and **"671M+ tokens"** (`zen-claude-agentic-full`).
- 🔴 **Draft claims "12B tokens of blockchain engineering context."** Two problems: (1) the cited token count (12B) matches neither the 721M description nor a single concrete number — at best it falls inside the loose `10B<n<100B` size class; (2) the dataset is **agentic/coding/Claude-conversation** data, **not** "blockchain engineering" data. **Fix:** cite either "size class 10B–100B tokens" or "721M+ tokens" with the repo link, and describe it as agentic/coding context, not blockchain-specific.

---

## 4. Hanzo Brain & related

### Hanzo Brain (`hanzoai/brain`) — ✅ REAL / 🟡 not what the draft implies
- Repo `https://github.com/hanzoai/brain` (private), TypeScript, `pushedAt: 2026-06-29`. Description: "Single binary brain … SQLite + FTS5 + zero-LLM typed edges. One `~/.hanzo/brain/brain.db` shared by TS/Python/Rust/Go runtimes."
- ✅ It exists. 🟡 But it is a **local knowledge/memory store (SQLite+FTS5, explicitly "zero-LLM")**, i.e. a developer-tooling memory layer — **not** a consensus-embedded reasoning engine. The draft's References list "[Brain] Hanzo Brain, hanzoai/brain" without a specific load-bearing claim, which is fine, but **do not** imply Brain is the operator-LLM or the Experience Ledger. The on-chain Experience Ledger is the Zoo DSO artifact (§1); Hanzo Brain is a separate local-agent memory product.
- Related real Hanzo infra that *is* relevant: `hanzoai/engine` (Rust inference engine), `hanzoai/llm` (router/MCP), `hanzoai/net` `hanzo-hmm` (Hamiltonian Market Maker — real Rust, prices heterogeneous compute; underpins the Compute-Settlement economics conceptually via HIP-004).

### HLLM lineage in Hanzo — ✅ REAL
- `hanzoai/universe/rust-sdk/crates/hanzo-hmm/README.md` references "HLLM (Hamiltonian Hidden-Markov LLM)"; `hanzoai/financials/papers/.../hanzo-hamiltonian-markets.tex` and `hanzoai/papers/hanzo-hmm/` formalize the Hamiltonian invariant `compute · demand = invariant`. Cross-org consistent with ZIP-0418.

---

## 5. Numbering & citation errors in the draft (must-fix)

### 🔴 ZIP-0420–0423 are ALL already allocated
The draft (§5.3) proposes:
| Draft proposes | Reality in `zoo-apps/ZIPs` |
|---|---|
| ZIP-0420 Cognitive Consensus | **TAKEN** — `zip-0420-7680-dimensional-embeddings.md` |
| ZIP-0421 SAIPs | **TAKEN** — `zip-0421-training-free-preference-optimization.md` |
| ZIP-0422 Compute Settlement | **TAKEN** — `zip-0422-computer-use-framework.md` |
| ZIP-0423 Reasoning-Trace Verification | **TAKEN** — `zip-0423-privacy-preserving-ai-training.md` (FHE) |

The AI range (0400–0499) is currently populated **through ZIP-0434**. **Next free block = ZIP-0435.** Recommended renumber: 0435 Cognitive Consensus / 0436 SAIPs / 0437 Compute Settlement / 0438 Reasoning-Trace Verification. (Also note the draft's own Q4 proposes FHE traces — and ZIP-0423 *is already the FHE ZIP*, so the paper should cite 0423 rather than collide with it.)

### ✅ ZIP-0015 (Zoo L2 Chain Architecture) — CORRECT
Draft §5.1 cites "Zoo L2 EVM (ZIP-0015)." Verified: `zoo-apps/ZIPs/ZIPs/zip-0015-zoo-l2-chain-architecture.md` exists, referenced by ZIP-0016 (tokenomics) and multiple papers. Good citation.

### 🟡 "ZIP-0000's AI range (400-499)" — CORRECT in spirit
`INDEX.md` confirms the range map: 0400–0499 = AI/ML. ZIP-0000 is the foundation/roadmap ZIP. Fine to cite.

### 🟡 References section author/date hygiene
- "[ZIP-0419] Z. Kelling, 2024" — authorship is **Antje Worring + Zach Kelling** (both listed in front-matter); year 2024 is the origination, amended 2026. Add Worring.
- "[HLLM-TF-GRPO] … Hanzo Industries / Zoo Labs Foundation, Oct 2025" — date ✅ (Oct 2025). Add external anchors arXiv:2510.08191 and arXiv:2512.24615 for the benchmark numbers, since those are the verifiable source.
- "[Quasar] luxfi/consensus" ✅. "[Zen] zenlm.org" ✅ (org + zenlm.org referenced in READMEs). "[Brain] hanzoai/brain" ✅ exists (but see §4 caveat).

---

## 6. Quantitative claims — verification table

| Claim in draft | Verdict | Source |
|---|---|---|
| PoAI: validators run GPU inference for blocks | ✅ | ZIP-0419 / LP-302 |
| HLLM: improvement via context, 0.2% of cost | ✅ | ZIP-0418, experience-ledger-dso.tex, PAPERS_README |
| $18 vs $10,000+; 100 vs 10,000 examples; 99.8% reduction | ✅ | PAPERS_README + ⚪ arXiv:2510.08191 / 2512.24615 |
| 82.7% AIME24 (Training-Free GRPO) | ✅ | PAPERS_README + ⚪ arXiv:2512.24615 |
| 847 TPS / 2.3 s finality / 94% useful work | ✅ | ZIP-0419 / poai-consensus.tex (100-node testnet) |
| 7680-dim embeddings, 31.87× BitDelta, 94.7% Recall@5 | ✅ | embedding-7680.tex, Embedding.lean |
| Zen-Eco 4B (~100ms/block inference) | 🟡 | model real (4B/32K); 100ms is a *design assumption*, not a measured guarantee |
| Zen-Nano 0.6B, Zen-Mini 8B edge models | 🟡 | zen-nano real; zen4-mini 8B real but **archived** |
| Zen Agentic Dataset "12B tokens, blockchain engineering" | 🔴 | real dataset; descriptions say 721M+ tokens / size class 10B–100B; content is agentic/coding, not blockchain |
| Quasar "pluggable finality drivers" | 🟡 | Quasar real; finality drivers are specifically Ray/Field; signing modes are the toggleable layer |
| "EasyCrypt proofs in threshold/consensus" | 🟡 | EasyCrypt ✅ in crypto/threshold/pulsar/fhe; consensus correctness is in **Lean**, not EasyCrypt |
| FHE-encrypted delayed-decryption traces | ✅ (feasible) | LP-013 (TFHE/GPU, F-Chain), luxfi/fhe repo, ZIP-0423 — all real |
| TEE attestation framework (PoAI) | ✅ | LP-302; institutional TEE composition LP-020 |
| Gym Protocol (decentralized training) | ✅ | ZIP-0407 → ZIP-0434, github.com/zooai/gym, Gym.lean |

---

## 7. Vaporware flags — what the paper must NOT assert as shipped

1. 🔴 **The self-evolution loop (telemetry → SAIP → multi-gate → vote → deploy) does not run in production.** The decision *plumbing* exists (`luxfi/consensus/ai`, real Go), but the end-to-end autonomous-proposal pipeline is new work. Keep "LLM proposes, humans decide"; don't imply the loop is live.
2. 🔴 **No "blockchain-specific" Zen model exists.** Zen edge models are general-purpose Apache-2.0. "Purpose-built for blockchain contexts" is aspirational.
3. 🔴 **"12B tokens of blockchain engineering context" is incorrect** on both count (721M+ / size-class 10B–100B) and content (agentic/coding, not blockchain). Must fix.
4. 🟡 **`CognitiveAttestation` tx type, the three-tier trace verifier, the GPU scheduler, and unified settlement accounting are all NEW** — the draft's §5.2 honestly lists them as new, which is correct. Keep that honesty; the rest of §5.1 ("no new infrastructure needed") is fair because those underlying components are real.
5. 🟡 **Hanzo Brain is a local SQLite memory store, not a consensus reasoner** — don't let the reference imply otherwise.

**Bottom line:** the paper's foundation is unusually well-grounded — PoAI, DSO, decentralized training, HLLM/TF-GRPO, the 7680-dim Experience Ledger, Quasar, the Zen edge models, FHE/TEE, and even a real autonomous-AI consensus package all exist as Final ZIPs/LPs, shipped repos, or externally-anchored results. The required fixes are surgical: **(a) renumber to 0435–0438; (b) cite `zoo-apps/ZIPs` not `zoo-labs/zips`; (c) correct the Zen-dataset token/content claim; (d) tighten the Quasar "finality driver" and "EasyCrypt-in-consensus" wordings; (e) cite `luxfi/consensus/ai` to strengthen the Cognitive-Consensus/SAIP mechanism claim; (f) calibrate the autonomy rhetoric to "mechanism exists, pipeline is new."**
