# kcolbchain Internal Research Brief — Q2 2026

**Date:** April 2, 2026
**Scope:** L2 scaling, payments, stablecoins, DeFi primitives, RWAs
**Status:** INTERNAL — for research planning and positioning

---

## Executive Summary

The blockchain infrastructure landscape is consolidating around several clear themes. This brief identifies the highest-signal research, papers, and developments across our core areas, and highlights where kcolbchain can publish original work.

**Top-level findings:**

1. **Native rollups could make proof systems optional.** EF demo'd L1 directly verifying L2 state — no ZK/fraud proofs needed. Early but massive implications.
2. **ZK proving costs collapsed 100x.** StarkNet S-Two, decentralized prover networks (Succinct, Brevis), and proof aggregation making ZK rollups economically competitive with optimistic.
3. **Stablecoins now move Treasury markets.** BIS paper shows stablecoin inflows reduce T-bill yields by 2.5-3.5 bps. $307B total market cap.
4. **NY Fed endorsed permissionless payment infra over FedNow.** Watershed institutional shift.
5. **Parallel EVM is production-ready.** Monad (300 MGas/s), MegaETH (1,700 MGas/s) — 10-100x over legacy EVM.
6. **Regulation is settled.** U.S. GENIUS Act signed July 2025. EU MiCA fully enforced Jan 2025. The question now is second-order effects.

---

## 1. L2 Scaling & Rollups

### Frontier Research

| Paper / Development | Team | Date | Key Insight |
|---|---|---|---|
| **Native Rollups PoC** | EF + Ethrex + L2BEAT | Mar 2026 | L1 verifies L2 state directly via EXECUTE precompile (EIP-8079). No proof systems needed. |
| **Based Rollups + Synchronous Composability** | Justin Drake / EF | 2025-26 | L1 validators sequence rollups → cross-rollup atomic transactions become native |
| **Based Preconfirmations** | Taiko, Puffer UniFi AVS | 2025 | Sub-10ms preconfirmation guarantees for based rollups |
| **Dynamic Fraud Proofs** | Picco, Fortugno (arXiv) | Feb 2025 | Sub-second finality under ideal conditions via dynamic challenge period adjustment |
| **Economic Censorship in Fraud Proofs** | ACM EC 2025 | Feb 2025 | Formal game theory for censorship attacks. Defenders have inherent economic advantage. |
| **Arbitrum BoLD** | Offchain Labs | Feb 2025 | First permissionless fraud proofs on major rollup. 6.4-day bounded dispute resolution. |
| **Arbitrum Stylus** | Offchain Labs | 2025 | WASM execution (Rust/C/C++) alongside EVM. 10-100x faster for crypto primitives. |

### ZK Rollup Advances

| Development | Team | Key Metric |
|---|---|---|
| **StarkNet S-Two Prover** | StarkWare | 100x proving efficiency improvement, 4s block times |
| **zkSync Atlas** | Matter Labs | Elastic chain — network of interconnected ZK rollups |
| **Scroll decentralized proving** | Scroll | Targeting 10K TPS, <30s proving times |
| **PlasmaFold** | IACR ePrint | Client-side ZK proving for privacy-preserving transfers |
| **Decentralized Prover Networks** | Succinct, Brevis | Commoditized ZK proving via open markets |

### Data Availability

| Development | Key Metric |
|---|---|
| **Ethereum Fusaka / PeerDAS** (Dec 2025) | 85% reduction in validator blob download. L2 costs down 50-90%. |
| **BPO1/BPO2 forks** (Dec 2025 / Jan 2026) | Blob target: 6 → 10 → 14. Max: 9 → 15 → 21. |
| **Celestia Matcha** (Q1 2026) | 56+ rollups, 160+ GB processed. Fibre targets 1 Tbit/s. |
| **EigenDA** | 100 MB/s throughput (DAC model, higher trust assumptions) |
| **Avail** | Universal chain-agnostic DA. Integrations with Arbitrum, Optimism, Polygon. |

### Parallel EVM

| Chain | Type | Throughput |
|---|---|---|
| **Monad** | L1 | 300 MGas/s (mainnet Nov 2025) |
| **MegaETH** | L2 | 1,700 MGas/s (testnet) |

### Open Research Questions (where kcolbchain can contribute)

1. **Native rollups vs proof systems** — what's the design space when L1 can verify L2 directly?
2. **Cross-rollup MEV under shared sequencing** — no formal solutions exist yet
3. **DA economics** — will Ethereum's blob expansion kill external DA layers?
4. **Parallel EVM correctness** — formal verification of optimistic parallel execution conflict detection

---

## 2. Payments & Stablecoins

### Key Papers

