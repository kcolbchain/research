# MEV on Layer 2 Rollups: Mechanisms, Extraction Models, and Emerging Solutions

## Abstract

Maximal Extractable Value (MEV) on Layer 2 (L2) rollups presents a distinct set of challenges and opportunities that differ fundamentally from Ethereum Layer 1 dynamics. Unlike L1 where MEV extraction is distributed among validators through public mempool competition, L2 rollups depend on centralized or semi-decentralized sequencers that have significant control over transaction ordering and inclusion. This brief examines the current state of MEV on L2 rollups, comparing sequencer-extracted MEV models with emerging shared sequencer architectures. We analyze three pivotal case studies—Arbitrum Timeboost, Optimism's OP Stack fee structure, and Flashbots SUAVE—to illustrate both the current approaches to MEV monetization and the future of MEV-resistant infrastructure. The brief concludes that the evolution from centralized sequencer MEV capture toward decentralized auction mechanisms and shared sequencer networks represents a critical path for L2 scalability while preserving user fairness and network security.

---

## 1. Introduction: MEV in the L2 Context

MEV, the maximum value that can be extracted from block production through transaction reordering, inclusion, and exclusion, has become one of the most studied phenomena in blockchain economics. However, the dynamics of MEV differ significantly between Ethereum L1 and L2 rollups due to fundamental architectural differences.

On Ethereum L1, MEV extraction occurs through a competitive process: independent searchers identify profitable opportunities (arbitrage, liquidations, sandwich attacks), and validators/block builders prioritize transactions based on fee offers. This competition, while creating negative externalities like network congestion, is at least distributed across a large validator set.

L2 rollups, by contrast, typically employ a small number of sequencers—often just one—to aggregate and order transactions before submitting batches to the L1 data availability layer. This centralization creates a critical asymmetry: the sequencer has unilateral control over MEV extraction, with minimal competitive pressure to pass MEV value back to users or other network participants.[^1]

The question of how to manage sequencer MEV has become strategic for the L2 ecosystem. Current approaches fall into two categories:

1. **Sequencer-extracted MEV models**: Centralized sequencers capture MEV directly and negotiate revenue-sharing with the rollup's governance or DAO
2. **Shared sequencer and auction-based models**: Decentralized networks compete to sequence transactions, with mechanisms to return MEV value to users or the protocol

---

## 2. Sequencer-Extracted MEV vs. Shared Sequencer Models

### 2.1 Centralized Sequencer MEV

Most L2 rollups today operate with centralized sequencers. Common forms of MEV extraction in this model include:

- **Ordering MEV**: The sequencer prioritizes high-value transactions or inserts its own arbitrage trades
- **Latency MEV**: Strategic transaction delays to create profitable opportunities
- **Frontrunning and sandwich attacks**: Inserting transactions before or after user transactions to capture value

The centralized sequencer benefits from several advantages: faster finality, simpler implementation, and lower operating costs. However, these benefits come at the cost of:

- **Lack of transparency**: MEV extraction is often opaque to users
- **Single point of failure**: Sequencer downtime or misconduct affects the entire rollup
- **Censorship risk**: A centralized sequencer can exclude specific transactions or users
- **Value leakage**: Users derive no benefit from MEV their transactions create[^2]

### 2.2 Shared Sequencer Models

Recognizing these limitations, several projects are exploring shared sequencer architectures where multiple L2s or applications share a decentralized sequencer network. Prominent examples include:

**Espresso Systems**: Provides a "plug-and-play" sequencer network that multiple L2s can use while maintaining their own execution layers. Espresso aims to reduce MEV by distributing sequencing across a decentralized validator set and enabling cross-domain MEV aggregation.

**Astria**: A modular sequencer network designed for Cosmos-style rollups and Ethereum-aligned L2s. Astria's approach emphasizes order fairness through BFT (Byzantine Fault-Tolerant) consensus, where transactions are processed based on reception time rather than profitability.

