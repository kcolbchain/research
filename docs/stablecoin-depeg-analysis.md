# Comparative Analysis of Stablecoin Depeg Events (2022–2025)

**Date:** May 2026

## Abstract

Stablecoin depegs between 2022 and 2025 fall into five distinct failure categories: endogenous-collateral death spiral (UST), reserve counterparty run (USDC/SVB), regulatory wind-down (BUSD), reserve-transparency crisis (TUSD), and venue-local oracle dislocation (USDe). This analysis examines each event's trigger mechanism, contagion path, and recovery dynamics, and applies the BIS/NBER run-risk frameworks to extract cross-cutting lessons.

## 1. Event Taxonomy

### UST (May 2022)
- **Type**: Endogenous-collateral death spiral
- **Mechanism**: LUNA mint-burn arbitrage turned reflexive—UST redemptions created new LUNA supply, depressing LUNA price and accelerating the spiral
- **Contagion**: Anchor withdrawals → Curve pool imbalance → cross-exchange selling → credit stress in CeFi lenders
- **Recovery**: None. Design abandoned after ~4 days, ~$60B destroyed
- **Reference**: NBER WP 31160, "Anatomy of a Run: The Terra Luna Crash"

### USDC (March 2023)
- **Type**: Reserve counterparty run
- **Mechanism**: $3.3B of USDC reserves held at SVB after bank failure → holders unsure of depositor protection → secondary-market selling to $0.87
- **Contagion**: DAI (USDC-backed reserves) depegged → DeFi pool imbalances → lending market stress
- **Recovery**: ~7 days, after Fed BTFP announcement + Circle reserve transparency
- **Reference**: NBER WP 31484, "Runs on Crypto Stablecoins"

### BUSD (February 2023)
- **Type**: Regulatory wind-down
- **Mechanism**: NYDFS directed Paxos to stop minting; SEC classified as unregistered security
- **Contagion**: User rotation to USDT, TUSD, FDUSD; exchange pair migration
- **Recovery**: Orderly wind-down; $7.9B redeemed in 32 days without market disruption
- **Reference**: Paxos, "Paxos Will Halt Minting New BUSD Tokens"

### TUSD (January 2024)
- **Type**: Reserve-transparency and liquidity run
- **Mechanism**: Heavy selling on Binance + renewed reserve attestation concerns + reduction in exchange incentives
- **Contagion**: Concentrated on exchange order books; limited broader market impact
- **Recovery**: Partial, after daily reserve attestation response
- **Reference**: Cointelegraph, "TrueUSD stablecoin depegs as holders dump $330M"

### USDe (October 2025)
- **Type**: Venue-local oracle/liquidity dislocation
- **Mechanism**: Binance internal oracle diverged from on-chain liquidity during liquidation cascade → USDe traded to ~$0.65 on Binance while on-chain mint/redeem remained close to peg
- **Contagion**: Concentrated in Binance liquidation systems; ENA governance token sold off
- **Recovery**: ~90 minutes on affected venue; no structural damage
- **Reference**: Ethena Governance, "Ethena's October 2025 Governance Update"

## 2. Comparative Analysis

### Trigger Classification

| Event | Collateral Type | Reserve Risk | Primary Trigger |
|-------|----------------|--------------|-----------------|
| UST | Endogenous (LUNA) | High | Death spiral |
| USDC | Fiat-backed | Medium | Bank counterparty |
| BUSD | Fiat-backed | Low | Regulatory |
| TUSD | Fiat-backed | High | Transparency |
| USDe | Synthetic (delta-neutral) | Medium | Oracle dislocation |

### Recovery Dynamics

| Event | Time to Trough | Recovery Time | Key Recovery Factor |
|-------|---------------|---------------|---------------------|
| UST | ~4 days | Never | — |
| USDC | ~2 days | ~7 days | FDIC/Fed guarantee |
| BUSD | Gradual | Months | Orderly wind-down |
| TUSD | Days | Days | Attestation response |
| USDe | Minutes | ~90 min | Multi-venue normalization |

## 3. BIS/NBER Run-Risk Framework

Three mechanisms explain the observed depeg dynamics:

1. **First-mover advantage**: holders who exit earliest receive closest to par; late-arriving holders absorb losses. Present in all five events, strongest in UST.

2. **Public information shocks**: BIS WP 1164 shows reserve transparency can increase run risk when holders believe asset quality is poor. USDC's SVB disclosure was stabilizing only after regulators confirmed depositor protection.

3. **Concentrated arbitrage**: NBER WP 33882 finds that USDC has ~521 arbitrageurs/month vs ~6 for USDT. Concentrated arbitrage improves peg stability but concentrates run risk.

## 4. Risk Assessment Checklist

For any stablecoin review, assess across seven axes:

1. **Collateral type**: fiat, crypto, endogenous, synthetic
2. **Reserve location**: banks, custodians, MMFs, exchanges, on-chain
3. **Redemption rights**: who, minimum size, fees, timing, enforceability
4. **Arbitrage structure**: direct redeemers, market makers, venue depth
5. **Oracle design**: single venue vs multi-venue weighted index
6. **Issuer/regulatory continuity**: license, supervisor, wind-down plan
7. **Composability exposure**: use as collateral in lending and structured products

## References

- BIS CPMI Paper No. 141, "Will the Real Stablecoin Please Stand Up?" 2023
- BIS WP 1164, "Public Information and Stablecoin Runs." 2024/2025
- NBER WP 31160, "Anatomy of a Run: The Terra Luna Crash." 2023
- NBER WP 33882, "Stablecoin Runs and the Centralization of Arbitrage." 2025
- NBER WP 31484, "Runs on Crypto Stablecoins." 2023
- BIS WP 1270, "Stablecoins and Safe Asset Prices." 2025
- Circle. "USDC Reserve Transparency Report." March 2023
- Paxos. "Paxos Will Halt Minting New BUSD Tokens." 2023
- Ethena Governance. "October 2025 Governance Update." 2025
- ECB WP 2987, "Stablecoins, MMFs and Monetary Policy." 2025
