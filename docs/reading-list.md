# Annotated Reading List for Blockchain Newcomers

A curated list of ~20 resources covering blockchain fundamentals, smart contracts, DeFi, L2 scaling, and MEV. Each entry includes why it matters for a newcomer.

## Blockchain Fundamentals

### 1. Bitcoin Whitepaper
Nakamoto, S. "Bitcoin: A Peer-to-Peer Electronic Cash System." 2008.
https://bitcoin.org/bitcoin.pdf
The original. Explains the double-spend problem, proof-of-work, and the chain-of-hash consensus model. Essential context for everything that followed.

### 2. Ethereum Whitepaper
Buterin, V. "Ethereum: A Next-Generation Smart Contract and Decentralized Application Platform." 2014.
https://ethereum.org/en/whitepaper/
Introduces the world computer concept: a blockchain that runs arbitrary code (smart contracts), enabling dApps, DAOs, and programmable money.

### 3. "The Byzantine Generals Problem"
Lamport, L., Shostak, R., Pease, M. ACM Transactions on Programming Languages and Systems, 1982.
https://lamport.azurewebsites.net/pubs/byz.pdf
The foundational paper on fault-tolerant distributed consensus. Understanding Byzantine fault tolerance is essential to understanding why blockchains work.

### 4. "SoK: Research Perspectives and Challenges for Bitcoin and Cryptocurrencies"
Bonneau, J., et al. IEEE S&P, 2015.
https://arxiv.org/abs/1504.06116
A systematization of knowledge paper that maps the early cryptocurrency research landscape. Good for understanding the design-space tradeoffs.

### 5. "Blockchain Technology Overview"
Yaga, D., et al. NIST Interagency Report 8202, 2018.
https://nvlpubs.nist.gov/nistpubs/ir/2018/NIST.IR.8202.pdf
Government-published primer that explains blockchain, distributed ledger, smart contracts, and applications in neutral, accessible language.

## Smart Contracts

### 6. Solidity Documentation
Ethereum Foundation. "Solidity Documentation."
https://docs.soliditylang.org/
The official language reference. Start with the "Solidity by Example" section for hands-on learning.

### 7. "Making Smart Contracts Smarter"
Luu, L., et al. ACM CCS, 2016.
https://dl.acm.org/doi/10.1145/2976749.2978309
First systematic analysis of smart contract vulnerabilities. Introduces the classification of security bugs that later became the standard audit checklist.

### 8. OpenZeppelin Contracts Documentation
OpenZeppelin. "Contracts Documentation."
https://docs.openzeppelin.com/contracts/
Industry-standard library for audited, tested contract primitives (ERC-20, ERC-721, access control, pausable). Newcomers should study the patterns before writing custom code.

## DeFi

### 9. "Automated Market Making: Theory and Practice"
Angeris, G., Chitra, T. arXiv, 2020.
https://arxiv.org/abs/2006.06058
Rigorous mathematical treatment of AMMs. Explains constant-product markets, concentrated liquidity, and the economics of arbitrage-driven price alignment.

### 10. Uniswap V3 Whitepaper
Adams, H., Zinsmeister, N., Moody, T., Robinson, D. "Uniswap V3 Core." 2021.
https://uniswap.org/whitepaper-v3.pdf
Explains concentrated liquidity, multiple fee tiers, and the oracle design. The dominant AMM model; understanding it is required for serious DeFi work.

### 11. "A Theory of Lending Protocols"
arXiv 2506.15295, 2025.
https://arxiv.org/pdf/2506.15295
Formal state-machine model of lending protocols. Covers overcollateralization, liquidation, and interest-rate mechanics. Bridges academic rigor with protocol design.

### 12. MakerDAO Documentation
Sky (formerly MakerDAO). "Sky Protocol Documentation."
https://docs.sky.money/
Case study in decentralized stablecoin design. Explains the DAI (now USDS) system: collateral vaults, liquidation, stability fees, and governance.

## Layer 2 Scaling

### 13. "SoK: Layer-Two Blockchain Protocols"
Jourenko, M., et al. Financial Cryptography, 2020.
https://fc20.ifca.ai/preproceedings/134.pdf
Comprehensive survey of L2 approaches: state channels, plasma, rollups, sidechains. Maps the design space and tradeoffs between them.

### 14. "An Incomplete Guide to Rollups"
Ethereum Foundation. "Rollup Documentation."
https://ethereum.org/en/developers/docs/scaling/rollups/
Official Ethereum.org primer on optimistic and ZK rollups. Explains how rollups post data to L1, verify state, and enable cheap transactions.

### 15. Arbitrum BoLD Documentation
Offchain Labs. "Arbitrum BoLD: Bounded Liquidity Delay."
https://docs.arbitrum.io/
First permissionless fraud proof system on a major rollup. Documents the dispute-resolution protocol that enables trustless L2 security.

### 16. "Based Rollups"
Drake, J. Ethereum Research Forum, 2024.
https://ethresear.ch/t/based-rollups/16740
Explains the concept of L1-sequenced rollups where Ethereum validators order L2 transactions. Introduces synchronous composability across rollups.

## MEV

### 17. "Flash Boys 2.0: Frontrunning in Decentralized Exchanges, Miner Extractable Value, and Consensus"
Daian, P., et al. IEEE S&P, 2020.
https://arxiv.org/abs/1904.05234
The paper that defined MEV. Explains how miners/validators extract value through front-running, back-running, and sandwich attacks on DEX transactions.

### 18. "MEV on Ethereum: A Policy Analysis"
Flashbots Research, 2023.
https://writings.flashbots.net/mev-on-ethereum-a-policy-analysis
Accessible overview of MEV as a policy problem. Covers PBS, MEV-Boost, order-flow auctions, and the tradeoffs between centralization and value capture.

### 19. "MEV and the Limits of Scaling"
Flashbots Research, 2025.
https://writings.flashbots.net/mev-and-the-limits-of-scaling
Shows that MEV scales super-linearly with blockchain throughput. Essential for understanding why MEV does not disappear as L2s grow.

## Blockchain Economics

### 20. "Tokenomics: The Crypto Economy of Blockchain Tokens"
Freni, P., et al. Journal of Economic Surveys, 2022.
https://doi.org/10.1111/joes.12529
Academic survey of token economics. Covers utility tokens, governance tokens, stablecoins, and the economic models that sustain blockchain networks.

### 21. "Will the Real Stablecoin Please Stand Up?"
BIS CPMI Paper No. 141, 2023.
https://www.bis.org/publ/bppdf/bispap141.htm
Central-bank analysis of 68 stablecoins. Finds continuous parity is the exception, not the rule. Essential for understanding stablecoin risk.

## How to Use This List

1. Start with items 1-5 (blockchain fundamentals)
2. Move to 6-8 (smart contracts)
3. Then 9-12 (DeFi)
4. Then 13-16 (L2 scaling)
5. Then 17-19 (MEV)
6. Finally 20-21 (economics)

Follow links to cited references within each paper to go deeper.
