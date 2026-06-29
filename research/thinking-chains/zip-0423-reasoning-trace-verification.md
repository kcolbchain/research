---
zip: 423
title: Reasoning Trace Verification via Semantic Embeddings
description: A formal verification primitive for LLM reasoning traces, defining the canonical trace encoding, the DSO 7680-dimensional embedding commitment, the cosine-distance equivalence test with calibrated thresholds, and the dispute escalation to deterministic re-execution that ZIP-0420 and ZIP-0421 build on.
author: Pattermesh (KCOLBCHAIN Research)
status: Draft
type: Standards Track
category: AI
created: 2026-06-30
requires: 410, 419
---

## Abstract

Reasoning Trace Verification (RTV) is the shared verification primitive underneath Cognitive Consensus (ZIP-0420) and SAIPs (ZIP-0421). It answers one question rigorously: *given two reasoning traces produced over the same committed context, are they equivalent enough to be treated as the same act of reasoning, and if not, who is right?* RTV defines (1) a canonical, content-addressable trace encoding; (2) a deterministic embedding of a trace into the DSO 7680-dimensional space (ZIP-0410); (3) a **semantic equivalence test** based on cosine distance with a calibrated, governance-tunable threshold and an explicit ROC-driven calibration procedure; (4) a **structural soundness check** that verifies a trace's evidence against committed state without any model; and (5) a **dispute escalation** to bit-exact deterministic re-execution under a pinned numeric profile, yielding a slashable verdict. By factoring this out as its own Standards-Track primitive, both the consensus layer and the governance layer share one audited, independently-testable verification module rather than re-implementing semantic comparison ad hoc.

## Motivation

ZIP-0420 (Tier-2 semantic sampling) and ZIP-0421 (adversarial review and outcome curation) both depend on a precise notion of "are these two reasoning traces the same?" Left informal, that notion invites two opposite failures: a test that is too strict false-challenges benign paraphrase and stalls cognitive rewards; a test that is too loose lets a substantively divergent (or adversarially crafted) trace pass as equivalent. Either failure undermines the whole Thinking Chains stack.

RTV exists to make trace equivalence a **specified, calibrated, and auditable** operation:

1. **One canonical encoding** so that `H(canonical(trace))` is a stable content address across implementations and the same trace never hashes two ways.
2. **A reproducible embedding** so that two honest validators on different hardware embed equivalent traces to within a bounded distance.
3. **A calibrated threshold `τ`** chosen from real ROC data, not guessed, with a defined re-calibration cadence.
4. **A structural check** that catches the cheapest, most dangerous attack (fabricated evidence) deterministically and model-free.
5. **A deterministic tie-breaker** (re-execution) so disputes have a single correct answer that can carry slashing.

This is the "Reasoning Trace Verification via Semantic Embeddings" component the source draft slots at ZIP-0423.

## Specification

Embeddings use the DSO 7680-dimensional space and embedding service (ZIP-0410). Hashes are BLAKE3-256. The trace data model is the `ReasoningTrace` of ZIP-0420 §4; RTV defines its canonical *encoding*, *embedding*, and *comparison*.

### 1. Canonical trace encoding

A trace is canonicalized before hashing or embedding so that semantically identical traces from different runtimes serialize identically:

```
canonical(trace) = RLP([
    u16(version),
    sorted_by_id([ canonical_node(n) for n in trace.nodes ])
])
canonical_node(n) = RLP([
    u32(n.id), u8(n.kind),
    sorted_ascending(n.parents),
    nfc_normalize(strip(n.statement)),     # Unicode NFC, trimmed, collapsed whitespace
    encode_evidence(n.evidence_ref),       # canonical Merkle path + value, or empty
    u16(n.confidence)
])
```

Rules: nodes ordered by `id`; parent lists sorted ascending; `statement` Unicode-NFC-normalized with collapsed internal whitespace and stripped ends; floating quantities expressed as fixed-point integers only. The content address is `trace_cid = H(canonical(trace))`. **There is exactly one canonical form per trace**, so `output_commit` (ZIP-0420) is implementation-independent.

### 2. Structural soundness (model-free, O(nodes))

`structurally_sound(trace, state_roots)` returns true iff:

