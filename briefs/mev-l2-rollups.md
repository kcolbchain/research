# MEV on L2 Rollups: Sequencer Extraction, Shared Sequencers, SUAVE, Timeboost, and OP Stack Revenue

**Date:** May 2026

**Scope:** 2000-3000 word research brief for kcolbchain/research issue #1

## Executive Summary

Maximal extractable value (MEV) on Ethereum L2 rollups is shaped less by proof-of-stake validator competition and more by sequencer design. Most production rollups still rely on a centralized or tightly controlled sequencer for low-latency ordering, so the entity that accepts, orders, and batches transactions receives a privileged view of order flow. That makes L2 MEV a governance and market-design problem: who controls ordering, who captures latency rents, and whether users or public goods receive any rebate from the value their activity creates.

This brief compares four approaches. First, centralized sequencer extraction gives one operator the best latency and information position, which is operationally simple but creates opacity, censorship risk, and an incentive to monetize order flow privately. Second, shared sequencers such as Espresso and Astria separate transaction ordering from rollup execution, offering a path toward common pre-confirmations, cross-rollup composability, and more neutral ordering markets. Third, Flashbots SUAVE proposes a domain-neutral auction and execution layer where users express preferences and competitive executors route order flow across chains. Fourth, Arbitrum Timeboost and OP Stack fee systems show how leading optimistic rollup ecosystems are already turning sequencing rights and transaction fees into protocol revenue.

The central conclusion is pragmatic: L2 MEV is unlikely to disappear. The credible design space is about disclosure, competition, allocation, and constraints on harmful ordering. Rollups that keep centralized sequencers need explicit policies for priority access and revenue use. Rollups that decentralize sequencing need to prove that added coordination cost does not break latency, liveness, or user experience.

## 1. Why L2 MEV Is Different

On Ethereum L1, MEV is mediated by proposers, builders, relays, searchers, and public or private order flow. The market is still concentrated in places, but no single application-layer operator is expected to be the only long-running orderer for all transactions. Rollups change that architecture. A rollup sequencer commonly receives user transactions, assigns an order, gives users a fast confirmation, and later posts data or commitments to Ethereum.

That position creates three MEV advantages. The first is information: the sequencer can see incoming order flow before most other actors. The second is latency: it can insert, delay, or prioritize transactions faster than an external searcher submitting through the public path. The third is discretion: if ordering rules are not explicit or externally enforced, users cannot easily distinguish benign batching from value extraction.

The risks are not only theoretical. L2BEAT tracks sequencer centralization and related escape-hatch assumptions across rollups, and its risk framework highlights that many systems still depend on one operator for transaction sequencing and censorship resistance assumptions. Academic work on rollups has also emphasized that centralized sequencing can create side channels and ordering advantages on the critical path from transaction submission to confirmation. The practical result is that an L2 can be cheaper and faster than L1 while also concentrating the most valuable ordering position.

## 2. Sequencer-Extracted MEV

In the simplest model, the sequencer captures the spread between what users pay for L2 execution and what the operator pays for L1 data availability, while also benefiting from any private ordering strategy it can legally and technically run. This can include latency arbitrage, backrunning, liquidations, and priority access sold to sophisticated participants. It does not require an explicit "MEV module"; the advantage exists because the sequencer controls ordering.

The model has real benefits. It is easier to operate, easier to optimize for low latency, and easier to reason about during incident response. A single sequencer can provide rapid soft confirmations without waiting for a validator committee. That is one reason centralized sequencing has remained common even among teams with decentralization roadmaps.

The tradeoff is value allocation. If MEV is extracted privately, users pay the cost through worse execution, less transparent routing, or order-flow discrimination. If MEV is captured by the protocol, it can fund public goods, security, or ecosystem incentives, but it still requires governance over who gets priority and under what rules. A centralized sequencer without a public policy leaves reviewers with no durable way to audit fairness.