| Paper | Authors / Institution | Date | Key Finding |
|---|---|---|---|
| **Stablecoin Runs and Centralization of Arbitrage** | Ma, Zeng, Zhang (NBER W33882) | 2025 | USDT: ~6 arbitrageurs/month. USDC: ~521. Concentrated arbitrage = better peg, worse run risk. |
| **Stablecoins and Safe Asset Prices** | Aldasoro, Ahmed (BIS WP 1270) | 2025 | Stablecoin inflows reduce T-bill yields by 2.5-3.5 bps. Stablecoins are macro-relevant. |
| **Public Information and Stablecoin Runs** | Ahmed, Aldasoro, Duley (BIS WP 1164) | 2025 | Reserve transparency can paradoxically *increase* run risk when holders believe quality is low. |
| **Stablecoins: A Revolutionary Payment Technology** | Ahmed et al. (NBER W34475) | Oct 2025 | First major analysis under GENIUS Act framework. Uses LLM-based podcast analysis. |
| **Stablecoins, MMFs and Monetary Policy** | ECB WP 2987 | 2025 | Stablecoin market cap drops ~10pp within 3 months of U.S. monetary policy shock. |
| **Stablecoins and Monetary Policy Transmission** | ECB WP 3199 | 2025 | Stablecoin growth could weaken ECB's ability to steer short-term rates. |
| **Learning from Terra-Luna** | ScienceDirect | 2025 | Hybrid algorithmic models reduce collapse events by 60-96%. |

### Institutional Developments

| Development | Key Detail |
|---|---|
| **NY Fed: "Payment Infrastructure Could Be Permissionless"** | Fed explicitly argues for permissionless over FedNow. Stablecoin transfers = $27.6T in 2024 (> Visa + Mastercard combined). |
| **IMF: "Understanding Stablecoins"** (Dec 2025) | Stablecoin holdings in Africa: near-zero (2020) → 1.5% of total deposits (2024). |
| **IMF: CBDC cross-border cost savings** | CBDCs could reduce cross-border costs ~60%, saving ~$17B on remittances alone. |
| **BIS Project Agora** | 7 central banks + 41 private firms. Tokenized deposits + wholesale CBDC. Report H1 2026. |
| **BIS Project mBridge** | 4,000+ cross-border txns, $55.5B settled. China's e-CNY = 95% of volume. |
| **Circle Arc chain** | Purpose-built stablecoin chain. Sub-second finality, StableFX, opt-in privacy. 2026 mainnet. |

### Regulation (settled)

| Framework | Status |
|---|---|
| **U.S. GENIUS Act** | Signed July 18, 2025. 1:1 reserves, OCC oversight, BSA/AML. Final regs by July 2026. |
| **EU MiCA** | Full enforcement Jan 2025. Only USDC/EURC compliant of top 10. USDT access uncertain. |
| **Hong Kong** | HKMA licensing effective Aug 1, 2025. |

### Market Snapshot (early 2026)

| Metric | Value |
|---|---|
| Total stablecoin market cap | ~$307B |
| USDT market share | ~60.8% |
| USDC market cap | ~$75.2B |
| 2025 transaction volume | ~$33T (72% YoY increase) |
| L2 TVL | ~$47B (stablecoins = 70%+ of flows) |
| Circle IPO (June 2025) | $31/share, +168% day 1, ~$18B valuation |

### Open Research Questions

1. **Stablecoin run dynamics under GENIUS Act** — how does regulation change the game theory?
2. **USDT vs USDC divergence under MiCA** — market structure implications
3. **L2 rollups as payment rails** — formal analysis of stablecoin payment economics on rollups
4. **CBDC vs stablecoin competition** — which wins for cross-border payments?

---

## 3. DeFi Primitives & Mechanism Design

### AMM Innovations

| Paper / Development | Team | Date | Key Insight |
|---|---|---|---|
| **Orbital AMM** | Dave White, Dan Robinson (Paradigm) | Jun 2025 | Spherical-shell concentrated liquidity for multi-asset stablecoin pools. Handles depeg gracefully. |
| **Mechanism Design for AMMs** | Chan, Wu, Shi (AFT 2025) | 2025 | Batch-processing AMM that eliminates within-block MEV. Proves "arbitrage resilience". |
| **Uniswap V4 Hooks** | Uniswap Labs | Jan 2025 | Custom logic before/after swaps. 2,500+ hook-enabled pools. AMM as platform, not monolith. |
| **BondMM-A** | arXiv 2512.16080 | Dec 2025 | Fixed-income AMM for arbitrary maturities in single pool. Toward on-chain yield curve. |
| **A Theory of Lending Protocols** | arXiv 2506.15295 | Jun 2025 | Formal state-machine model of lending. Framework for provably safe protocol design. |

### Lending & Yield

