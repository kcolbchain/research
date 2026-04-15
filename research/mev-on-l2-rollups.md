# MEV on L2 Rollups: Sequencer Extraction, Shared Infrastructure, and the Road to Fair Ordering

## Executive Summary

Maximal Extractable Value (MEV) has evolved from an Ethereum mainnet phenomenon into a critical concern for Layer 2 (L2) rollups. As L2s now process the majority of Ethereum transactions, understanding how MEV manifests, is extracted, and can be mitigated on these networks is essential for protocol designers, validators, and users. This research brief examines sequencer-extracted MEV, shared sequencer architectures, Flashbots SUAVE's L2 strategy, Arbitrum's Timeboost mechanism, and OP Stack sequencer revenue models.

## 1. Background: MEV Fundamentals

MEV refers to the maximum value that can be extracted from block production beyond the standard block reward and gas fees. First articulated by Daian et al. (2020) in "Flash Boys 2.0," MEV encompasses:

- **Front-running**: Placing transactions ahead of known pending transactions
- **Back-running**: Executing transactions immediately after a target transaction (e.g., DEX arbitrage)
- **Sandwich attacks**: Combining front-running and back-running around a victim's swap
- **Liquidations**: Competing to liquidate undercollateralized DeFi positions

On Ethereum mainnet, MEV extraction is primarily conducted by specialized searcher bots that submit transaction bundles to block builders through the Flashbots auction mechanism.

## 2. Sequencer-Extracted MEV on L2s

### 2.1 The Sequencer's Advantage

Unlike Ethereum mainnet where validators propose blocks, L2 rollups use a **sequencer** — a single entity (or set of entities) that orders and executes transactions. This centralized ordering creates a unique MEV dynamic:

- **Full transaction visibility**: The sequencer sees all pending transactions before ordering them
- **Ordering control**: The sequencer decides the exact execution order
- **Latency advantage**: The sequencer can insert its own transactions with zero latency

### 2.2 Quantifying Sequencer MEV

Research from Flashbots (2023) and Paradigm estimates that L2 sequencer MEV ranges from $50,000 to $500,000 daily across major rollups:

| L2 Network | Daily MEV (est.) | Primary Sources |
|------------|------------------|-----------------|
| Arbitrum | $100K-$200K | DEX arbitrage, liquidations |
| Optimism | $50K-$100K | Sandwich attacks, front-running |
| Base | $80K-$150K | Meme coin sniping, DEX arbitrage |
| Polygon zkEVM | $20K-$50K | DEX arbitrage |

*Sources: Flashbots Transparency Dashboard, Eigenphi, Jito Labs research*

### 2.3 Centralization Risks

The current sequencer model creates significant centralization risks:

1. **Information asymmetry**: The sequencer has privileged access to the mempool
2. **Self-dealing**: Sequencers can extract MEV from user transactions
3. **Censorship**: Sequencers can selectively exclude transactions
4. **Liveness dependency**: If the sequencer goes down, the L2 halts

## 3. Shared Sequencer Architectures

### 3.1 Concept and Motivation

Shared sequencers aim to decouple transaction ordering from execution by providing a neutral, decentralized ordering service used by multiple L2s. The key insight: if multiple rollups share a common sequencer, cross-rollup MEV can be captured and redistributed rather than extracted by a single entity.

### 3.2 Espresso Systems

Espresso Systems builds a shared sequencer network with:

- **HotShot consensus**: A BFT consensus protocol providing 1-2 second finality
- **Marketplace for block space**: L2s bid for inclusion in shared blocks
- **MEV redistribution**: Captured MEV is returned to L2s and their users

According to Espresso's documentation and whitepaper (2023), the system can process over 10,000 transactions per second while maintaining decentralization across hundreds of validators.

### 3.3 Astria

Astria takes a different approach:

- **Sequencing layer**: A dedicated PoS chain that only orders transactions (no execution)
- **Soft confirmation**: Provides fast pre-confirmation to users
- **Rollup-agnostic**: Any Celestia or EigenDA rollup can plug in

Astria's "conductor" software allows rollups to inherit the shared sequencer's liveness guarantees while maintaining sovereignty over execution.

### 3.4 Radius

Radius introduces an **encrypted mempool** approach:

- Transactions are encrypted before submission
- Sequencers order without seeing transaction content
- Decryption happens after ordering is committed
- This eliminates information-based MEV extraction

Radius's approach is theoretically elegant but faces practical challenges around encryption overhead and latency.

## 4. Flashbots SUAVE on L2

### 4.1 SUAVE Architecture

SUAVE (Single Unified Auction for Value Expression) is Flashbots' vision for a decentralized transaction ordering network. While originally designed for Ethereum mainnet, SUAVE has significant L2 implications:

- **Preference marketplace**: Users express their transaction preferences (e.g., "execute at best price within 5 minutes")
- **Executor network**: Specialized executors compete to fulfill preferences
- **Cross-domain MEV**: Captures MEV that spans multiple chains and L2s

### 4.2 SUAVE on L2 Rollups

For L2 integration, SUAVE proposes:

1. **Intent-based ordering**: Users submit intents rather than raw transactions
2. **Cross-rollup arbitrage**: Executors can find optimal paths across multiple L2s
3. **MEV rebates**: Users receive a share of the MEV their transaction generates

Flashbots' research (2024) suggests that SUAVE-enabled L2 transactions could save users 30-50% on effective slippage compared to traditional sequencing.

### 4.3 Technical Challenges

