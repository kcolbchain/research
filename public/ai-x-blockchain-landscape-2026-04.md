# AI x Blockchain Landscape Review — April 2026

> Public version of the internal landscape review. Covers 20+ projects across
> 7 categories. Last updated: 2026-04-14.

## 1. Decentralized Compute for AI

| Project | Chain | Approach | Maturity |
|---------|-------|----------|----------|
| Akash Network | Cosmos | Serverless GPU marketplace | Mainnet |
| Render Network |Solana| GPU rendering + AI inference | Mainnet |
| Golem | Ethereum | P2P compute sharing | Mainnet |
| io.net | Solana | Decentralized GPU cluster | Mainnet |

## 2. AI Agent Frameworks with Crypto

| Project | Features | Token |
|---------|----------|-------|
| Autonolas | On-chain agent services | OLAS |
| Fetch.ai | Autonomous agents, multi-agent systems | FET |
| Bittensor | Subnet-based ML marketplace | TAO |
| Modulus Labs | On-chain ML inference (SNARKs) | — |

## 3. ZK-ML (Zero-Knowledge Machine Learning)

| Project | Approach | Status |
|---------|----------|--------|
| Modulus | ZK-SNARKs for neural net inference | Live |
| EZKL | ZK proofs for PyTorch models | Active dev |
| RISC Zero | ZKVM for ML verification | Active dev |

## 4. Data DAOs & Training Markets

| Project | Model | Token |
|---------|-------|-------|
| Ocean Protocol | Data marketplace | OCEAN |
| Grass | Web scraping DAO | — |
| Vana | User-owned data pools | — |

## 5. AI x DePIN

| Project | DePIN Category | AI Use |
|---------|---------------|--------|
| HiveMapper | Mapping | AI-powered 3D mapping |
| DIMO | Vehicle data | ML on telemetry |
| WeatherXM | Weather stations | Forecast models |

## 6. Token-Based AI Inference

| Project | Mechanism | Token |
|---------|-----------|-------|
| Together AI | Pay-per-token via API | — |
| Modulus | ZK-verified inference | — |
| Pond | Graph ML on-chain | — |

## 7. Agent-to-Agent Payments (our focus)

| Project | Approach | Status |
|---------|----------|--------|
| Switchboard (kcolbchain) | Native-ETH escrow, escrow-oracles | Alpha |
| Coinbase x402 | HTTP 402 payment standard | Draft |
| Lit Protocol | PKP-based agent signing | Mainnet |
| Autonolas | Agent services with multisig | Mainnet |

## Key Takeaways

1. **No native-ETH agent-to-agent escrow exists** besides Switchboard. Most projects use ERC-20 wrappers or off-chain settlement.
2. **ZK-ML is the most active R&D area** — enabling verifiable AI inference on-chain.
3. **Data DAOs are growing** but most lack the quantitative verification that escrow-oracles provide.
4. **DePIN + AI is the fastest-growing intersection**, with 10+ projects launching each quarter.

## References

- [Switchboard Agent Payment Protocol](https://github.com/kcolbchain/switchboard/blob/main/docs/agent-payment-protocol.md)
- [escrow-oracles SPEC.md](https://github.com/kcolbchain/escrow-oracles/blob/main/docs/SPEC.md)
- [EIP-7702: Smart Account Payments](https://eips.ethereum.org/EIPS/eip-7702)
- [coinbase/x402 Standard](https://github.com/coinbase/x402)