1. The node set is a DAG with `parents[i] < i` for all nodes (acyclicity by construction).
2. ≥1 `PREMISE` (no parents); ≥1 `CONCLUSION` transitively reachable from premises.
3. Every `EVIDENCE` node carries an `evidence_ref` whose `StateMerklePath` verifies against `state_roots`, and whose `value` matches the leaf. **A fabricated fact fails here, with no model invoked.**
4. Confidences ∈ [0, 10000]; the conclusion's confidence equals the trace's reported `confidence`.

Structural soundness is a *precondition* for any further verification. An unsound trace is rejected regardless of its embedding.

### 3. Deterministic trace embedding

```
embed(trace) ∈ ℝ^7680
embed(trace) = L2_normalize( DSO_encoder( canonical(trace) ) )
```

`DSO_encoder` is the governance-pinned encoder from ZIP-0410, addressed by `encoder_hash` (carried alongside `model_hash`). Embedding operates on the **canonical** encoding (§1), so the embedding input is implementation-independent. The output is L2-normalized so cosine similarity reduces to a dot product. Cross-hardware embedding drift is bounded: for honest reproductions of equivalent traces, `1 − cos(e, e') ≤ δ_embed` (default `δ_embed = 0.02`) under the pinned `encoder_hash`; this bound is what makes `τ` meaningful across heterogeneous validators.

### 4. Semantic equivalence test

```
equivalent(e_a, e_b ; τ)  :=  cos(e_a, e_b) ≥ τ          # e both L2-normalized
```

`τ` is a governance parameter (the same `τ_sem` ZIP-0420 Tier-2 uses; default 0.92). `equivalent` is the operation ZIP-0420 committees and ZIP-0421 reviewers call. It is **symmetric** and, because embeddings are L2-normalized, numerically `cos = ⟨e_a, e_b⟩`.

Semantics of the regions:

| cosine | interpretation | action |
|--------|----------------|--------|
| `≥ τ` | semantically equivalent | ACCEPT |
| `[τ − γ, τ)` | borderline (default `γ = 0.03`) | committee may request one extra reproduction before deciding |
| `< τ − γ` | substantive divergence | CHALLENGE → escalate (§6) |

### 5. Threshold calibration (ROC procedure)

`τ` MUST be derived from data, not chosen arbitrarily. The procedure, run at the cadence below:

1. **Build a labeled corpus** from chain history: positive pairs = independent honest reproductions of the same `(context, seed, model_hash)` (label *equivalent*); negative pairs = traces from materially different contexts or with adversarially altered conclusions (label *divergent*). Negatives include challenge-confirmed divergences from ZIP-0420 Tier-3 adjudications — real, ground-truthed bad traces.
2. **Compute the ROC** of the cosine test over the corpus.
3. **Choose `τ`** at the operating point that bounds the false-challenge rate (honest pair scored as divergent) below a governed ceiling (default ≤ 1%), then minimizes the missed-divergence rate subject to that ceiling.
4. **Publish** `{τ, encoder_hash, corpus_root, roc_summary}` on-chain; the new `τ` activates at a future epoch via governance.

**Cadence:** re-calibration is mandatory on any `encoder_hash` change and recommended every `T_cal` epochs (default 8192) or whenever the rolling false-challenge rate drifts outside [0.5%, 2%]. This makes `τ` an evidence-backed, drift-tracked parameter rather than a magic constant, and gives ZIP-0421's simulation gate a concrete invariant to check when a SAIP proposes changing `τ`.

### 6. Dispute escalation (deterministic, authoritative)