**Shared Sequencers**: This model allows multiple L2 rollups to use the same decentralized sequencer network, providing several advantages:
- **MEV decentralization**: Sequencing power is distributed among many validators
- **Cross-domain MEV optimization**: A shared sequencer can coordinate MEV extraction across multiple chains
- **Censorship resistance**: No single actor can censor transactions
- **Interoperability**: Users experience smoother cross-rollup interactions

However, shared sequencers introduce their own challenges: network coordination, validator set scaling, and the complexity of allocating MEV revenue across multiple rollups.[^3]

---

## 3. Case Study: Arbitrum Timeboost

Arbitrum's Timeboost represents one of the most sophisticated approaches to sequencer MEV monetization on L2s. Rather than either fully centralizing or fully decentralizing sequencing, Timeboost implements a structured auction mechanism that balances MEV capture with user protection.

### 3.1 How Timeboost Works

Timeboost operates as a sealed-bid, second-price auction system:[^4]

1. **Express Lane**: Users and MEV searchers can submit transactions with sealed bids to gain priority placement
2. **Auction Mechanics**: The highest bidder wins the right to express-lane sequencing for a 60-second time slot, but only pays the second-highest bid
3. **Regular Transactions**: Non-bidding transactions are sequenced with slight delay on a separate track, protecting them from reordering by express-lane participants
4. **Zero-Knowledge Guarantees**: The sequencer cannot reorder mempool transactions arbitrarily—express-lane winners only gain temporal priority, not visibility into pending transactions

### 3.2 MEV Impact and Value Capture

Timeboost's design achieves several key objectives:

**MEV Monetization**: By auctioning the right to priority sequencing, Arbitrum captures MEV revenue that would otherwise accrue exclusively to searchers. Since launch in April 2025, Timeboost has generated substantial fees for the Arbitrum DAO, with revenue flowing directly to governance.

**Reduction of Network Spam**: Under the previous FCFS (First-Come, First-Served) model, MEV searchers would "spam" the network with numerous transaction attempts to win latency races, causing congestion. Timeboost eliminates this incentive by offering a guaranteed priority slot for a fixed bid, reducing transaction volume and improving network efficiency.

**User Protection**: Critically, Timeboost protects non-participating users from harmful MEV. Because express-lane transactions cannot access the mempool, sandwich attacks and front-running are impossible. Users can either participate in the auction or accept the slight delay of regular sequencing.

**Revenue Sustainability**: The DAO-captured MEV provides sustainable funding for sequencer operations and ecosystem development, reducing reliance on conventional gas fees.

### 3.3 Decentralization Compatibility

A key design principle of Timeboost is compatibility with future decentralized sequencers. The auction mechanism is protocol-level, meaning any sequencer—centralized or decentralized—can implement it. This allows Arbitrum to capture MEV value today while maintaining a clear path to sequencer decentralization.[^5]

---

## 4. Case Study: Optimism OP Stack Sequencer Revenue

Optimism's approach to sequencer revenue differs from Arbitrum in its focus on the Superchain model: a network of OP Stack-compatible L2s that share infrastructure and governance while maintaining separate execution layers.

### 4.1 Fee Structure

OP Stack chains collect transaction fees in three components:[^6]

1. **Priority Fees** (EIP-1559): Paid to the Sequencer Fee Vault; variable based on network demand
2. **Base Fees**: Accumulated in the Base Fee Vault; follows EIP-1559 mechanics
3. **L1 Data Fees**: Charged to cover L1 batch submission costs; includes blob-base-fee adjustments post-Ecotone upgrade

The L1 Data Fee is particularly significant on OP Stack chains. It represents the cost to users for providing data availability to L1, calculated as:

```
L1 Data Fee = (gas cost in wei) × (l1 basefee or blobfee) × (scalar factor)
```

This fee directly reflects Ethereum's L1 security properties, creating economic alignment between the L2 and L1.

### 4.2 Revenue Distribution: The Superchain Model

Unlike Arbitrum, where MEV is primarily captured by a single sequencer, the OP Stack implements a revenue-sharing model across the Superchain:[^7]

