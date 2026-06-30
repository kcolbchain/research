# Provenance-Preserving Digital Currency: Lineage-Tracked CBDC for Auditable Monetary Flow

> **Authors:** Pattermesh (P. Sundaram) — KCOLBCHAIN Research, with Abhishek Krishna ([@abhicris](https://github.com/abhicris)).
> **Status:** DRAFT — internal review · **Date:** June 2026 · **Paper 02** of the June 2026 batch.

---

## ⚠️ Disclaimer & context (read first)

- **Research, design-stage, not a product.** This is a *research paper* — a protocol design and a working prototype, not a shipped, audited, licensed, or operational system. The model is grounded in a **mock e₹ ledger** with **no real money, no custody, and no deposits**.
- **The e₹ is the RBI's CBDC.** The e-rupee (e₹) is the Reserve Bank of India's Central Bank Digital Currency; **the RBI issues and controls it** — this project does not, and is independent and unaffiliated. The in-prototype issuer is a clearly-labelled **Simulated RBI** behind a default-deny authorisation gate. Everything here is **subject to RBI/regulatory approval and the licences we will seek**.
- **Forward-looking material is a model, not a forecast.** Any figures are clearly-labelled **models with stated assumptions** — not forecasts, promises, or commitments. Read the assumptions before the numbers.
- **Privacy framing.** The auditability argument in §1 is offered as a **policy rationale paired with privacy-by-design** and India's Digital Personal Data Protection (DPDP) Act, 2023 — **not** a case for blanket surveillance. §4 is the binding constraint on §1.
- **Claims are grounded.** On-prototype claims are checkable against the named repository ([`kcolbchain/erupee-wallet`](https://github.com/kcolbchain), `wallet/ledger.py`, PR #3). External claims are traceable to a cited source. If a claim is not, that is a bug — please open an issue.
- **Attribution & licence.** Ideas and initiatives here are by Pattermesh (P. Sundaram), partner — KCOLBCHAIN, with Abhishek Krishna (@abhicris). Published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Cite as: *"kcolbchain Research, Provenance-Preserving Digital Currency, June 2026."*

---

## Abstract

A retail Central Bank Digital Currency (CBDC) creates a design tension that paper cash never had to resolve: the same ledger that makes the currency *auditable* can also make it *surveillable*. We argue that this tension is not binary, and we propose a concrete middle path — **provenance-preserving digital currency**, in which units of issued value carry a verifiable **lineage** (a parent → child derivation history, like the serial number on a banknote but with the complete family tree attached) while the *visibility* of that lineage is **scoped by law** rather than open by default.

The model is grounded in a working prototype: the **note-lineage ledger** of the `kcolbchain/erupee-wallet` design-stage sandbox (`wallet/ledger.py`, introduced in PR #3). There, e₹ is modelled not as a scalar balance but as denominated, immutable **notes**; issuance mints a note with no parents; a transfer spends the payer's notes and creates child notes (one to the payee, an optional change note back to the payer) that record the consumed note ids as `parents`. From any note one can **trace back** to a mint ("how did this value arrive?") and **trace forward** to its descendants ("how was it spent onward?"). We formalise three properties the prototype enforces — **value conservation** (mint is the only creator of value; transfers neither create nor destroy it), **traceability** (every live unit has a complete path back to an authorised issuance), and **selective disclosure** (the lineage graph is computed from immutable state, but exposure is mediated by a default-deny authorisation gate and lawful, scoped access).

We then give a privacy design that pairs auditability with DPDP-style data-minimisation and purpose-limitation, position the work against the CBDC, UTXO-provenance, and privacy-technology literatures, and state our limitations frankly: this is a sandbox with synthetic data, a *Simulated* RBI, and **no claim of regulatory approval**. The contribution is a design vocabulary and a runnable reference for a question every CBDC programme must answer — *how much provenance, visible to whom, under what legal control?* — not a deployable system.

---

## 1. Motivation

### 1.1 The auditability–privacy tension

Physical cash is anonymous and untraceable by construction: a banknote's serial number is recorded at issuance, but the chain of hands it passes through is not. This is a feature for privacy and a liability for oversight — cash is the settlement medium of choice for the informal, untaxed, and illicit economy precisely because its flow is opaque. A digital bearer instrument inverts the default. A CBDC ledger *can* record every transfer, which makes it powerful for auditability — and, in the wrong design, an instrument of total financial surveillance.

The mainstream framing treats these as opposite ends of a single slider: more auditability necessarily means less privacy. We reject the framing as a false dichotomy. The relevant design variables are *separable*:

1. **Whether** a unit of value carries a provenance record at all (the data model).
2. **Who** can read that record, **for what purpose**, and **under what authority** (the access model).

A system can score high on (1) — every unit fully traceable in principle — while keeping (2) tightly scoped, so that lineage is *visible to a lawful auditor with cause* and *not visible to the public, the issuer-as-marketer, or any party without a legal basis*. Provenance-preserving currency is the design that holds (1) and (2) apart on purpose. The lineage exists; its visibility is governed.

### 1.2 The policy rationale (and its limits)

Two legitimate public-policy aims motivate provenance, and we state them as *rationale*, not as licence:

- **Auditability of public money.** Central-bank money is a public good. The ability to demonstrate — to an auditor, a court, or a regulator — that a given unit of CBDC traces back to an authorised issuance, and to follow a specific flow forward when there is lawful cause (fraud, sanctions evasion, a court order), is a legitimate supervisory capability. It is the digital analogue of a bank's existing AML/CFT obligations, not a new surveillance power invented for the CBDC.
- **Reducing reliance on the cash-based informal economy.** A large informal, cash-settled economy erodes the tax base and concentrates illicit settlement in an opaque medium. A traceable-*in-principle* digital alternative can, over time, shift some of this activity onto rails where lawful oversight is *possible* — without making every transaction *publicly* visible.

These are paired, throughout this paper, with the hard constraint that **a rationale for capability is not a rationale for blanket access**. The capability to trace must be matched by:

- **Legal gating** — access only on a lawful basis (statute, warrant, regulatory mandate), never at the discretion of an operator.
- **Data minimisation and purpose limitation**, in the spirit of India's **Digital Personal Data Protection (DPDP) Act, 2023** — the lineage record exists, but the personal data attached to it is collected and exposed only as far as a stated, lawful purpose requires.
- **Proportionality** — the default state is privacy; disclosure is the exception, scoped to a specific note, flow, or party, and recorded.

We are explicit that mass, suspicionless monitoring of lawful citizens' spending is **not** a goal of this design and is **not** what provenance enables when (2) is implemented correctly. The remainder of the paper treats §4 (privacy design) as the binding envelope on §1's rationale, not an afterthought.

### 1.3 Scope of this paper

We describe a *model* and a *prototype*. We do **not** describe a deployed system, claim any throughput or adoption, or assert any regulatory status. The e₹ is the RBI's instrument; nothing here is live or licensed; named regulatory dependencies are dependencies *we would have to satisfy*, not relationships we claim to hold (see §6).

---

## 2. Model: the note-lineage ledger

This section is grounded directly in the working prototype: `wallet/ledger.py` in `kcolbchain/erupee-wallet`, introduced in **PR #3**, with its contract pinned in the repository's `SPEC.md` (§1 data model, §3 trace API). It is a **design-stage sandbox** — a mock ledger, synthetic data, no network I/O, a Simulated RBI issuer. We describe what it actually does.

### 2.1 Value as denominated notes, not a balance

The central modelling choice is to represent e₹ **not as a scalar account balance** but as a set of denominated, immutable **notes**. A note is the atom of value and carries its own provenance. In the prototype a note is the record:

| field        | type            | meaning                                                                 |
|--------------|-----------------|-------------------------------------------------------------------------|
| `id`         | str             | opaque note id, e.g. `note-000007`                                      |
| `amount`     | int (paise)     | denomination, `> 0`; integer paise (1 e₹ = 100 paise), no floats        |
| `owner`      | str             | wallet id currently holding the note                                    |
| `parents`    | list[str]       | note ids this note was derived from (`[]` for a freshly minted note)    |
| `origin`     | str             | `"mint"` for issued notes, else `"transfer"`                            |
| `minted_at`  | int             | logical-clock value at which the note was created                       |
| `spent`      | bool            | `True` once consumed by an onward transfer; a live note is a **leaf**   |
| `spent_at`   | int \| null     | logical-clock value at which it was spent                               |
| `event_id`   | str             | id of the transaction (mint or transfer) that created the note          |

This is a **UTXO-style** representation specialised for a CBDC. A wallet's balance is *derived*, not stored: it is the sum of the wallet's live (unspent) notes. Notes are never mutated in place — spending a note flips its `spent` flag and *creates new child notes*, so the historical record is append-only and immutable, which is what makes the lineage trustworthy after the fact.

### 2.2 Issuance: the only source of value

A single privileged operation creates value: **mint**, performed by the issuer (in the prototype, the clearly-labelled *Simulated* RBI). A mint creates a fresh note with `parents = []` and `origin = "mint"`, owned by the destination wallet. By construction this is the *only* event whose output value is not drawn from pre-existing notes. Mint is gated (see §2.5); while the gate is closed, a mint is denied and recorded, and no value enters the system.

### 2.3 Transfer: spend, split, and the forward lineage edge

A transfer of `amount` paise from payer to payee proceeds, on an allowed and solvent transfer, as follows (this is the exact prototype behaviour):

1. **Select inputs (FIFO).** The payer's live notes are taken oldest-first (ordered by `minted_at`, then `id`) until they cover `amount`. A spent note can never be selected again — this is precisely what prevents a double-spend.
2. **Spend the inputs.** Each selected input is marked `spent` at the current logical-clock instant. It is now a non-leaf interior node of the lineage graph.
3. **Create child notes.** The engine creates a **payee note** of `amount` owned by the payee. If the inputs over-cover the amount, it also creates a **change note** of the remainder back to the payer. *Both* children record **all** consumed input ids as their `parents` (a merge), so lineage is complete on both legs — the change a payer keeps is as traceable as the value the payee receives.

The `parents` field on a child is the **forward lineage edge**: it says "this note was derived from those notes." The prototype additionally maintains a `children` index (parent id → child ids) so the graph can be walked efficiently in both directions.

### 2.4 Tracing: back to a mint, forward to the leaves

From any note id — or any wallet id, in which case the wallet's live notes are the anchors — the ledger computes two graphs (the prototype's `trace(ref)` returns `{root, back, forward}`):

- **Back graph (`_ancestors`).** Walk `parents` edges transitively. This terminates at mint notes (which have no parents). It answers: *"How did this value arrive? Which authorised issuance(s) is it ultimately derived from, and through whose hands?"*
- **Forward graph (`_descendants`).** Walk `children` edges transitively, terminating at live leaves. It answers: *"How was this value spent onward?"*

Both graphs are returned as JSON `{nodes, edges}`, where an edge `from → to` always means *`to` was derived from `from`* (parent → child) — the same orientation in both directions, so the two graphs compose into one consistent provenance DAG. Edges carry the logical-clock instant and the originating event id, so the trace is a fully ordered, reproducible audit object, not a heuristic reconstruction.

A worked micro-example (logical time in brackets):

```
[t=1] mint        →  note-1 (₹100, owner=A, parents=[], origin=mint)
[t=2] A pays B ₹30:
         spend note-1
         note-2 (₹30, owner=B, parents=[note-1], origin=transfer)   ← payee
         note-3 (₹70, owner=A, parents=[note-1], origin=transfer)   ← change
[t=3] B pays C ₹30:
         spend note-2
         note-4 (₹30, owner=C, parents=[note-2], origin=transfer)

trace(note-4).back     = note-4 → note-2 → note-1 (mint)      "arrived via A→B→C"
trace(note-1).forward  = note-1 → {note-2, note-3}; note-2 → note-4
```

### 2.5 The compliance and issuer-authorisation gate

Every mint and transfer is routed through a compliance pre-check **before** any value moves, with a **default-deny** issuer-authorisation gate as the first and hardest condition. The prototype's gate is deliberately strict, mirroring the real-world rule that e₹ may be handled only through an RBI-authorised CBDC channel: while the `rbi_authorised` flag is false, *both* mint and transfer deny. The demo flips this on only in an explicit, loudly-labelled `SANDBOX_AUTHORISED` mode.

A denied operation is **still recorded** as a transaction (with `decision = "deny"`, empty inputs/outputs, and stable reason codes) — a refusal is an auditable outcome, not a gap in the trail. The check also models per-party KYC tiers (`none` cannot transact; `min` has a per-transfer cap; `full` has none) and a synthetic sanctions screen, returning stable reason codes (`RBI_NOT_AUTHORISED`, `KYC_REQUIRED`, `KYC_TIER_LIMIT`, `SANCTIONED_PARTY`, `AMOUNT_NOT_POSITIVE`, `INSUFFICIENT_FUNDS`). Crucially, *any* non-allow or malformed return is treated as deny (fail-safe). This gate is the hook on which §4's lawful, scoped-access design hangs: provenance data exists, but the system's posture is closed by default.

### 2.6 Determinism

All time in the model is a **logical monotonic clock** — an integer incremented on every event — never the wall clock. This makes the entire ledger, and every trace it produces, deterministic and reproducible: the same sequence of operations yields byte-identical lineage graphs. For an audit instrument this is not a convenience but a requirement — a provenance proof must be reproducible to be trustworthy.

---

## 3. Properties

We state three properties the model is designed to hold and show how the prototype enforces each. These are *design properties of the prototype*, not theorems about a deployed system.

### 3.1 Value conservation

> **Property.** Mint is the only operation that creates value. A transfer conserves value exactly: the sum of consumed inputs equals the payee amount plus any change.

*Enforcement.* A transfer's outputs are constructed as `payee_note(amount) + change_note(Σinputs − amount)`, with `change` created only when positive, so `Σ inputs = payee + change` by construction. No transfer path creates a note whose value is not drawn from spent inputs. The prototype exposes two independent supply measures — `total_issued` (sum of allowed mints) and `in_circulation` (sum of all live notes) — and they are computed by **different** code paths so they cross-check each other; in the sandbox they are necessarily equal because mint is the sole value source, transfers conserve, and nothing is burned. Integer paise throughout means there is no floating-point drift to erode conservation.

### 3.2 Traceability

> **Property.** Every live unit of value has a complete, finite lineage path back to one or more authorised mints; and every flow can be followed forward to its current leaves.

*Enforcement.* Because (a) mint is the only parentless note, (b) every transfer-created note records its consumed inputs as `parents`, and (c) notes are immutable and append-only, the parent relation forms a finite DAG whose only roots are mints. The back-trace therefore *always* terminates at issuance; the forward-trace always terminates at live leaves. There is no way to introduce a live note with no ancestry except an authorised mint, which is exactly the auditability guarantee: unexplained value cannot exist on the ledger.

### 3.3 Selective disclosure

> **Property.** The *existence* of a full lineage record is independent of its *visibility*. The graph is computed from immutable state on demand; exposure is mediated by a default-deny authorisation gate and (in a real deployment) a lawful-access policy.

*Enforcement and framing.* In the prototype, the lineage is surfaced through a strictly **read-only projection** (`explorer/api.py`) that never mutates ledger state and returns deep copies, and through a **per-wallet history projection** (`wallet/history.py`) that renders only the transactions touching a given wallet, from that wallet's point of view. The architectural separation matters: *recording* provenance (the ledger) is distinct from *disclosing* it (the projections), and the projections can be placed behind access control without changing the ledger. This is the technical seam at which a deployment inserts lawful, scoped disclosure (§4): the same default-deny posture that gates value movement (§2.5) gates lineage exposure. In the sandbox the explorer is open because the data is synthetic; in any real system it would not be.

---

## 4. Privacy design (lawful, scoped)

This section is the binding constraint on §1. A provenance-preserving CBDC is only defensible if disclosure is **lawful, scoped, minimal, and proportionate by default**. We describe a design posture, aligned with India's **DPDP Act, 2023**, and we are explicit about what we are *not* proposing.

### 4.1 Default-deny, lawful-basis access

The system's default posture is **closed**. The prototype's default-deny authorisation gate (§2.5) is the model for this at the value layer; the same principle applies to the *disclosure* layer. Reading the lineage of a note, or following a flow, should require a **lawful basis** — a statutory mandate, a regulatory function, or a judicial order — and never the unconstrained discretion of an operator or the issuer. No party gets standing access to "everyone's spending"; access is to a *specific* note, flow, or party, *for a stated purpose*.

### 4.2 Separation of identity from lineage (data minimisation)

The note-lineage graph in the prototype is over **wallet ids and note ids** — opaque strings — not over personal identities. Identity and KYC are a *separate* concern (in the prototype, separate `identities`/`kyc` modules), bound to an identity record, with KYC for PAN-linking modelled as **format validation only** on synthetic data — it verifies nothing real and calls no external network. This separation is deliberate and is the core of the privacy design:

- The **provenance layer** answers "this value derives from that issuance, via these wallets." It needs no names to do its job.
- The **identity layer** answers "who controls this wallet." It is governed independently and disclosed independently.
- **Re-association** of a lineage with a legal identity is itself a privileged, lawful-basis operation — exactly the point at which DPDP purpose-limitation and minimisation bite. One can audit a *flow* without unmasking the *people* unless and until there is a lawful reason to.

In DPDP terms: the personal data (the identity behind a wallet) is **collected and processed for a specified, lawful purpose**, kept **minimal**, and **not** automatically fused with the transaction graph. The lineage is pseudonymous at rest; de-pseudonymisation is the controlled exception.

### 4.3 Purpose limitation, proportionality, and accountability

- **Purpose limitation.** A disclosure is scoped to its stated purpose (e.g. investigating a specific suspected fraud), not a general grant.
- **Proportionality.** The scope of a trace should be the minimum needed — a single flow, not a whole population; a window, not all history.
- **Accountability / auditing the auditors.** Because every ledger event (including denials) is recorded with a logical-clock instant and reason codes, *disclosure requests themselves* can be made into recorded, reviewable events in a deployment — the oversight mechanism is itself auditable. This is the structural answer to "who watches the watchers."

### 4.4 What this design explicitly is not

- It is **not** public transparency of personal spending. Provenance being *recorded* is not provenance being *published*. The visibility is governed (§3.3, §4.1).
- It is **not** suspicionless mass monitoring. The default is privacy; lawful, scoped disclosure is the exception (§4.1, §4.3).
- It is **not** a unilateral capability for the operator or issuer. Access requires an external lawful basis, not internal discretion (§4.1).
- It is **not** a complete privacy technology. We do not, in the current prototype, implement cryptographic confidentiality (e.g. zero-knowledge proofs of a valid spend that hide the lineage from the verifier). We discuss this as future work (§7) and as a limitation (§6).

---

## 5. Related work

- **CBDC architecture and privacy.** Central-bank and standards work — including the **Bank for International Settlements** and partner central banks (e.g. Project Hamilton, Project Tourbillon) and the **RBI's e₹ pilots** (retail CBDC-R as a digital bearer instrument via authorised banks; wholesale CBDC-W; programmability and UPI-interoperability features) — frames the retail-CBDC privacy debate and the "authorised channel" model. Our gate (§2.5) is a sandbox analogue of the authorised-channel rule, and our §4 posture is one point in the BIS-articulated design space between account-based transparency and cash-like anonymity. The **DPDP Act, 2023** is the Indian data-protection frame our §4 aligns to.
- **UTXO and value-provenance models.** The note model is a UTXO specialisation: value as immutable, denominated outputs consumed and re-created by transactions, with explicit parent links — the lineage being the transaction DAG. We borrow the UTXO graph's natural traceability and discard its public-ledger transparency, replacing the latter with governed disclosure.
- **Privacy-preserving payments.** A literature on confidential and anonymous digital cash (from Chaumian blind-signature e-cash through modern shielded-transaction systems and zero-knowledge constructions) sits at the *opposite* pole from full provenance. Our position is explicitly *between* these poles: full provenance *recorded*, governed disclosure *exposed* — with cryptographic confidentiality of the lineage itself flagged as future work (§7).
- **AML/CFT and the travel rule.** Existing obligations on regulated intermediaries (KYC, sanctions screening, originator/beneficiary data above thresholds) are the policy context for §1's rationale and the prototype's reason-coded compliance gate. Provenance-preserving currency is intended to make these *existing* obligations cleaner to satisfy, not to create a new surveillance regime.
- **kcolbchain internal work.** This paper sits alongside the collective's stablecoin and depeg analyses and the `erupee-wallet` sandbox's exchange/settlement Track-2 designs (CBDC ↔ stablecoin ↔ fiat, multilateral net settlement), which reuse the *same* note-lineage ledger as the single source of e₹ truth.

---

## 6. Limitations and regulatory path

### 6.1 Limitations

1. **Design-stage sandbox, synthetic data.** The model is grounded in a *mock* ledger with a *Simulated* RBI and fabricated identities/KYC/PAN. There is **no real money, custody, or deposit**, and **no network I/O**. Nothing here is live, licensed, or operational.
2. **No cryptographic confidentiality (yet).** Selective disclosure in the current prototype is *architectural* (separation of layers, default-deny, read-only projections), not *cryptographic*. A production design would likely need zero-knowledge or commitment-based confidentiality so that a flow can be validated without revealing the lineage to the validator (§7). We do not claim to have built this.
3. **Privacy posture is a design, not a proof.** §4 describes the intended governance envelope. Whether a *deployment* honours it depends on legal, operational, and key-management controls outside the prototype. The model makes the right posture *possible*; it does not *guarantee* it.
4. **Scaling and metadata.** A note/UTXO model produces graph growth and change-note proliferation; lineage queries over a long history have cost, and even pseudonymous graphs can leak via metadata/traffic analysis. These are real, unaddressed-here engineering and privacy concerns.
5. **No empirical or adoption claims.** This paper reports *no* throughput, user, revenue, or adoption numbers. Any such figure elsewhere would be a clearly-labelled model with assumptions, not a forecast.

### 6.2 Regulatory path

The e₹ is the RBI's instrument; **the RBI issues and controls it.** Any move from this sandbox toward anything real is **gated on approvals and licences we would have to seek**, and on counsel review. Per kcolbchain's **two-track model**, the open-source framework takes no custody and holds no deposits; any commercial, deposit-handling operation would be a **separate, separately-licensed Track-2 entity** carrying its own regulatory obligations and risk. Concretely, the dependencies (as documented in the sandbox's compliance notes) include:

- **RBI** — authorisation to handle the e₹ only through an approved CBDC channel; **PSS Act, 2007** authorisation for any payment system; and confirmation of treatment of any stablecoin/settlement legs.
- **NPCI** — for any UPI interoperability, the PSP/sponsor-bank arrangement and applicable rules.
- **DPDP Act, 2023** — the data-protection envelope for §4 (lawful basis, purpose limitation, minimisation), to be implemented and verified with counsel.
- **A sponsor/authorised bank** — named candidates (e.g. SBI) are **stated, not finalised**; no such partnership is concluded, and any reference is aspirational pending agreement and approval.

Nothing in this paper is legal, tax, or financial advice. All of the above must be **resolved before any deployment**.

---

## 7. Future work

1. **Cryptographic selective disclosure.** Replace architectural confidentiality with zero-knowledge / commitment-based proofs so a transfer's validity (conservation, authorised ancestry, no double-spend) can be verified *without* the verifier learning the lineage — closing the gap in §4.4 and §6.1(2).
2. **Threshold / lawful-access cryptography.** Make de-pseudonymisation (§4.2) require a *quorum* of independent authorities (e.g. threshold decryption) rather than a single operator's action, with the request recorded as an auditable event (§4.3) — encoding "auditing the auditors" cryptographically.
3. **Tiered privacy by value/role.** Explore cash-like *full* anonymity below a small threshold (low-value, retail) with progressively stronger provenance/identity binding above it, formalising proportionality (§4.3) into the protocol.
4. **Metadata-resistance.** Study and mitigate the traffic-analysis and change-note linkage leaks noted in §6.1(4).
5. **Formalisation.** State and prove §3's properties (conservation, traceability, governed disclosure) as theorems over a formal model of the ledger, beyond the prototype's enforced behaviour.
6. **Interoperability.** Extend the note-lineage model across the Track-2 exchange and settlement layers (CBDC ↔ stablecoin ↔ fiat, multilateral net settlement) so provenance survives instrument boundaries — all gated, as ever, on the approvals and licences we would seek.

---

## References

1. Reserve Bank of India (2022). "Concept Note on Central Bank Digital Currency." RBI Department of Currency Management.
2. Reserve Bank of India (2022–). "Digital Rupee (e₹) Pilot — Retail (CBDC-R) and Wholesale (CBDC-W)." RBI Press Releases.
3. Government of India (2023). "The Digital Personal Data Protection Act, 2023." Ministry of Electronics and Information Technology.
4. Government of India (2007). "The Payment and Settlement Systems Act, 2007." Reserve Bank of India.
5. Bank for International Settlements (2021). "Central Bank Digital Currencies: System Design and Interoperability." BIS / CPMI Report (with partner central banks).
6. Federal Reserve Bank of Boston & MIT DCI (2022). "Project Hamilton Phase 1: A High Performance Payment Processing System Designed for CBDC." Federal Reserve Bank of Boston.
7. Bank for International Settlements Innovation Hub (2023). "Project Tourbillon: Exploring Privacy, Security and Scalability for CBDCs." BIS Innovation Hub.
8. Chaum, D. (1983). "Blind Signatures for Untraceable Payments." *Advances in Cryptology — CRYPTO '82*, Springer.
9. Nakamoto, S. (2008). "Bitcoin: A Peer-to-Peer Electronic Cash System." (UTXO transaction-graph model.)
10. Sasson, E. Ben, et al. (2014). "Zerocash: Decentralized Anonymous Payments from Bitcoin." *IEEE Symposium on Security and Privacy*.
11. Financial Action Task Force (2021). "Updated Guidance for a Risk-Based Approach to Virtual Assets and VASPs" (the "travel rule"). FATF.
12. kcolbchain (2026). "erupee-wallet — design-stage CBDC sandbox: note-lineage ledger (`wallet/ledger.py`, PR #3) and `SPEC.md`." [github.com/kcolbchain](https://github.com/kcolbchain).

---

*Research paper prepared for the kcolbchain research repository. The e-rupee (e₹) is the Reserve Bank of India's CBDC; this project is independent and unaffiliated, design-stage, and subject to RBI approval and the licences we will seek. Not legal, tax, or financial advice. Last updated: June 2026.*

*Conceived and written by Pattermesh (P. Sundaram), partner — KCOLBCHAIN, with Abhishek Krishna (@abhicris).*