For data analysis, researchers should separate three streams that are often mixed together. Transaction fees are visible on-chain and can be reconciled from fee vaults or chain-level dashboards. Sequencer gross revenue is broader because it includes L2 fees before L1 posting costs. MEV revenue is narrower and harder to measure because it requires identifying profitable ordering-sensitive strategies. Public dashboards from Dune, Blockworks Research, L2BEAT, EigenPhi, and protocol-specific analytics can support estimates, but most MEV figures should be presented as estimates rather than accounting facts.

## 3. Shared Sequencers

Shared sequencers try to remove the ordering monopoly from an individual rollup. Instead of every chain operating its own sequencer, multiple rollups can send transactions to a common sequencing layer that provides ordering, pre-confirmations, and sometimes cross-domain coordination. Execution remains inside each rollup, but the ordering service becomes shared infrastructure.

Espresso is a leading example. Its documentation describes a network that rollups can integrate with to receive confirmations from Espresso while retaining their own execution logic. Espresso's design centers on HotShot consensus and a marketplace for sequencing services, aiming to give rollups faster interoperability and a more decentralized ordering layer than a single in-house sequencer. The MEV implication is that ordering rights can become a competitive market rather than a private operator privilege.

Astria takes a modular sequencing approach. Its sequencing layer orders rollup transactions but does not execute them. The rollup receives ordered blocks from Astria and applies its own state transition function. That separation is important: it means a rollup can outsource ordering and pre-confirmation without outsourcing application logic or settlement. Astria's docs frame the sequencer as a service for sovereign rollups that want fast confirmations and shared liveness without each team bootstrapping a full sequencing network.

Shared sequencing can also improve cross-rollup UX. If two rollups use the same ordering layer, cross-domain messages and arbitrage can be sequenced with better coordination. That may reduce some race conditions and make cross-rollup applications less fragmented. It can also make cross-domain MEV more visible, because the ordering layer sees multiple domains rather than one chain in isolation.

The hard parts are incentives and adoption. A shared sequencer only becomes valuable when enough rollups and users route order flow through it. It must also decide how to allocate sequencing fees, MEV, and responsibility for downtime. A rollup that already earns significant sequencer revenue may hesitate to hand ordering to an external network unless the shared layer improves security, regulatory posture, or user acquisition enough to compensate.

## 4. Flashbots SUAVE on L2

SUAVE, short for Single Unified Auction for Value Expression, is Flashbots' proposal for separating the expression of transaction preferences from the execution and block-building market. Rather than treating each chain's mempool as an isolated venue, SUAVE is designed as a domain-neutral environment where users, wallets, searchers, and builders can express preferences and compete to satisfy them.

The L2 relevance is cross-domain MEV. A user's best execution path may involve Ethereum L1, an optimistic rollup, a zk rollup, a centralized exchange bridge, and multiple liquidity venues. A single rollup sequencer can only optimize a narrow slice of that route. SUAVE's thesis is that order flow should be auctioned and executed in a shared market where competition can return more value to users or applications, instead of letting each domain's privileged orderer internalize the opportunity.

SUAVE is not a drop-in replacement for every rollup sequencer. It introduces integration complexity, trust and privacy assumptions, and latency questions. Flashbots' public writing describes SUAVE as an evolving architecture for MEV markets, not as a fully mature production standard. That distinction matters for research: SUAVE should be evaluated as a credible direction for intent-based and cross-domain execution, while current rollup revenue models should be evaluated as production mechanisms.

For L2 teams, the strategic question is whether to keep MEV local or route it to a broader market. Local capture is easier to govern and can directly fund the rollup. A SUAVE-like market may produce better execution and reduce harmful information asymmetry, but it requires users and applications to trust a more complex auction stack. The likely near-term outcome is hybridization: wallets and applications may use intent or auction systems for high-value order flow while ordinary transactions continue through native sequencers.

## 5. Arbitrum Timeboost

