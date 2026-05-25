# MEV on L2 Rollups: Landscape, Mechanisms, and Design Tradeoffs

**Date:** May 2026

## Abstract

Maximal extractable value (MEV) on Ethereum L2 rollups differs fundamentally from L1 MEV. The centralized or tightly controlled sequencer model gives the ordering entity a privileged view of transaction flow, creating value extraction opportunities that resemble L1 proposer power but without competitive block-building. This brief surveys the MEV landscape on L2s, covering sequencer-extracted MEV, shared sequencer proposals (Espresso, Astria), Flashbots SUAVE, Arbitrum Timeboost, and OP Stack revenue mechanisms.

## 1. Why L2 MEV Is Different

On Ethereum L1, MEV is intermediated by a market of proposers, builders, relays, and searchers. No single actor is expected to order all transactions permanently. Rollups change this: the sequencer receives, orders, batches, and posts all transactions to L1. This creates three structural advantages:

- **Information advantage**: the sequencer sees all pending order flow before external actors
- **Latency advantage**: it can insert, delay, or prioritize its own transactions with zero network latency
- **Discretion advantage**: without transparent ordering rules, users cannot distinguish benign batching from value extraction

L2BEAT's risk framework tracks sequencer centralization across rollups, showing that most production systems still depend on a single sequencer for liveness and censorship resistance.

## 2. Sequencer-Extracted MEV

In the baseline model, the sequencer captures the spread between L2 fees collected and L1 data posting costs, plus any profitable ordering strategies it can execute. This includes:

- Latency arbitrage between L2 venues
- Backrunning of large swaps
- Liquidation prioritization
- Priority access sold to sophisticated participants

The advantage is inherent—the sequencer controls ordering. The tradeoff is between operational simplicity (centralized sequencers are fast and easy to operate) and value allocation (users bear costs through worse execution and opaque routing).

**Protocol-level mitigations** include public ordering policies, fee-vault transparency, and governance oversight of sequencer revenue use.

## 3. Shared Sequencers

Shared sequencers (Espresso, Astria) separate ordering from execution by providing a common sequencing layer for multiple rollups.

### Espresso
Espresso provides a shared sequencer with HotShot consensus (BFT, 1-2s finality) and a marketplace for sequencing services. Rollups retain their own execution while receiving ordered blocks from Espresso. The MEV implication: ordering rights become a competitive market rather than a private operator privilege.

### Astria
Astria's modular sequencing layer orders transactions without executing them. Rollups receive ordered blocks and run their own state transitions. This design emphasizes sovereignty—rollups outsource ordering without outsourcing application logic.

**Key challenge**: bootstrapping adoption. A shared sequencer needs enough rollups and users routing order flow through it to provide competitive latency and liveness guarantees.

## 4. Flashbots SUAVE

SUAVE (Single Unified Auction for Value Expression) proposes a domain-neutral preference-execution market. Users express transaction preferences and executors compete to satisfy them across chains.

For L2s, SUAVE addresses cross-domain MEV—a user's optimal execution path may span L1, multiple rollups, and CEX venues. SUAVE's thesis is that order flow should be auctioned in a shared market rather than internalized by each domain's privileged sequencer.

**Current status**: evolving architecture. SUAVE should be evaluated as a credible direction for intent-based execution rather than a production standard.

## 5. Arbitrum Timeboost

Timeboost introduces an explicit priority auction. Searchers bid for a short time window of transaction priority. Regular transactions continue through the standard path.

**Design goals**: replace opaque latency competition with priced, bounded priority; direct auction proceeds to the DAO.

**Limitations**: does not solve backrunning, cross-domain MEV, or sophisticated-actor concentration. Its main contribution is governance clarity—the protocol defines and sells priority explicitly.

## 6. OP Stack Sequencer Revenue

The OP Stack routes L2 fees—both execution and L1 data posting costs—into the Superchain ecosystem. OP Mainnet contributes 100% of net transaction-fee profit; member chains contribute the greater of 2.5% of gross fees or 15% of net profit.

**Important distinction**: fee revenue does not equal MEV. Fee flows can be tracked from vault addresses and documentation; MEV requires transaction-level strategy analysis.

## 7. Comparative Summary

| Approach | MEV Reduction | Complexity | Centralization | Status |
|----------|--------------|------------|----------------|--------|
| Centralized sequencer | None | Low | High | Production |
| Timeboost (priority auction) | Moderate | Medium | Low | Proposed |
| Shared sequencer (Espresso) | Moderate | High | Low | Testnet |
| SUAVE (intent auction) | High | Very High | Low | Development |
| Encrypted mempool (Radius) | High | Very High | Low | Research |

## 8. Data Sources

- L2BEAT Risk Framework: https://l2beat.com/scaling/risk
- Flashbots Transparency Dashboard
- Dune Analytics (L2 fee and MEV dashboards)
- EigenPhi MEV Analytics: https://eigenphi.io
- Arbitrum Research Forum: https://research.arbitrum.io
- Optimism Docs: https://docs.optimism.io

## References

1. Daian, P., et al. "Flash Boys 2.0." IEEE S&P, 2020.
2. L2BEAT. "Scaling Risk Analysis." https://l2beat.com/scaling/risk
3. Arbitrum DAO Forum. "Constitutional AIP: Timeboost." 2024.
4. Optimism Docs. "Transaction Fees on OP Stack." https://docs.optimism.io
5. Espresso Systems. "Rollup Architecture." https://docs.espressosys.com
6. Astria Docs. "The Astria Sequencing Layer." https://docs.astria.org
7. Flashbots. "The Future of MEV is SUAVE." https://writings.flashbots.net
8. Flashbots. "MEV and the Limits of Scaling." 2025.
9. Ben Weintraub et al. "Rolling in the Shadows: Analyzing MEV Across L2 Rollups." arXiv:2405.00138.