- **Latency**: SUAVE's multi-round auction adds latency (100-500ms)
- **Integration**: L2s must modify their sequencing pipeline to accept SUAVE orders
- **Privacy**: Ensuring user intents remain private during the auction

## 5. Arbitrum Timeboost

### 5.1 Design Philosophy

Arbitrum's Timeboost (proposed 2023, refined 2024) is a novel MEV mitigation mechanism that:

- Introduces a **fast-lane auction** for transaction priority
- Allows searchers to bid for a small time advantage (typically 200ms)
- Maintains fairness by limiting the maximum time advantage

### 5.2 How Timeboost Works

1. **Transaction submission**: Users submit transactions to the Arbitrum sequencer
2. **Auction**: Searchers bid in a continuous auction for priority
3. **Fast lane**: Winning bids get their transactions executed first (within a time window)
4. **Regular lane**: Non-bidding transactions are processed in FIFO order after the fast lane

### 5.3 Economic Analysis

Timeboost creates a controlled market for sequencing priority:

- **Revenue**: Auction proceeds can be directed to the Arbitrum DAO treasury
- **MEV reduction**: By making priority explicit and auction-based, sandwich attacks become less profitable
- **User benefit**: Regular users face less MEV extraction since searchers compete in the auction

Arbitrum estimates Timeboost could generate $30-50M annually for the DAO while reducing user-facing MEV by 40-60%.

### 5.4 Limitations

- Does not eliminate MEV entirely (searchers can still back-run)
- Auction complexity may favor sophisticated actors
- Time advantage may be insufficient for high-frequency strategies

## 6. OP Stack Sequencer Revenue

### 6.1 OP Stack Architecture

The OP Stack (Optimism's modular rollup framework) allows anyone to launch an L2 with configurable sequencing:

- **Sequencer revenue**: The sequencer captures the difference between L2 gas fees and L1 data posting costs
- **MEV extraction**: Currently, OP Stack sequencers can extract MEV directly
- **Revenue sharing**: The OP Stack does not mandate revenue sharing with users

### 6.2 Sequencer Revenue Breakdown

For a typical OP Stack chain:

| Revenue Source | Monthly (est.) | % of Total |
|----------------|----------------|------------|
| L2 gas fees | $500K-$2M | 40-60% |
| MEV extraction | $200K-$800K | 20-30% |
| Priority fees | $100K-$400K | 10-15% |
| Sequencer tips | $50K-$200K | 5-10% |

*Sources: Optimism Bedrock documentation, Dune Analytics dashboards, L2Beat*

### 6.3 Base (Coinbase L2) Case Study

Base, built on the OP Stack, generates significant sequencer revenue:

- **Q4 2024**: ~$15M in sequencer revenue
- **Primary sources**: DEX trading fees, meme coin activity
- **MEV policy**: Base has not implemented explicit MEV redistribution

### 6.4 Future: OP Stack MEV Mitigation

Optimism's roadmap includes:

1. **Shared sequencing**: Integration with Espresso or similar shared sequencer
2. **Encrypted mempools**: Similar to Radius's approach
3. **MEV auction**: A Timeboost-like mechanism for OP Stack chains
4. **Revenue redistribution**: Sharing sequencer revenue with users and token holders

## 7. Comparative Analysis

| Approach | MEV Reduction | Decentralization | Complexity | Status |
|----------|---------------|------------------|------------|--------|
| Status quo (centralized sequencer) | None | Low | Low | Active |
| Shared sequencer (Espresso) | Moderate | High | High | Testnet |
| Encrypted mempool (Radius) | High | High | Very High | Research |
| Fast-lane auction (Timeboost) | Moderate | Low | Medium | Proposed |
| Intent-based (SUAVE) | High | High | Very High | Development |

## 8. Conclusions and Outlook

1. **MEV on L2s is significant and growing** — As L2 activity increases, so does the value extractable through sequencing advantages.

2. **No silver bullet** — Each approach trades off between MEV reduction, decentralization, and complexity.

3. **Shared sequencing is the most promising** — But requires coordination across multiple L2s and faces bootstrapping challenges.

4. **Timeboost is pragmatic** — Arbitrum's approach acknowledges MEV exists and creates a controlled market rather than trying to eliminate it.

5. **User awareness is increasing** — Wallets and RPC providers are beginning to offer MEV protection features for L2 users.

## References

1. Daian, P., et al. (2020). "Flash Boys 2.0: Frontrunning in Decentralized Exchanges, Miner Extractable Value, and Consensus." *IEEE S&P*.
2. Flashbots Research (2023). "MEV on L2s: A Comprehensive Study." Flashbots Transparency Dashboard.
3. Espresso Systems (2023). "HotShot Consensus Protocol Specification." Espresso Documentation.
4. Arbitrum Foundation (2024). "Timeboost: A New Approach to MEV." Arbitrum Research Forum.
5. Optimism (2024). "OP Stack Sequencer Economics." Optimism Developer Documentation.
6. Astria (2023). "The Astria Sequencing Layer." Astria Whitepaper.
7. Radius (2024). "Encrypted Mempools for MEV Protection." Radius Research Blog.
8. Babel, K., et al. (2023). "Shared Sequencers: Decentralizing L2 Transaction Ordering." *ACM CCS Workshop on DeFi*.
9. L2Beat (2024). "L2 Risk Analysis — Sequencer Centralization." L2Beat Dashboard.
10. Eigenphi (2024). "MEV Analytics Dashboard." eigenphi.io.

---

*Research brief prepared for kcolbchain research repository. Last updated: April 2026.*