- **OP Mainnet**: All sequencer revenue flows to the Optimism Collective, funding governance and ecosystem development
- **Other OP Chains** (e.g., Base): Contribute a percentage of sequencer revenue to the Optimism Collective:
  - Greater of 2.5% of total sequencer revenue (L2 base fees + priority fees), OR
  - 15% of net on-chain profit (total revenue minus L1 gas costs)

As of Q4 2024, the Optimism Collective had accumulated over 15,673 ETH (~$40+ million) from Superchain sequencer revenue, demonstrating the scale of MEV on L2s.

### 4.3 Strategic OP Token Alignment

In January 2026, Optimism governance proposed linking sequencer revenue directly to OP token demand through a buyback mechanism. The proposal allocates 50% of monthly sequencer fees toward OP token market purchases, with acquired tokens held in treasury or subject to future burn/redistribution votes. This approach explicitly ties L2 MEV capture to L1 token value, creating a virtuous cycle: higher L2 activity → more MEV → more OP buybacks → stronger token economics.

---

## 5. Flashbots SUAVE: A Decentralized Alternative

While Arbitrum and Optimism focus on optimizing centralized and DAO-governed sequencer models, Flashbots SUAVE (Single Unified Auction for Value Expression) proposes a radical decentralization of the entire MEV market infrastructure.

### 5.1 SUAVE Architecture

SUAVE is not a rollup but rather a specialized sequencing layer that any blockchain—including L1, L2s, and sidechains—can plug into for decentralized transaction ordering and MEV management.[^8]

**Core Components**:

1. **Universal Preference Environment**: A specialized blockchain that aggregates "preferences"—user-signed messages expressing transaction intent—from all participating chains. Unlike traditional mempools, the preference environment enables rich expression: simple transfers, complex multi-chain swaps, and conditional execution.

2. **Optimal Execution Market**: A competitive network of "executors" who bid to fulfill user preferences. Executors can access encrypted preferences and compete to provide the best execution, capturing any MEV their execution creates but obligated to return a portion to users.

3. **Decentralized Block Building**: A permissionless network of block builders that construct blocks for all participating domains using encrypted preference data. This prevents information asymmetries and front-running.

### 5.2 MEV Redistribution Mechanisms

SUAVE's innovation lies in its redistribution philosophy:

- **User Privacy**: Transaction content is encrypted until ordering is finalized, preventing MEV extraction by sequencers or builders
- **Executor Competition**: Multiple executors bid to fulfill each user preference, competing execution quality forces them to return MEV to users
- **Decentralized Block Building**: Instead of a monolithic builder controlling all MEV, SUAVE enables collaborative block building where multiple parties contribute, reducing concentration risk

The goal is not to eliminate MEV (impossible at the protocol level) but to redistribute it fairly: users who generate MEV benefit from it, rather than sequencers or builders capturing it unilaterally.

### 5.3 Roadmap and Deployment

Flashbots has outlined a three-phase rollout:[^9]

1. **SUAVE Centauri** (Early 2025): Privacy-aware orderflow auction with centralized trust assumption; devnet available for testing
2. **SUAVE Andromeda** (Mid 2025): Mainnet launch; SGX-based encryption replaces centralized trust; open orderflow for builders
3. **SUAVE Helios** (2025+): Decentralized block-building network; multi-domain MEV support; onboarding of second domain

A key aspect of SUAVE's design is gradual decentralization: Flashbots will run centralized infrastructure initially for bootstrapping but intends to transition to full decentralization, positioning itself as a neutral marketplace designer rather than a profit-extracting participant.

---

## 6. Comparative Analysis

### 6.1 Centralization vs. Decentralization Tradeoff

| Aspect | Arbitrum Timeboost | OP Stack | SUAVE |
|--------|-------------------|----------|-------|
| **Sequencer Model** | Centralized + Auction | Centralized + DAO Revenue Share | Decentralized Network |
| **MEV Capture** | Arbitrum DAO | Optimism Collective | Users (via executor competition) |
| **User Protection** | Yes (no reordering) | Partial (fee transparency) | Yes (encrypted preferences) |
| **Scalability** | High (single sequencer) | High (single sequencer) | TBD (network overhead) |
| **Decentralization Path** | Compatible | Roadmap TBD | Native |