| Development | Key Detail |
|---|---|
| **Aave V4** (Mar 2026) | Hub-and-spoke: shared liquidity + risk-specific markets. RWA collateral via Horizon ($2.8B+). ~29% of all DeFi TVL. |
| **Pendle** | $9B TVL. Boros (funding rate tokenization) hit $6.9B OI. 50-60% of yield tokenization market. |
| **EigenLayer** | $19.7B TVL, 4.6M+ ETH. Pivoted to "Verifiable Cloud". EigenCloud (AI-focused) attracted $170M. |
| **Morpho Blue** | Minimal immutable lending (~650 lines). Permissionless market creation. |

### Intent-Based Trading

CoW Protocol, UniswapX, 1inch Fusion, Across Protocol — users specify outcomes, solvers compete to fill. Up to 20% of volume from solver-internal matches (no AMM needed). Structurally reduces MEV.

### MEV Research

| Development | Team | Key Insight |
|---|---|---|
| **BuilderNet v1.2** | Flashbots + Beaverbuild + Nethermind | Decentralized block building on TEEs. Flashbots ceased all centralized builders Dec 2024. |
| **Blind Arbitrage with FHE** | Flashbots | Searchers backrun encrypted txns using FHE. Prototype works, not yet practical. |
| **MEV and the Limits of Scaling** | Flashbots | MEV scales super-linearly with throughput. Scaling alone doesn't solve MEV. |
| **Order Flow Auctions / MEV-Share** | Flashbots | Programmable-privacy auctions. Users capture portion of MEV their txns create. |
| **Encrypted Mempools** | Shutter Network | Protocol-level front-running prevention. Response to a16z limitations analysis. |

---

## 4. RWA Tokenization

### Market Snapshot

| Metric | Start 2025 | Q1 2026 |
|---|---|---|
| Total tokenized RWAs on-chain | ~$5B | **$33B+** |
| Tokenized US Treasuries | ~$1.7B | **$7.3-10B** |
| On-chain private credit | $1.14B | **$3.2B+** |

### Key Players

| Entity | Product | AUM / Status |
|---|---|---|
| **BlackRock BUIDL** | Tokenized US Treasury fund | $2.8B AUM. Used as collateral in DeFi lending. 4-stage tokenization roadmap. |
| **Franklin Templeton BENJI** | Government money-market fund on 7 chains | $800M+. Tokenized ETF partnership with Ondo (Mar 2026). |
| **Ondo Finance** | OUSG (treasuries), USDY (yield stablecoin) | $1.4B TVL. Leading DeFi-native RWA issuer. |
| **Centrifuge** | On-chain private credit | $1.1B+ active loans, 8-12% yields. Whitelabel platform for institutions (Nov 2025). |

### Projections
- BCG: tokenized RWAs at **$16T by 2030**
- Citi: **$4-5T** in tokenized securities by 2030
- Centrifuge: RWA TVL exceeding **$100B by end 2026**

---

## 5. New Financial Primitives

### Prediction Markets
- **Polymarket**: $8B+ monthly volume (Mar 2026). CFTC-approved. Acquired QCEX ($112M). Official X/Twitter partner.
- **pm-AMM** (Paradigm): Specialized AMM for prediction markets where tokens converge to 0 or 1.

### On-Chain Perpetuals
- **Hyperliquid**: 70-80% market share in perp DEXs. $350B+ monthly volume. >$100M/month revenue.
- **dYdX**: $2.18B TVL on own Cosmos chain. 220+ markets, 50x leverage.
- Total perp DEX volume: **$7.9T in 2025** (65% of all-time lifetime volume).

### Protocol-Native Stablecoins
- **GHO/sGHO** (Aave): >54% of GHO staked. Yield funded by protocol revenue.
- **USDS** (Sky/Maker): DAI→USDS transition. Spark Protocol $3B+ TVL. Dynamic DSR 0-8.75%.

---

## 4. Research Positioning — Where kcolbchain Should Publish

Based on this brief, the highest-impact research positions for kcolbchain:

### Tier 1: Immediate (publish in next 30 days)

1. **"Native Rollups: What Happens When L1 Can Verify L2 Directly?"**
   - Analyse the EF PoC, design space implications, what becomes obsolete
   - This is brand-new (March 2026), nobody has published a comprehensive analysis yet

2. **"Stablecoins Are Macro Now: What the BIS and NY Fed Papers Mean"**
   - Synthesize BIS WP 1270, NY Fed Liberty Street, GENIUS Act into one piece
   - Position kcolbchain at the intersection of crypto research and macro policy

3. **"The DA War: Will Ethereum's Blob Expansion Kill Celestia?"**
   - Compare post-Fusaka economics (PeerDAS + BPO1/BPO2) vs Celestia Matcha vs EigenDA
   - Data-driven cost analysis for rollup teams

### Tier 2: Medium-term (next 60 days)