When the equivalence test yields CHALLENGE (or a ZIP-0421 reviewer disputes a trace's grounding), RTV escalates to the **authoritative** check — bit-exact deterministic re-execution, which embeddings only approximate:

1. Reconstruct the committed context (ZIP-0420 §3) — fully determined by `context_root`.
2. Run the pinned model under the **pinned numeric profile** bound by `model_hash` (fixed quantization, kernel profile, temperature 0, fixed `seed`), producing the deterministic canonical trace `t*`.
3. Compare per-logit within `ε_num` (default 2⁻¹⁰); decode is greedy, so `canonical(t*)` is unique.
4. **Verdict:** the trace whose `trace_cid == H(canonical(t*))` is correct.
   - If the challenged proposer matches `t*` → the challenger is wrong (false-challenge slash, per ZIP-0420).
   - If the challenged proposer diverges from `t*` beyond `ε_num` → the proposer is wrong (invalid-attestation slash).

Re-execution is `O(n)` in adjudicators worst-case but invoked only on dispute; the embedding test (§4) handles the common case in `O(k)`. The embedding distance is thus a *cheap probabilistic filter*, and re-execution is the *exact judge* — RTV makes the relationship between the two explicit and slashing-backed.

### 7. The verification function (composed)

```
verify_trace(trace_a, trace_b, ctx, state_roots, τ) =
    require structurally_sound(trace_a, state_roots)        # §2, model-free precondition
    e_a, e_b = embed(trace_a), embed(trace_b)               # §3
    c = cos(e_a, e_b)                                        # §4
    if c >= τ:           return ACCEPT
    if c >= τ - γ:       return BORDERLINE                   # request one more reproduction
    return escalate(ctx) # §6 deterministic re-execution -> slashable verdict
```

ZIP-0420 Tier-2 calls `verify_trace` with `trace_b` being the committee member's local reproduction; ZIP-0421 uses `structurally_sound` to gate SAIP root-cause grounding and `equivalent` to detect when reviewer reproductions of a SAIP's analysis diverge from the author's.

## Rationale

**Why a separate ZIP rather than inlining into ZIP-0420?** Both the consensus layer and the governance layer need identical trace-equivalence semantics. Factoring RTV out gives one audited module, one place to calibrate `τ`, and one canonical encoding — avoiding subtly different comparisons in two places, which would be a correctness and security hazard.

**Why embeddings as the fast path and re-execution as the judge?** Text equality punishes benign paraphrase and cross-hardware nondeterminism; bit-exact re-execution is correct but too expensive for every block. Cosine distance in the DSO space is cheap, tolerant of paraphrase, and sensitive to substantive divergence — an ideal *filter*. Reserving exact re-execution for disputes pairs a cheap probabilistic test with an exact authoritative one, and §6 makes their relationship formal rather than folklore.

**Why canonicalize before embedding and hashing?** Without a single canonical form, two honest runtimes could produce traces that differ only in ordering/whitespace yet hash and embed differently, manufacturing spurious challenges. Canonicalization (§1) removes that entire class of false positives at the source.

**Why a documented ROC calibration with a cadence?** A guessed `τ` is both a liveness risk (too strict → false challenges stall rewards) and a security risk (too loose → divergence slips through). Grounding `τ` in labeled data — including real challenge-confirmed divergences as negatives — and re-calibrating on encoder change or drift makes the threshold defensible and gives ZIP-0421 a concrete invariant to simulate against.

**Why keep structural soundness model-free and first?** It is the cheapest check and it catches the highest-severity attack (fabricated evidence). Running it as a hard precondition means no compute is spent embedding or re-executing a trace that already lies about committed state.

## Backwards Compatibility

- RTV is a **library/primitive specification**; it introduces no new transaction type or consensus rule on its own. It is consumed by ZIP-0420 and ZIP-0421.
- The canonical encoding (§1) is the definition of `output_commit`/`trace_cid` used by ZIP-0420; any pre-RTV ad-hoc encoding is superseded, but since no Cognitive Consensus deployment predates ZIP-0420, there is no live data to migrate.
- `τ`, `γ`, `δ_embed`, `ε_num`, `T_cal` are governance parameters with the defaults above; changing them is a vote (and, per §5, a `τ` change requires a published re-calibration). Shipping defaults means existing ZIP-0420/0421 behavior is unchanged on adoption.
- A change of `encoder_hash` (ZIP-0410) forces re-calibration (§5) and re-embedding is required only for *newly verified* traces; historical `trace_cid` content addresses are unaffected because they hash the canonical encoding, not the embedding.

## Security Considerations

1. **Adversarial near-collisions (embedding evasion):** an attacker might craft a divergent trace whose embedding sits just above `τ`. Mitigations: the structural check (§2) independently verifies evidence (an evasive trace still cannot fabricate Merkle-verified facts); the borderline band `[τ−γ, τ)` triggers extra reproductions; and any honest party can escalate to deterministic re-execution (§6), where the evasion is exposed bit-exactly and slashed. The embedding test is a filter, never the final authority.
2. **Threshold mis-calibration:** bounded by the ROC procedure (§5) with a governed false-challenge ceiling and mandatory re-calibration on encoder change/drift; `τ` changes go through governance with a published corpus root, auditable after the fact.
3. **Canonicalization ambiguity:** §1 fixes ordering, normalization, and numeric representation so the canonical form is unique; an implementation that produces a non-canonical encoding simply fails to match honest `trace_cid`s and is detected.
4. **Encoder poisoning / drift:** `encoder_hash` is governance-pinned (ZIP-0410); a swapped encoder invalidates `τ` and forces re-calibration before any new `τ` activates, preventing a silent encoder change from skewing equivalence.
5. **Re-execution nondeterminism:** §6 pins quantization/kernel/decode via `model_hash` and bounds per-logit difference by `ε_num`; hardware classes that cannot meet the profile are ineligible to adjudicate (consistent with ZIP-0420 §5.3), so the authoritative check cannot itself become a source of disagreement.
6. **Privacy of disputed traces:** when traces carry sensitive content (MEV/strategy, ZIP-0420 Open Question 4), structural and embedding checks can be performed inside a TEE over the canonical encoding (the embedding and `trace_cid` are computed there), so a dispute does not force public disclosure of the full trace; only the verdict and commitments leave the enclave.
7. **DoS via forced escalation:** because escalation is slashing-backed (a false challenge is penalized, ZIP-0420), an attacker spamming challenges to trigger expensive re-execution pays for each lost challenge, making the attack self-limiting.

## Reference Implementation (sketch)

```python
def canonical(trace) -> bytes:
    nodes = sorted(trace.nodes, key=lambda n: n.id)
    return rlp([u16(trace.version), [canonical_node(n) for n in nodes]])

def canonical_node(n) -> bytes:
    return rlp([u32(n.id), u8(n.kind), sorted(n.parents),
                nfc_collapse(n.statement.strip()),
                encode_evidence(n.evidence_ref), u16(n.confidence)])

def structurally_sound(trace, state_roots) -> bool:
    if not is_dag_ascending(trace.nodes): return False
    if not (has_premise(trace) and conclusion_reachable(trace)): return False
    for n in trace.nodes:
        if n.kind == EVIDENCE:
            if not merkle_verify(n.evidence_ref.path, state_roots): return False
            if leaf_value(n.evidence_ref.path) != n.evidence_ref.value: return False
        if not (0 <= n.confidence <= 10000): return False
    return True

def embed(trace) -> Vec7680:                       # DSO encoder, pinned encoder_hash
    return l2_normalize(dso_encoder(canonical(trace)))

def verify_trace(a, b, ctx, state_roots, tau, gamma=0.03) -> Verdict:
    if not structurally_sound(a, state_roots): return INVALID
    c = dot(embed(a), embed(b))                    # cosine, both L2-normalized
    if c >= tau:          return ACCEPT
    if c >= tau - gamma:  return BORDERLINE        # request one more reproduction
    return escalate(ctx, a)                        # deterministic re-exec -> slashable

def escalate(ctx, challenged) -> Verdict:          # §6, authoritative
    t_star = pinned_profile_infer(reconstruct(ctx))   # eps_num bound, greedy, fixed seed
    return ACCEPT if blake3(canonical(challenged)) == blake3(canonical(t_star)) \
                  else SLASH_PROPOSER

def calibrate_tau(corpus, max_false_challenge=0.01) -> float:
    roc = roc_curve(cosine_scores(corpus), labels(corpus))
    return operating_point(roc, fpr_ceiling=max_false_challenge)  # min missed-divergence s.t. FPR<=ceiling
```

Components reused: ZIP-0410 DSO encoder (`encoder_hash`) and 7680-dim space; ZIP-0420 `ReasoningTrace` data model, context reconstruction, pinned-profile re-execution, and slashing; ZIP-0419 TEE for private-trace verification.

## Copyright

Copyright and related rights waived via CC0.