Arbitrum Timeboost is one of the clearest examples of making L2 priority access explicit. Instead of leaving latency races to private infrastructure, Timeboost introduces an auction for an "express lane" advantage. Arbitrum governance posts and research describe a mechanism where a bidder can win the right to submit transactions with priority for a short time window, while regular transactions continue through the ordinary path.

The design goal is not to remove MEV. It is to replace opaque latency competition with a priced and bounded priority market. Searchers that previously competed by sending many transactions or optimizing private connectivity can instead bid for time advantage. The protocol can then direct auction proceeds to the DAO or another governed destination.

Timeboost also reflects a key L2 reality: users value fast confirmations, and rollups do not want to degrade the base user experience just to decentralize every step immediately. A priority auction can preserve a fast centralized sequencer path while constraining how priority is sold. That makes it more practical than a full shared sequencer migration for a mature chain with existing liquidity and applications.

The mechanism still has limitations. Sophisticated actors are better positioned to price priority, and auction design can centralize around firms with better MEV models. Timeboost also does not solve all forms of backrunning or cross-domain MEV. Its main contribution is governance clarity: the rollup publicly defines the priority product, sells it through a protocol-level mechanism, and can account for the proceeds.

Researchers tracking Timeboost should avoid relying only on headline annualized revenue estimates. The better method is to monitor official Arbitrum documentation, DAO governance posts, and dashboards that break out auction bids, payments, express-lane usage, and destination addresses. Those data sources can show whether the mechanism is reducing spam, concentrating winners, or creating durable DAO revenue.

## 6. OP Stack Sequencer Revenue

The OP Stack shows a different path: standardize rollup infrastructure and route part of chain economics back to the Optimism Collective. OP Stack transaction fees include L2 execution fees and L1 data fees. Optimism's fee documentation explains that users pay for execution on the L2 and for the cost of publishing transaction data to Ethereum, with post-Ecotone fee logic also accounting for blob-related costs.

Revenue attribution is therefore more nuanced than "the sequencer gets all fees." A chain operator may collect L2 base and priority fees through fee vaults, pay L1 data costs, and then share revenue under Superchain agreements. Optimism documentation states that Superchain member chains contribute the greater of 2.5% of gross transaction fees or 15% of net transaction-fee profit to the Optimism Collective, while OP Mainnet contributes 100% of its net transaction-fee profit.

This model turns sequencer economics into an ecosystem funding mechanism. Instead of only charging a franchise fee or relying on token issuance, the Superchain can connect usage on member chains to public goods funding and governance. That may become increasingly important if many OP Stack chains share upgrades, security assumptions, developer tooling, and interoperability standards.

The MEV question remains only partly answered. Fee sharing is transparent when it concerns fee vaults and reported revenue, but it does not automatically prove that harmful ordering MEV is absent or rebated. Researchers should distinguish OP Stack fee revenue from MEV extraction. The former can be tracked from docs, vault addresses, and public reporting. The latter requires transaction-level strategy analysis, comparable prices across venues, and evidence that ordering affected execution.

## 7. Comparative View

| Model | Main benefit | Main risk | Best current use case |
| --- | --- | --- | --- |
| Centralized sequencer | Lowest operational complexity and fast soft confirmations | Opaque value extraction and censorship risk | Early or high-throughput rollups optimizing UX |
| Protocol priority auction, such as Timeboost | Public price for latency advantage and DAO revenue | Sophisticated actors may dominate auctions | Mature rollups that want to monetize priority without full sequencing decentralization |
| OP Stack revenue sharing | Turns chain usage into ecosystem funding | Fee revenue transparency does not equal MEV transparency | Multi-chain ecosystems with shared governance and standards |
| Shared sequencer | More neutral ordering and cross-rollup coordination | Bootstrapping, latency, and incentive complexity | Rollups seeking decentralization or interoperability as a differentiator |
| SUAVE-like auction layer | Cross-domain execution and user preference markets | Integration, privacy, and maturity risk | High-value order flow, intents, and multi-chain execution |