### 6.2 Revenue and Economic Sustainability

All three models generate revenue but allocate it differently:

- **Timeboost**: Auction revenue → Arbitrum DAO, sustainable funding for infrastructure
- **OP Stack**: Fee revenue → Optimism Collective, with strategic OP buybacks to align incentives
- **SUAVE**: Executor fees → users (directly) and builders (as competition outcome), sustainable via neutral marketplace design

The ideal model depends on a rollup's governance philosophy: centralized MEV capture with DAO governance (Arbitrum/Optimism) vs. user-aligned MEV redistribution (SUAVE).

### 6.3 Cross-Domain MEV

An emerging concern is **cross-domain MEV**—opportunities that span multiple chains. SUAVE is explicitly designed to handle this through its unified preference environment and decentralized block-building network. Arbitrum and OP Stack, being single-chain sequencers, cannot easily coordinate cross-domain MEV, creating a potential advantage for SUAVE-integrated L2s.[^10]

---

## 7. Emerging Challenges and Future Directions

### 7.1 Shared Sequencers and Interoperability

Beyond SUAVE, other shared sequencer projects are gaining traction:

- **Metis**: Pioneered decentralized PoS sequencers on L2, with validators earning proportional MEV
- **Based Rollups**: A new paradigm where Ethereum L1 proposers sequence rollup transactions, inheriting L1's decentralization but increasing MEV competition
- **Encryption-as-a-Service**: Shutter Network and others use Distributed Key Generation to encrypt transactions until ordering is finalized, preventing MEV pre-confirmation

Each approach makes different tradeoffs between decentralization, user protection, and operational complexity.

### 7.2 Regulatory and Fairness Considerations

As MEV mechanisms mature, regulators may scrutinize whether auction-based sequencing constitutes fair access to blockspace. The SEC and other bodies have shown interest in transaction ordering and front-running prevention, particularly as L2s handle more financial activity.

Timeboost and SUAVE both provide clear answers to fairness objections: they restrict sequencer/builder reordering power and ensure all users have equal access to priority sequencing (via auction). This may make auction-based models more defensible than purely centralized sequencing.

### 7.3 Incentive Alignment in Multi-Chain Futures

As L2 ecosystems expand and users transact across multiple chains, sequencer incentive alignment becomes critical. A sequencer that can extract MEV across domains has a strong economic advantage, which may drive consolidation or the adoption of unified sequencing layers like SUAVE.

Conversely, rollups that fail to monetize MEV efficiently may struggle to fund sequencer operations and security, creating a competitive disadvantage.

---

## 8. Data Sources and Methodological Notes

This brief synthesizes research from:

1. **Protocol Documentation**:
   - Arbitrum official Timeboost documentation[^11]
   - Optimism OP Stack specifications and revenue model papers[^12]
   - Flashbots SUAVE whitepaper and research publications[^13]

2. **On-Chain Data**:
   - Arbitrum Timeboost revenue tracking (Dune Analytics[^14])
   - Optimism Superchain fee distribution (Blockworks, Gate.io)[^15]

3. **Academic and Research Sources**:
   - "Rolling in the Shadows: MEV in Rollups" (Weintraub et al.)
   - Flashbots transparency reports and MEV research collective[^16]
   - Ethereum Research Forum discussions on decentralized sequencing and shared sequencers[^17]

4. **Industry Publications**:
   - The Block, Bankless, and Paradigm research on L2 economics
   - Medium articles from ecosystem researchers (Vitalik.ca, Polynya, others)

---

## 9. Conclusion

MEV on L2 rollups is neither a problem to be eliminated nor an externality to be ignored—it is a structural feature of blockchains that must be actively managed through mechanisms and incentives.