4. **"Cross-Rollup MEV Under Shared Sequencing"** — open problem, formal treatment needed
5. **"ZK Proving Costs: The 100x Collapse"** — benchmark S-Two, Succinct, Brevis
6. **"Parallel EVM Correctness"** — formal analysis of Monad/MegaETH conflict detection
7. **"Orbital and the New AMM Era"** — analyse Paradigm's Orbital, V4 hooks, batch-processing AMMs
8. **"RWAs at $33B: What's Actually Working"** — data-driven analysis of BlackRock BUIDL, Ondo, Centrifuge

### Tier 3: Ongoing series

9. **kcolbchain weekly research roundup** — newsletter format, curated from this brief
10. **Tool reviews** — deep-dive open-source tool analysis (Foundry, Slither, SP1, etc.)
11. **Protocol teardowns** — how specific protocols work under the hood

---

## Sources

### L2 & Scaling
- [Native Rollups PoC — The Block](https://www.theblock.co/post/393160/)
- [Based Rollups — Bankless](https://www.bankless.com/based-rollups-justin-drake)
- [Arbitrum BoLD — The Block](https://www.theblock.co/post/340278/)
- [StarkNet 2025 Year in Review](https://www.starknet.io/blog/starknet-2025-year-in-review/)
- [Fusaka PeerDAS — EF Blog](https://blog.ethereum.org/2025/11/06/fusaka-mainnet-announcement)
- [Dynamic Fraud Proofs — arXiv:2502.10321](https://arxiv.org/abs/2502.10321)
- [Economic Censorship — arXiv:2502.20334](https://arxiv.org/abs/2502.20334)
- [Celestia Economics — BlockEden](https://blockeden.xyz/blog/2026/01/16/celestia-blob-economics/)
- [Succinct Prover Network — Yahoo Finance](https://finance.yahoo.com/news/succinct-first-decentralized-prover-network-200500829.html)

### Payments & Stablecoins
- [NBER W33882 — Stablecoin Runs](https://www.nber.org/papers/w33882)
- [NBER W34475 — Stablecoins as Payment Technology](https://www.nber.org/papers/w34475)
- [BIS WP 1270 — Stablecoins and Safe Asset Prices](https://www.bis.org/publ/work1270.htm)
- [BIS WP 1164 — Public Information and Stablecoin Runs](https://www.bis.org/publ/work1164.htm)
- [NY Fed — Permissionless Payment Infrastructure](https://libertystreeteconomics.newyorkfed.org/2025/11/the-future-of-payment-infrastructure-could-be-permissionless/)
- [IMF — Understanding Stablecoins](https://www.imf.org/en/publications/departmental-papers/issues/2025/12/02/understanding-stablecoins-570602)
- [ECB WP 2987](https://www.ecb.europa.eu/pub/pdf/scpwps/ecb.wp2987~1919e51abf.en.pdf)
- [ECB WP 3199](https://www.ecb.europa.eu/pub/pdf/scpwps/ecb.wp3199~ad552b59ec.en.pdf)
- [GENIUS Act — White House](https://www.whitehouse.gov/fact-sheets/2025/07/fact-sheet-president-donald-j-trump-signs-genius-act-into-law/)

### DeFi & RWA
- [Paradigm — Orbital AMM](https://www.paradigm.xyz/2025/06/orbital)
- [Paradigm — TradFi Tomorrow](https://www.paradigm.xyz/2025/03/tradfi-tomorrow-defi-and-the-rise-of-extensible-finance)
- [arXiv — Mechanism Design for AMMs](https://arxiv.org/abs/2402.09357)
- [arXiv — A Theory of Lending Protocols](https://arxiv.org/pdf/2506.15295)
- [arXiv — BondMM-A Fixed-Income AMM](https://arxiv.org/html/2512.16080v1)
- [Flashbots — BuilderNet](https://www.flashbots.net/)
- [Flashbots — Blind Arbitrage with FHE](https://writings.flashbots.net/blind-arbitrage-fhe)
- [Flashbots — MEV and Limits of Scaling](https://writings.flashbots.net/mev-and-the-limits-of-scaling)
- [BlackRock Tokenization 4-Stage Plan](https://www.binaryx.com/blog/blackrocks-tokenization-vision-explained)
- [Franklin Templeton + Ondo Tokenized ETFs](https://crypto.news/franklin-templeton-and-ondo-launch-tokenized-etfs/)
- [Centrifuge 2026 Predictions](https://centrifuge.io/blog/2026-real-world-asset-tokenization)
- [Aave 2025 Recap](https://aave.com/blog/aave-2025-recap)
- [Pendle — $69.8B in Yield Settled](https://www.dlnews.com/external/pendle-settles-698-billion-in-yield/)

---

*This brief is a living document. Last updated: April 2, 2026.*