## 8. Data Sources and Caveats

Reliable L2 MEV research should combine at least four source types.

First, protocol documentation defines the mechanism. Arbitrum governance posts and research explain Timeboost's priority auction; Optimism docs explain OP Stack fees and revenue sharing; Flashbots writing explains SUAVE's goals and architecture; Espresso and Astria docs explain shared sequencing assumptions.

Second, on-chain accounting shows fee flows. For OP Stack chains, fee vaults, L1 data costs, and chain-operator reporting are stronger evidence than social posts. For Arbitrum Timeboost, auction payment addresses and DAO reporting are stronger than annualized screenshots.

Third, market analytics estimate extractable value. Dune dashboards, Blockworks Research, L2BEAT, EigenPhi, and protocol-specific dashboards can help quantify revenue and concentration, but methods vary. Any daily or monthly MEV number should include a method note.

Fourth, transaction-level studies identify user harm. Sandwiching, latency arbitrage, and liquidations require strategy classification, not just fee accounting. This is where academic work and reproducible open-source analysis are most valuable.

The largest caveat is that "sequencer revenue" and "MEV" are not synonyms. Sequencer revenue can be legitimate fees net of L1 costs. MEV is value from ordering, inclusion, exclusion, or information advantage. A rollup can have high sequencer revenue with little harmful MEV, or low visible revenue with significant private extraction.

## Conclusion

MEV on L2 rollups is a design choice hidden inside sequencing architecture. Centralized sequencers are fast and practical, but they concentrate the ordering position. Shared sequencers and SUAVE-like systems broaden competition, but they add coordination and integration costs. Arbitrum Timeboost and OP Stack revenue sharing show that production rollups are already experimenting with explicit ways to monetize or redirect sequencing value.

The strongest near-term standard is not "no MEV." It is transparent mechanism design: public ordering rules, accountable revenue destinations, clear user protections, and reproducible data. As L2 activity keeps moving away from Ethereum L1 execution, the credibility of rollups will depend on whether they can make sequencing power legible and contestable.

## References

1. L2BEAT, "Scaling risk analysis." https://l2beat.com/scaling/risk
2. Ben Weintraub et al., "Rolling in the Shadows: Analyzing the Extraction of MEV Across Layer-2 Rollups." https://arxiv.org/abs/2405.00138
3. Arbitrum DAO Forum, "Constitutional AIP: Proposal to adopt Timeboost, a new transaction ordering policy." https://forum.arbitrum.foundation/t/constitutional-aip-proposal-to-adopt-timeboost-a-new-transaction-ordering-policy/25167
4. Arbitrum Research, "Decentralized Timeboost Specification." https://research.arbitrum.io/t/decentralized-timeboost-specification/9676
5. Arbitrum, "Gattaca Titan: Timeboost Live on Arbitrum." https://blog.arbitrum.io/gattaca-titan-timeboost-live-on-arbitrum/
6. Optimism Docs, "Transaction fees on OP Stack." https://docs.optimism.io/op-stack/transactions/fees
7. Optimism Docs, "Capital allocation." https://docs.optimism.io/governance/capital-allocation
8. Optimism Docs, "The OP Stack." https://docs.optimism.io/op-stack/introduction/op-stack
9. Flashbots, "The Future of MEV is SUAVE." https://writings.flashbots.net/the-future-of-mev-is-suave
10. Flashbots, "SUAVE specs." https://github.com/flashbots/suave-specs
11. Espresso Systems, "FAQ." https://www.espressosys.com/faq
12. Espresso Systems Docs, "Rollup architecture." https://docs.espressosys.com/network/concepts/rollup-architecture
13. Astria Docs, "The Astria Sequencing Layer." https://docs.astria.org/overview/components/the-astria-sequencing-layer
