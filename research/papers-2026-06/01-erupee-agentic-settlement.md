# The e-Rupee as an Agentic Settlement Layer: CBDC-native machine-to-machine payments

> **Authors:** Pattermesh (P. Sundaram), partner — KCOLBCHAIN Research, with Abhishek Krishna ([@abhicris](https://github.com/abhicris)).
> **Status:** DRAFT — internal review · **Date:** June 2026 · **Paper 01 of the June 2026 batch.**
> Cite as: *"kcolbchain Research, *The e-Rupee as an Agentic Settlement Layer*, June 2026."* Licensed [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

---

> ## ⚠️ Disclaimer & status (read first)
>
> - **This is a research paper and a design, not a product.** It analyses a direction and describes prototypes. Nothing here is a shipped, audited, licensed, or production system.
> - **The e-rupee (e₹) is the Reserve Bank of India's Central Bank Digital Currency.** The RBI issues and controls it. kcolbchain does **not** issue, mint, custody, or operate the CBDC. Everything in this paper is **design-stage and subject to the approvals and licences we will seek from the RBI, NPCI, and counsel** under the Payment and Settlement Systems Act, 2007 and the RBI CBDC framework. Named counterparties (e.g. SBI) are *stated, not finalised*.
> - **The e₹ work described here is a sandbox.** The `erupee-wallet` repository runs a **mock e₹ ledger with a clearly-labelled Simulated RBI issuer**, behind a default-deny authorisation gate, with **synthetic identities only** and **no network I/O, no custody, no deposits, and no real money** ([erupee-wallet SPEC.md](https://github.com/kcolbchain/erupee-wallet)). Per kcolbchain's two-track model, any commercial e₹ wallet/exchange is a separately-licensed **Track-2 spin-out** carrying its own regulatory risk — not kcolbchain.
> - **Any numbers are illustrative model outputs, not forecasts.** Where this paper shows figures, they come from a **consistency-checking simulator with stated assumptions** ([kcolbchain/token-simulator](https://github.com/kcolbchain/token-simulator)) and are labelled as such. They are not benchmarks, throughput claims, revenue forecasts, or promises.
> - **Grounding.** Every repository or pull-request claim should be checkable against the named kcolbchain repo. If you find a claim that is not, that is a bug — please open an issue.

---

## Abstract

Autonomous software agents can already discover services, call tools, and delegate work, but they still struggle to *pay* for that work cleanly at runtime — and when they do pay, the resulting flows are largely anonymous, hard to reconcile, and outside any regulator's view. Two trends are converging on this gap. First, the dormant HTTP `402 Payment Required` status code has been revived as a coordination point for machine-to-machine (M2M) payment, through protocols such as x402, the Machine Payments Protocol (MPP), and AP2, alongside high-frequency stablecoin clearing (Circle Nanopayments). Second, central banks — the Reserve Bank of India foremost among them — have moved real Central Bank Digital Currency (CBDC) into live pilots, producing a programmable, central-bank-issued settlement instrument for the first time.

This paper asks what an *agentic settlement layer* looks like if the final settlement asset is a CBDC rather than a private stablecoin or unbacked token. We argue that the e₹ is unusually well-suited to the role: it is final central-bank money (no issuer-default or de-peg risk), it is natively identity-bound (matching the compliance posture agentic commerce will ultimately be held to), and as a programmable instrument it can be conditioned on machine-checkable policy.

We ground the architecture in concrete, open-source work the kcolbchain collective has already built: the **e₹ provenance ledger and default-deny compliance gate** in `erupee-wallet` ([PR #1](https://github.com/kcolbchain/erupee-wallet/pull/1), [PR #3](https://github.com/kcolbchain/erupee-wallet/pull/3)); the **x402 middleware and `AgentEscrow` contract** in `switchboard` ([PR #131](https://github.com/kcolbchain/switchboard/pull/131), [PR #132](https://github.com/kcolbchain/switchboard/pull/132)), whose oracle-gated `releaseByAttestation` path is the technical hinge that lets escrow release be conditioned on a compliance attestation; the **sequential, fail-closed safety gates** of `SafeSwap`; and the **e₹ settlement layer** stub ([PR #4](https://github.com/kcolbchain/erupee-wallet/pull/4)) that nets obligations against the CBDC as the final settlement asset. The central design claim is that an HTTP-native M2M payment handshake, a trustless escrow whose release is policy-conditioned, and a CBDC-native compliance-gated settlement rail compose into a coherent stack — one in which an agent's payment is *both* permissionless to initiate *and* identified, screened, and final to settle.

We are explicit about what is real and what is not. The payment substrate (switchboard) is working, tested, open-source code. The e₹ pieces are a **design-stage sandbox** that touches no real CBDC, no real UPI handle, and no real deposit, and that ships nothing before counsel review and explicit RBI/NPCI authorisation. This paper is a protocol-design contribution and a regulatory-path proposal, not a deployment.

---

## 1. Motivation

The economic case for per-request, agent-initiated payment is now familiar. A single human goal handed to an agent — "research this market," "assemble this dataset," "book this trip within these constraints" — fans out into hundreds or thousands of small purchases: one model inference, one dataset row, one crawl, one peer-agent task. Traditional web billing assumes a human creates an account, stores a card, buys credits, and rotates API keys. That model is too slow, too stateful, and too human-in-the-loop for an agent that needs to buy one thing, once, right now, and may never call that provider again ([kcolbchain Research, *Agent-to-Agent Payment Protocols in 2026*](../../briefs/agent-to-agent-payment-landscape-2026.md)).

The payments industry's answer has been to make access itself payable at the HTTP boundary. x402 turns `402 Payment Required` into a real handshake; MPP generalises machine billing across many rails; AP2 governs *authorization* and dispute evidence; Circle Nanopayments compresses settlement cost so sub-cent payments are economical. These are genuine, fast-moving primitives, and kcolbchain has surveyed them in detail and built against them (§5; the switchboard explorer compares x402 / MPP / AP2 / Circle / on-chain escrow side by side).

But a second question sits underneath "how does an agent pay?": **in what does an agent pay, and with what finality and what accountability?** Most agentic payment demos settle in a private stablecoin or an unbacked token. That works for a testnet, but it imports three problems into any system that hopes to operate at scale under real rules:

1. **Settlement-asset risk.** A private stablecoin can de-peg or fail (kcolbchain's own depeg research catalogues the history; see [*Stablecoin Depeg Events*](../../depeg_analysis.md)). If thousands of machine payments per second settle in an instrument that can break, the agent economy inherits that fragility.
2. **The accountability gap.** "The agent paid" does not answer "who is the customer, who is the merchant, and who is the liable counterparty?" Anonymous, irreversible M2M flows are exactly what AML, sanctions, and travel-rule regimes are built to scrutinise. An agentic payment rail that ignores this will not survive contact with a regulator.
3. **Reconciliation.** Netting and auditing high-volume machine payments after the fact is hard when value is a fungible scalar with no lineage.

A CBDC is a direct answer to all three. It is **final central-bank money** — there is no issuer to default and no peg to break. It is **natively identity-bound** in every framework that contemplates it — which is a feature, not a bug, for a payment class that will be held to a high compliance bar. And as a **programmable instrument**, settlement can be conditioned on machine-checkable policy. The e₹ is, today, the most advanced large-economy retail-plus-wholesale CBDC pilot in the world. The thesis of this paper is therefore narrow and concrete: **if you are going to build a settlement layer for autonomous agents, a CBDC is a better final settlement asset than a private token — and the e₹ is the natural candidate for India.**

This is a design-stage thesis. The remainder of the paper develops the architecture, grounds each component in code kcolbchain has already written, argues why a CBDC fits, and is candid about the (substantial) regulatory and technical work that stands between the design and anything live.

---

## 2. Background

### 2.1 The e₹ / CBDC reality

The e₹ is the Reserve Bank of India's CBDC — a sovereign liability issued and controlled by the central bank, not a private stablecoin and not a kcolbchain instrument. The RBI has run e₹ in **live pilots since December 2022**: a retail variant (CBDC-R) distributed to users through authorised banks, and a wholesale variant (CBDC-W) for interbank settlement, with growing UPI interoperability. Handling the e₹ — operating a wallet, integrating UPI, or running an exchange that moves between the CBDC, stablecoins, and bank fiat — is **heavily regulated** in India under the RBI, the Payment and Settlement Systems Act, 2007 (PSS Act), the RBI CBDC framework, and NPCI/UPI rules ([erupee-wallet README](https://github.com/kcolbchain/erupee-wallet)).

Two facts from this reality shape everything below. First, **the e₹ may only be handled through an RBI-authorised CBDC channel** — the pilots distribute the CBDC via authorised banks, and there is no permissionless mint. Second, **kcolbchain never issues, mints, or custodies the CBDC.** kcolbchain's role is to build the open-source, compliance-first *rails around* the instrument; any entity that actually takes custody of user e₹ is a separately-licensed Track-2 spin-out carrying its own regulatory risk. These are not caveats bolted onto the design — they are load-bearing constraints that the architecture is built to honour (§3, §6).

To reason about and prototype these rails without touching any real CBDC, the `erupee-wallet` repository runs a **sandbox**: a mock e₹ ledger with a clearly-labelled *Simulated RBI* issuer, synthetic identities and (format-only, never-verified) PANs, no outbound network I/O, and a default-deny authorisation gate that mirrors the real "authorised-channel-only" rule ([erupee-wallet SPEC.md §0, §1.5; AUTHORS.md](https://github.com/kcolbchain/erupee-wallet)). When this paper describes "the e₹ ledger," it means this sandbox model of the CBDC — the design we would seek to operate, against the real instrument, *subject to RBI approval*.

### 2.2 x402 and HTTP-native machine payment

x402 revives the `402 Payment Required` status code as a coordination point. In the core flow, a resource server responds to an unpaid request with `402` plus a machine-readable `accepts[]` envelope describing acceptable payment requirements; the client selects a scheme, constructs a signed payment payload, and retries with it attached (conventionally in an `X-PAYMENT` header); a facilitator verifies or settles the payment, and the server returns the protected resource ([Coinbase x402 docs](https://docs.cdp.coinbase.com/x402/welcome); [coinbase/x402](https://github.com/coinbase/x402)). Its strength is minimal integration friction — any HTTP route can become payable — and it is network- and token-agnostic by design. Its deliberate limitation is scope: x402 proves a payment was attached to a request; it does not, by itself, encode delegated consent, dispute evidence, refund rules, or the compliance status of the counterparties (§5).

MPP generalises this into a broader machine-billing surface (sessions, subscriptions, receipts, multiple rails); AP2 sits at the authorization layer, with Checkout Mandates, Payment Mandates, and receipts that answer "what was the agent allowed to do, and what did it actually do?"; Circle Nanopayments attacks settlement *cost* by batching off-chain USDC authorisations. kcolbchain's prior brief argues these are complementary rather than competing — x402/MPP meter access, AP2 governs authorization, Circle lowers cost — and that a substrate is needed to compose them with the operational safety controls real agents require ([*Agent-to-Agent Payment Protocols in 2026*](../../briefs/agent-to-agent-payment-landscape-2026.md)).

### 2.3 Agentic payments and their failure modes

The operational failures of autonomous payments mostly live *outside* the happy-path payment protocol. An agent in a loop can burn its whole balance on gas. A client that mismanages nonces can stall or double-spend under a reorg. An escrow with no timeout lets a silent counterparty grief the payer indefinitely. A wire format that is verbose enough for humans is wasteful on a hot agent-to-agent path. switchboard was built precisely around these failure modes: it ships hard per-hour/per-day **gas budgets**, a reorg-aware **nonce manager**, a trustless **escrow** with timeout and challenge period, and a compact binary wire (ZAP) for high-volume A2A messaging, alongside the x402 middleware ([kcolbchain/switchboard README](https://github.com/kcolbchain/switchboard)). This is the substrate the present paper extends toward CBDC-native settlement.

---

## 3. Architecture

The proposed agentic settlement stack has three planes, each grounded in built kcolbchain work:

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  PLANE 1 — ACCESS & ESCROW  (switchboard — working, tested, open source)   │
  │                                                                            │
  │   Agent A ──HTTP 402 + accepts[]──▶ Agent B        x402_middleware.py      │
  │   Agent A ──X-PAYMENT (signed)────▶ Agent B        (gates the route)        │
  │                     │                                                       │
  │                     ▼  settle larger / conditional work                     │
  │            AgentEscrow.sol  ── createPaymentWithPolicy(policyHash) ──┐      │
  │            timeout · challenge period · refund · cancel             │      │
  └────────────────────────────────────────────────────────────────────│──────┘
                                                                         │ release conditioned on
                                                                         │ a compliance attestation
                                                                         ▼
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  PLANE 2 — COMPLIANCE GATE  (erupee-wallet — design-stage sandbox)         │
  │                                                                            │
  │   compliance.precheck.evaluate(ctx) -> {decision, reasons}                 │
  │   default-deny RBI gate · KYC tier · sanctions · travel-rule thresholds    │
  │   stable reason codes: RBI_NOT_AUTHORISED, KYC_REQUIRED, KYC_TIER_LIMIT,   │
  │                        SANCTIONED_PARTY, AMOUNT_NOT_POSITIVE, …            │
  │   (peg/slippage circuit-breakers reuse SafeSwap's fail-closed gates)       │
  └────────────────────────────────────────────────────────────│──────────────┘
                                                                 │ allow → move value
                                                                 ▼
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  PLANE 3 — CBDC SETTLEMENT  (erupee-wallet — design-stage sandbox)         │
  │                                                                            │
  │   provenance ledger (wallet/ledger.py): e₹ as NOTES with full lineage      │
  │     mint / transfer (split+merge) · trace back · trace forward             │
  │   settlement layer (settlement/): net SettlementLegs into a batch and      │
  │     settle against the e₹ as the FINAL settlement asset                    │
  │                                                                            │
  │   ⚖️  e₹ is the RBI's CBDC. Only ever handled via an RBI-authorised        │
  │       channel. Sandbox here uses a Simulated RBI issuer; nothing live.     │
  └──────────────────────────────────────────────────────────────────────────┘
```

The rest of this section walks the three planes and the hinge that joins them.

### 3.1 The access & escrow plane — switchboard (built)

switchboard is the working substrate. Two of its components matter most here.

**x402 middleware** (`switchboard/x402_middleware.py`). A drop-in FastAPI/Flask middleware that gates a route behind on-chain payment: an unpaid request gets a standards-compliant `402` with the `accepts[]` envelope, and a retry carrying a verified `X-PAYMENT` signature is served ([switchboard README, §"What's in the box"](https://github.com/kcolbchain/switchboard); the middleware is shipped and unit-tested). This is the agent-facing front door — the point at which a provider agent says "this resource costs X" and a caller agent pays to proceed.

**`AgentEscrow.sol`** (`contracts/AgentEscrow.sol`, with `src/payment_protocol.py` as the Python client). For work that is larger than a single metered call, or where the payee should be paid only on acceptance, switchboard provides a trustless escrow with a `Payment` state machine — `Created → Locked → {Released | Refunded | Cancelled}` — and three baseline release paths: `confirmPayment` (payer releases to payee), `requestRefund` (payer reclaims, but only after `timeout_blocks + challengePeriod`), and `cancelPayment` (payer reclaims while still locked). The wire-level `PaymentRequest` is a canonical, sorted-key JSON object with an EIP-155 `chain_id` and a content hash, specified in [switchboard's agent-payment-protocol doc](https://github.com/kcolbchain/switchboard/blob/main/docs/agent-payment-protocol.md). These guarantee the two core escrow properties: *the payee is paid only if the work is accepted (or past a timeout the payer cannot grief), and the payer can recover funds if the payee disappears.*

Crucially for this paper, `AgentEscrow` also carries a **policy-conditioned release path**. A payment can be created via `createPaymentWithPolicy` with a non-zero `policyHash`; such a payment becomes eligible for `releaseByAttestation(requestId, attestationHash, signatures)`, which releases the escrowed funds **only if a configured oracle aggregator verifies the attestation against the declared policy** (the contract reverts otherwise, and a `policyHash` of `0x00` keeps the classic payer-only release). The contract is explicit that the aggregator is set once at construction and that `address(0)` disables oracle release entirely — a conservative default. This `policyHash` / `releaseByAttestation` mechanism is the technical hinge of the whole design (§3.4): it is exactly the seam through which escrow release can be made *conditional on a compliance verdict*.

### 3.2 The compliance gate — erupee-wallet (design-stage)

The `erupee-wallet` sandbox treats **compliance as the product, not an afterthought**. Its compliance gate is a single, well-specified function:

```
compliance.precheck.evaluate(ctx) -> { "decision": "allow" | "deny", "reasons": [code, ...] }
```

The `ctx` carries the transfer's `kind`, `from`/`to`, `amount`, the `rbi_authorised` flag, both parties' KYC tiers, a `sanctioned` flag, and the logical clock. The gate is **default-deny**: if `rbi_authorised` is falsy, it denies with `RBI_NOT_AUTHORISED`; any malformed or non-`allow` return is treated by the ledger as a deny. The stable reason-code vocabulary (`RBI_NOT_AUTHORISED`, `KYC_REQUIRED`, `KYC_TIER_LIMIT`, `SANCTIONED_PARTY`, `AMOUNT_NOT_POSITIVE`, plus the ledger-enforced `INSUFFICIENT_FUNDS`) is the contract every caller relies on ([erupee-wallet SPEC.md §2, §1.5](https://github.com/kcolbchain/erupee-wallet); the gate and ledger were implemented in [PR #1](https://github.com/kcolbchain/erupee-wallet/pull/1) and [PR #3](https://github.com/kcolbchain/erupee-wallet/pull/3)).

Three properties make this gate the right Plane-2 primitive for agentic settlement:

- **It gates *everything*.** The wallet, the exchange, and the settlement layer all call the gate before any hold, pay, swap, or settle — "no KYC + clean screen + authorised channel → no access" ([compliance/README](https://github.com/kcolbchain/erupee-wallet)). There is no path to move e₹ that bypasses it.
- **A denial is a recorded, first-class outcome.** A denied transfer is still written to the event log (`decision="deny"`, empty inputs/outputs, value unmoved), so the audit trail shows the *refusal*, not just the successes ([SPEC.md §1.2](https://github.com/kcolbchain/erupee-wallet)). For a regulated rail this is essential: you must be able to prove what you blocked.
- **It is deterministic and machine-checkable.** The verdict is a pure function of the context, which is precisely what an *attestation oracle* needs in order to sign "this settlement is compliant" (§3.4).

The same fail-closed discipline governs market-risk gates. The exchange leg of `erupee-wallet` runs hard **depeg** and **slippage** circuit-breakers (`DEPEG_BLOCKED`, `SLIPPAGE_EXCEEDED`) before any swap ([SPEC.md §7.5](https://github.com/kcolbchain/erupee-wallet)), and these reuse the design of **SafeSwap**, kcolbchain's safety-first swap orchestrator.

### 3.3 SafeSwap: fail-closed gates as the safety model

SafeSwap contributes the *shape* of the safety layer: a swap is **"both legs or neither,"** and a trade that cannot pass its gates does not execute. Its orchestrator runs five gates **in order and short-circuits on the first failure** — `simulate → depeg → slippage → private-route → atomic-settle` — so a blocked swap leaves the wallet untouched ([SafeSwap ARCHITECTURE.md](https://github.com/kcolbchain/SafeSwap); `safeswap/gates.py`). Each gate is a small `gate(intent, ctx) -> GateResult` with an explicit pass/fail and human-readable reason. Two of these gates wire directly into the rest of this stack: the `depeg` gate is the design behind the exchange's depeg circuit-breaker above, and the **`atomic-settle` gate's stated v1 target is `switchboard` escrow** — it passes only when "both chains are escrow-capable — they release together, or both refund." SafeSwap thus models, in working code, the same principle the agentic settlement layer needs: *sequential, fail-closed gates in front of value movement, with the settlement step backed by escrow.*

### 3.4 The hinge: policy-conditioned escrow release against a CBDC

The three planes compose through one mechanism. Recall that `AgentEscrow` can lock a payment under a `policyHash` and release it only when an oracle aggregator attests that the policy is satisfied (§3.1). Recall, independently, that the e₹ compliance gate produces a deterministic `allow`/`deny` verdict over a transfer context (§3.2). The design joins them: **the policy an agentic payment is escrowed under is the e₹ compliance verdict, and the attestation that releases the escrow is a signed statement that the settlement passed the gate.**

Concretely, in the design-stage flow:

1. Agent A pays Agent B for a resource over **x402** (Plane 1) for small, immediate calls; for larger or acceptance-gated work, A locks funds in **`AgentEscrow` under a `policyHash`** that commits to "settle only against an e₹ leg that the compliance gate allows."
2. When the work is delivered and an e₹ settlement is to occur, the proposed settlement context — identified parties, amount, authorised-channel flag, KYC tiers, sanctions screen — is evaluated by **`compliance.precheck.evaluate`** (Plane 2). A `deny` is recorded and the escrow is *not* released; the payer can still recover via the timeout/refund path.
3. On `allow`, an attestation (the verdict bound to the `policyHash`) is produced and the escrow releases via `releaseByAttestation`; the corresponding **e₹ leg moves through the provenance ledger** (Plane 3), and the settlement layer nets it against the CBDC.

The result is a payment that is **permissionless to *initiate*** (any agent can hit a `402`, lock an escrow) but **identified, screened, and final to *settle*** (no e₹ moves without passing the default-deny gate, and the final asset is central-bank money). This is the property we believe an agentic settlement layer that hopes to operate under real rules must have, and it falls out of composing primitives kcolbchain has already built rather than inventing a new monolith.

We stress the status honestly: the switchboard escrow, its `policyHash`/`releaseByAttestation` path, and the x402 middleware are **built and tested**; the e₹ compliance gate, provenance ledger, and settlement layer are a **design-stage sandbox over a Simulated RBI**; and the *binding* of an e₹ compliance verdict to an on-chain attestation oracle is a **design proposal in this paper**, not a deployed integration. The aggregator that would sign such attestations, and its authorisation to do so on behalf of a regulated entity, is itself gated on the regulatory path in §6.

### 3.5 The settlement plane — e₹ as final settlement asset

Underneath the gate sits the part that makes this a *settlement layer* and not just a payment relay: the **e₹ provenance ledger** and the **settlement layer** that nets against it.

The provenance ledger (`wallet/ledger.py`) models the e₹ **not as a scalar balance but as denominated notes that carry full lineage** — like a banknote serial number with the complete family tree attached. The Simulated RBI *mints* notes; *transfers* split and merge them, recording `(from, to, amount, logical-time, parent note ids)`; notes are immutable (spending a note marks it spent and creates children). From any note or wallet one can **trace back** ("how did this e₹ arrive?", note → parents → … → a mint) and **trace forward** ("how was it spent onward?"). A logical monotonic clock timestamps every event, making the system deterministic and its tests reproducible ([SPEC.md §0, §1.1](https://github.com/kcolbchain/erupee-wallet)). For high-volume machine payments this lineage is not a luxury: it is what makes after-the-fact reconciliation and audit tractable when value is moving between agents at machine speed.

The settlement layer (`settlement/`, [PR #4](https://github.com/kcolbchain/erupee-wallet/pull/4)) is the most forward-looking piece. It accepts settlement instructions, nets them as `SettlementLeg` obligations into a batch, and **settles the net positions against the e₹ as the central, final settlement asset**, with finality and audit records and reserve proofs required before any RWA-backed stablecoin in the series settles ([settlement/README](https://github.com/kcolbchain/erupee-wallet)). In the agentic context, the design intent is that a flood of small inter-agent payments need not each touch the CBDC: they can clear off-ledger or in a stablecoin leg and **net into the e₹ at settlement**, with the CBDC as the anchor of finality — the same architectural instinct as Circle's batching (§5), but with a central-bank instrument at the base. The layer is explicitly a **design target, not a live system**, dependent on RBI/NPCI authorisation for any CBDC settlement participation.

### 3.6 Illustrative model output (labelled — not a forecast)

To reason about whether netting-into-a-CBDC changes the economics, one can run a *consistency-checking* model rather than guess. kcolbchain's [token-simulator](https://github.com/kcolbchain/token-simulator) is built for exactly this: it is explicit that it models *trajectories under stated assumptions* and that **price-style point predictions are "nonsense"** while structural alarms ("burn rate exceeds circulating within N years") are real signals ("a consistency check, not a prediction"). Any figure produced for an agentic-settlement scenario — e.g. how aggregated batch settlement changes effective per-payment overhead under a chosen batch size and fee assumption — is therefore an **illustrative model output with its assumptions stated alongside it, not a benchmark, throughput claim, or forecast.** This paper deliberately reports **no fabricated performance numbers**; where modelling is warranted, it is the simulator's labelled, assumption-bound trajectories that should be cited, never a bare headline figure.

---

## 4. Why a CBDC fits agentic settlement

Section 1 stated the thesis; here we make the four-part case explicit, contrasting the CBDC against the private-stablecoin / unbacked-token default that most agentic-payment demos use today.

**4.1 Finality without issuer or peg risk.** A CBDC is a direct liability of the central bank — final money. There is no private issuer that can become insolvent and no peg that can break. For a settlement layer that may clear a high volume of machine payments, this removes a whole class of systemic fragility that private stablecoins carry, and which kcolbchain has documented at length ([*Stablecoin Depeg Events*](../../depeg_analysis.md)). An agent economy settling in central-bank money inherits the central bank's finality, not a stablecoin's tail risk. This is the single strongest argument for a CBDC base, and it is structural rather than aspirational.

**4.2 Identity-binding matches the compliance bar agentic commerce will face.** A recurring worry about M2M payments is anonymity at scale (§1, §5). A CBDC framework is identity-bound by construction; the e₹ design assumes verified holders and authorised channels. Rather than fighting this, the architecture *embraces* it: the default-deny gate (§3.2) makes "identified, screened, authorised" a precondition of settlement, not a bolt-on. The property an agentic rail needs — permissionless to initiate, accountable to settle (§3.4) — is *easier* to achieve when the settlement asset is already identity-aware than when it is an anonymous token onto which compliance must be retrofitted.

**4.3 Programmability enables policy-conditioned settlement.** A CBDC contemplated as a programmable instrument can have settlement conditioned on machine-checkable policy. That is precisely what the `policyHash` / `releaseByAttestation` hinge exploits (§3.4): the payment is released only against an attestation that the compliance policy holds. A non-programmable bank transfer cannot offer this; an unbacked token can offer the programmability but not the finality or the identity-binding. The CBDC is the instrument where *all three* — finality, identity, programmability — coincide.

**4.4 Provenance and netting suit machine-speed reconciliation.** Modelling the CBDC as notes-with-lineage (§3.5) gives every unit of settled value an auditable family tree, and the settlement layer nets high-frequency flows into the CBDC rather than touching it on every micro-payment. Together these make the two hardest operational problems of high-volume M2M payment — *audit* and *reconciliation* — tractable in a way a fungible, lineage-free scalar does not.

The honest counter-arguments are real and we hold them in view: a CBDC is **permissioned**, so the "permissionless to initiate" property holds only up to the authorised-channel boundary; CBDC rails are **jurisdiction-bound** (the e₹ is India's), so a global agent economy would need many such rails or bridges; and **availability and programmability are set by the central bank**, not by kcolbchain, so the design can only ever propose, never assume, the features it would use. Section 6 treats these as first-class limitations rather than footnotes.

---

## 5. Related work

**HTTP-native M2M payment.** x402 is the closest sibling: it makes resource access payable at the HTTP boundary with a `402` + `accepts[]` + signed-retry handshake ([Coinbase x402 docs](https://docs.cdp.coinbase.com/x402/welcome); [coinbase/x402](https://github.com/coinbase/x402); [x402.org](https://www.x402.org/)). This paper *uses* x402 as Plane 1 rather than competing with it. The difference in emphasis is the settlement asset and the compliance gate: x402 is deliberately neutral about what is paid and does not encode counterparty compliance, whereas this design pins the final asset to a CBDC and makes a compliance verdict a release condition.

**Machine billing and authorization.** MPP generalises machine billing across rails with sessions, subscriptions, and receipts ([mpp.dev](https://mpp.dev/); [paymentauth.org](https://paymentauth.org/)); AP2 governs *authorization* and dispute evidence through Checkout/Payment Mandates and receipts ([AP2 specification](https://ap2-protocol.org/ap2/specification/); [AP2 flows](https://ap2-protocol.org/ap2/flows/)). These are complementary: AP2-style mandates could sit *above* this stack to govern what an agent was authorized to buy, while this stack governs how the resulting e₹ settlement is screened and finalised. kcolbchain's own comparison of these rails, including a side-by-side explorer, is in [*Agent-to-Agent Payment Protocols in 2026*](../../briefs/agent-to-agent-payment-landscape-2026.md).

**High-frequency stablecoin clearing.** Circle Nanopayments compresses settlement cost by having buyers sign off-chain USDC authorisations, letting sellers verify instantly, and batching on-chain settlement through Circle Gateway ([Circle Nanopayments docs](https://developers.circle.com/gateway/nanopayments); [Circle blog, 2026-03-10](https://www.circle.com/blog/circle-nanopayments-launches-on-testnet-as-the-core-primitive-for-agentic-economic-activity)). The architectural instinct — net many small payments, settle in bulk — is the same one this paper's settlement layer applies (§3.5). The distinction is the base asset: Circle nets into USDC (a private stablecoin); this design nets into the e₹ (central-bank money), trading Circle's chain-and-issuer flexibility for finality and identity-binding.

**Agent-payment substrates and safety.** switchboard is the substrate this paper builds on — x402 middleware, `AgentEscrow`, gas budgets, nonce manager, ZAP wire ([kcolbchain/switchboard](https://github.com/kcolbchain/switchboard)) — and SafeSwap contributes the fail-closed, both-legs-or-neither gate model ([kcolbchain/SafeSwap](https://github.com/kcolbchain/SafeSwap)). The contribution of *this* paper relative to that body of work is the CBDC settlement plane and the `policyHash`-attestation binding that conditions release on a compliance verdict.

**CBDC and stablecoin context.** The e₹ is the RBI's CBDC, in live retail and wholesale pilots since December 2022 under the RBI CBDC framework and the PSS Act, 2007 ([erupee-wallet README, SPEC, AUTHORS](https://github.com/kcolbchain/erupee-wallet)). The settlement-asset-risk argument (§1, §4.1) draws on kcolbchain's empirical depeg work ([*Stablecoin Depeg Events*](../../depeg_analysis.md)). We do not claim novelty for the observation that CBDCs are programmable and identity-bound; the contribution is connecting that observation to a concrete, built agentic-payment substrate.

---

## 6. Limitations and regulatory path

This is a design-stage paper, and the gap between the design and anything live is large and mostly regulatory. We enumerate the limitations rather than minimise them.

**6.1 Regulatory gating is the binding constraint, not a footnote.** Nothing in the e₹ plane operates without explicit RBI/NPCI authorisation and PSS-Act licensing held by the operating entity. The e₹ may only be handled through an RBI-authorised CBDC channel; there is no permissionless path. Per kcolbchain's two-track model, **kcolbchain supplies the open-source framework and takes no custody and holds no deposits**, while any commercial wallet/exchange/settlement operator is a **separate, separately-licensed Track-2 entity** carrying its own regulatory risk (the GANGES / INDR pattern) ([erupee-wallet README §"Two-track model", AUTHORS.md](https://github.com/kcolbchain/erupee-wallet)). Named counterparties such as SBI are *stated, not finalised*. The attestation oracle proposed in §3.4 would, in any live system, require its own authorisation to attest compliance on behalf of a regulated entity — it cannot be assumed.

**6.2 The e₹ pieces are a sandbox, with no real CBDC, money, or PII.** The `erupee-wallet` ledger, gate, and settlement layer run against a **Simulated RBI** with **synthetic identities only**, **format-only (never-verified) PANs**, **no outbound network I/O**, and **no custody or deposits** ([SPEC.md §0; AUTHORS.md](https://github.com/kcolbchain/erupee-wallet)). The sandbox is a faithful *model* of the rails we would seek to operate; it is explicitly not the rails themselves. Mapping the sandbox onto a real RBI-authorised channel is unbuilt and is the dependency that gates everything downstream.

**6.3 The CBDC↔attestation binding is a proposal, not an integration.** The technical hinge (§3.4) relies on switchboard's `releaseByAttestation` (built and tested) being driven by an oracle that signs e₹ compliance verdicts. That oracle, its trust model, its key management, and its authorisation are **designed but not implemented**. The contract's conservative defaults (oracle release disabled unless an aggregator is set at construction; `policyHash` of `0x00` keeps payer-only release) are deliberate, but they do not by themselves make the attestation *trustworthy* — that is future work and a governance question, not a coding task.

**6.4 CBDC-specific constraints we do not control.** A CBDC is permissioned and jurisdiction-bound, and its availability and programmability are set by the central bank (§4, closing paragraph). The "permissionless to initiate" property therefore holds only up to the authorised-channel boundary; cross-border or multi-jurisdiction agent settlement would need multiple CBDC rails, bridges, or stablecoin legs that net into the respective CBDCs. We cannot assume the e₹ will expose the programmability the policy-conditioned design would ideally use; the design must degrade gracefully to gate-then-settle even without on-instrument programmability.

**6.5 No performance claims.** This paper reports **no benchmarks and no throughput, latency, or revenue numbers** for the e₹ plane. The switchboard test suite demonstrates that the *substrate's* components pass their unit tests, but that is a correctness signal, not a performance claim. Any economic or volume figure must be a clearly-labelled, assumption-bound output of the consistency-checking simulator (§3.6), and even then is a model, not a forecast.

**6.6 Disputes, refunds, and consent are out of scope here.** This paper governs *settlement* (is the payment final and screened?), not *commerce authorization* (was the agent allowed to make this purchase, and what happens in a dispute?). The latter is AP2's territory (§5) and would need to sit above this stack. We deliberately do not claim to solve it.

**The regulatory path, stated plainly.** The order of operations we would follow is: (1) keep the e₹ work a sandbox over a Simulated RBI; (2) take the compliance posture, the default-deny gate, and the provenance/audit model to counsel and to the RBI/NPCI as a *design* for a compliant CBDC-native settlement rail; (3) build nothing that touches a real e₹, UPI handle, or deposit before explicit RBI/NPCI authorisation and PSS-Act licensing are held by an operating Track-2 entity; (4) only then implement and independently review the attestation oracle and the live CBDC channel integration. None of this paper's design is an authorisation, an offer, or financial or legal advice.

---

## 7. Future work

- **Implement and harden the attestation oracle.** Specify the oracle aggregator that signs e₹ compliance verdicts for `releaseByAttestation`, its threshold/multi-sig trust model, key management, and revocation — then subject it to independent review. This is the single most important unbuilt piece (§6.3).
- **Wire the e₹ gate to switchboard escrow end-to-end in the sandbox.** Demonstrate the full §3.4 flow against the Simulated RBI: x402 access → `policyHash` escrow → `compliance.precheck.evaluate` → `releaseByAttestation` → provenance-ledger settlement, with denials correctly recorded and escrows correctly refunded. This validates the design without touching any real CBDC.
- **Model the netting economics, honestly.** Use the [token-simulator](https://github.com/kcolbchain/token-simulator) to produce labelled, assumption-bound trajectories for how batch settlement into the e₹ changes effective per-payment overhead under varying batch sizes and fee assumptions — as consistency checks, never as forecasts or benchmarks.
- **Cross-rail and CAIP-2 generalisation.** switchboard's protocol already anticipates extending `chain_id` to CAIP-2 strings for non-EVM chains; explore how a CBDC settlement plane composes with multiple jurisdictions' rails and with stablecoin legs that net into the respective CBDCs (§6.4).
- **Compose with AP2-style authorization above the settlement plane.** Sketch how Checkout/Payment Mandates could sit above this stack so that *what an agent was authorized to buy* and *whether the resulting settlement is screened and final* are governed by complementary layers (§5, §6.6).
- **Post-quantum and wire-format hardening on the hot path.** switchboard already tracks PQ signatures and the ZAP binary wire; evaluate their interaction with attestation-gated settlement for high-volume A2A traffic.

---

## References

- AP2 Protocol. "Agentic Payment Protocol (v0.2)." 2025. https://ap2-protocol.org/ap2/specification/. Accessed 2026-06.
- AP2 Protocol. "Flows." 2025. https://ap2-protocol.org/ap2/flows/. Accessed 2026-06.
- Circle. "Nanopayments." 2026. https://developers.circle.com/gateway/nanopayments. Accessed 2026-06.
- Circle. "Circle Nanopayments Launches on Testnet as the Core Primitive for Agentic Economic Activity." 2026-03-10. https://www.circle.com/blog/circle-nanopayments-launches-on-testnet-as-the-core-primitive-for-agentic-economic-activity. Accessed 2026-06.
- Coinbase Developer Platform. "x402 Overview." https://docs.cdp.coinbase.com/x402/welcome. Accessed 2026-06.
- coinbase/x402. "README." https://github.com/coinbase/x402. Accessed 2026-06.
- kcolbchain Research. "Agent-to-Agent Payment Protocols in 2026." June 2026. [`briefs/agent-to-agent-payment-landscape-2026.md`](../../briefs/agent-to-agent-payment-landscape-2026.md).
- kcolbchain Research. "Stablecoin Depeg Events." May 2026. [`depeg_analysis.md`](../../depeg_analysis.md).
- kcolbchain/erupee-wallet. "README, SPEC.md, AUTHORS.md, compliance/README, settlement/README." e₹ provenance ledger + compliance gate ([PR #1](https://github.com/kcolbchain/erupee-wallet/pull/1), [PR #3](https://github.com/kcolbchain/erupee-wallet/pull/3)); e₹ settlement layer ([PR #4](https://github.com/kcolbchain/erupee-wallet/pull/4)). https://github.com/kcolbchain/erupee-wallet.
- kcolbchain/SafeSwap. "README, ARCHITECTURE.md, `safeswap/gates.py`." Fail-closed, both-legs-or-neither swap orchestrator. https://github.com/kcolbchain/SafeSwap.
- kcolbchain/switchboard. "README, `docs/agent-payment-protocol.md`, `contracts/AgentEscrow.sol`, `switchboard/x402_middleware.py`." x402 middleware + AgentEscrow with `policyHash` / `releaseByAttestation` ([PR #131](https://github.com/kcolbchain/switchboard/pull/131), [PR #132](https://github.com/kcolbchain/switchboard/pull/132)). https://github.com/kcolbchain/switchboard.
- kcolbchain/token-simulator. "README, `token_simulator/`." Consistency-checking tokenomics simulator (not a price predictor). https://github.com/kcolbchain/token-simulator.
- Machine Payments Protocol. "MPP." https://mpp.dev/. Accessed 2026-06.
- Machine Payments Protocol. "Specifications." https://paymentauth.org/. Accessed 2026-06.
- Reserve Bank of India. CBDC (e₹) retail (CBDC-R) and wholesale (CBDC-W) pilots, live since December 2022; RBI CBDC framework and the Payment and Settlement Systems Act, 2007. As summarised in the erupee-wallet README/SPEC; consult RBI primary sources for authoritative detail.
- x402.org. "Payment Required — Internet-Native Payments Standard." https://www.x402.org/. Accessed 2026-06.

---

*The e-rupee (e₹) is the Reserve Bank of India's CBDC; kcolbchain is independent of and unaffiliated with the RBI, and does not issue, mint, or custody the CBDC. This paper is informational research only — not legal, tax, or financial advice, and not an offer of any security, token, or payment service. Everything touching the e₹ is design-stage and subject to the approvals and licences we will seek. Conceived by Patty (P. Sundaram), partner — KCOLBCHAIN, with Abhishek Krishna (@abhicris).*