Current L2 leaders—Arbitrum and Optimism—have chosen a pragmatic path: centralized sequencers with transparent, protocol-level auction mechanisms (Timeboost) or DAO-governed revenue sharing (Superchain). These approaches capture MEV value for the protocol and community while maintaining user protection and network efficiency.

Flashbots SUAVE represents an alternative vision: fully decentralized MEV redistribution through competitive executors and encrypted preferences. SUAVE's roadmap toward mainstream adoption suggests the L2 ecosystem is experimenting with multiple MEV architectures, each with distinct security and fairness properties.

The future of MEV on L2s will likely involve:

1. **Continued experimentation**: Multiple models (auction-based, decentralized, shared sequencer) will coexist, each optimized for different use cases
2. **Cross-domain integration**: As users transact across multiple L2s and domains, unified MEV handling (via SUAVE or similar) becomes economically compelling
3. **Regulatory clarity**: Fairness in sequencing and transaction ordering will likely attract regulatory scrutiny, making transparent mechanisms like Timeboost more defensible
4. **Value alignment**: Successful L2s will explicitly tie MEV capture to token value and ecosystem growth, as Optimism's buyback model demonstrates

Ultimately, the MEV mechanisms chosen by L2s will determine their long-term sustainability, user adoption, and competitive position in the multi-chain future.

---

## References

[^1]: Scalingx, "Dealing with MEV in Rollups," Medium (2024). https://medium.com/@scalingx/dealing-with-mev-in-rollups-3ffa2072f154

[^2]: Weintraub, Ben, "Rolling in the Shadows: Side Channels on the Critical Path," academic paper (2024). https://ben-weintraub.com/files/rolling-in-the-shadows.pdf

[^3]: Gate.io, "The Allure of MEV: Why Decentralizing Sequencers is Hard," Learning Center (2024). https://www.gate.com/learn/articles/the-allure-of-mev-why-decentralizing-sequencers-is-hard/1925

[^4]: Arbitrum Foundation, "Timeboost: A Gentle Introduction," Official Documentation (2025). https://docs.arbitrum.io/how-arbitrum-works/timeboost/gentle-introduction

[^5]: Arbitrum Foundation, "Timeboost FAQ," Official Documentation (2025). https://docs.arbitrum.io/how-arbitrum-works/timeboost/timeboost-faq

[^6]: Optimism Foundation, "Transaction Fees on OP Stack," OP Stack Documentation (2024). https://docs.optimism.io/op-stack/transactions/fees

[^7]: Optimism Foundation, "How (and Why) the Superchain Drives Fees to the Optimism Collective," Official Blog (2024). https://www.optimism.io/blog/how-(and-why)-the-superchain-drives-fees-to-the-optimism-collective

[^8]: Flashbots Collective, "The Future of MEV is SUAVE," Writings (2024). https://writings.flashbots.net/the-future-of-mev-is-suave

[^9]: Ibid.

[^10]: Paradigm Research, "Order Flow Auctions and Centralization," Medium (2023). https://writings.flashbots.net/order-flow-auctions-and-centralisation

[^11]: Arbitrum Foundation, Timeboost Official Documentation (2025). https://docs.arbitrum.io/how-arbitrum-works/timeboost/

[^12]: Optimism Foundation, OP Stack Specifications and Protocol Documentation (2024). https://specs.optimism.io/

[^13]: Flashbots Collective, "SUAVE: A Research Collective" (2024). https://github.com/0xapriori/Suave-research

[^14]: Entropy Advisors, "Arbitrum Timeboost Dashboard," Dune Analytics (2025). https://dune.com/entropy_advisors/arbitrum-timeboost

[^15]: Blockworks Research, "Base and Optimism Collective Fee Split," Analytics (2024). https://blockworks.com/analytics/base/base-optimism-collective-fee-split

[^16]: Flashbots Collective, Transparency Reports and MEV Research. https://collective.flashbots.net/

[^17]: Ethereum Research, "Shared Sequencers and Decentralized Sequencing," Forum Discussions (2024). https://ethresear.ch/

---

**Word Count**: 2,847 words  
**Date**: April 2026  
**Submission**: For kcolbchain/research repository

