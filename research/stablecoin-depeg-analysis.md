# Comparative Analysis of Stablecoin Depeg Events (2022-2025)

## Executive Summary

Stablecoin depegs — when a token designed to maintain a $1 peg trades significantly below or above that value — have become one of the most consequential phenomena in digital asset markets. This analysis examines the major depeg events from 2022 to 2025, comparing trigger mechanisms, contagion paths, recovery timelines, and protocol-level mitigations. We apply the Bank for International Settlements (BIS) and National Bureau of Economic Research (NBER) run-risk frameworks to understand these events through the lens of traditional banking theory.

## 1. Introduction: The Stablecoin Landscape

As of early 2026, the stablecoin market exceeds $200 billion in total market capitalization, dominated by:

- **USDT (Tether)**: ~$140B — Fiat-backed, centralized
- **USDC (Circle)**: ~$45B — Fiat-backed, regulated
- **DAI (MakerDAO/Sky)**: ~$8B — Crypto-collateralized, decentralized
- **FRAX (Frax Finance)**: ~$1.5B — Algorithmic, partially collateralized
- **USDe (Ethena)**: ~$5B — Synthetic dollar (delta-neutral)

The failure of any major stablecoin poses systemic risk. This analysis focuses on the three most significant depeg events in recent history.

## 2. Terra/UST Collapse (May 2022)

### 2.1 Overview

The TerraUSD (UST) collapse remains the largest stablecoin failure in history, destroying approximately $60 billion in market capitalization within one week.

| Metric | Value |
|--------|-------|
| Peak market cap | $18.6B (UST) + $40B (LUNA) |
| Time to collapse | ~4 days (May 9-13, 2022) |
| Recovery | Never (UST abandoned) |
| Total losses | ~$60B |

### 2.2 Trigger Mechanism

The UST depeg was triggered by a combination of factors:

1. **Concentrated liquidity withdrawal**: On May 7, 2022, approximately $2 billion in UST was withdrawn from the Curve Finance UST-3pool in a short period, reducing liquidity depth significantly.

2. **Death spiral dynamics**: UST used an algorithmic "mint-burn" mechanism:
   - 1 UST could always be redeemed for $1 worth of LUNA
   - As UST dropped below $1, arbitrageurs minted LUNA (inflating supply) to buy UST
   - Increased LUNA supply → LUNA price dropped → More LUNA needed per UST redemption → Accelerated death spiral

3. **Luna Foundation Guard (LFG) depletion**: The reserve fund ($3B in BTC) was insufficient to defend the peg, and its deployment signaled weakness.

### 2.3 Contagion Path

The contagion spread through multiple channels:

1. **Direct holders**: ~$18B in UST holders faced near-total losses
2. **LUNA holders**: $40B in LUNA market cap destroyed
3. **DeFi protocols**: Anchor Protocol (where ~70% of UST was deposited) became insolvent
4. **CeFi lenders**: Celsius, Voyager, and Three Arrows Capital had significant UST/LUNA exposure
5. **Broader market**: Total crypto market cap fell ~$500B in the following month

### 2.4 Run-Risk Framework (BIS/NBER Analysis)

Applying the Diamond-Dybvig (1983) bank run model:

- **Illiquid assets, liquid liabilities**: UST's "reserves" were endogenous (LUNA), creating a fundamental mismatch
- **First-mover advantage**: Users who redeemed early received more value, incentivizing panic
- **Self-fulfilling prophecy**: Once confidence eroded, the mechanism guaranteed collapse

BIS Working Paper No. 1013 (2022) noted that UST "exhibited all the characteristics of a classical bank run, but without any of the regulatory safeguards."

## 3. USDC Depeg — SVB Crisis (March 2023)

### 3.1 Overview

Unlike UST, the USDC depeg was caused by traditional banking system stress rather than protocol design flaws.

| Metric | Value |
|--------|-------|
| Pre-crisis market cap | $43B |
| Minimum price reached | ~$0.87 (March 11, 2023) |
| Recovery time | ~7 days |
| Final impact | Temporary; full recovery |

### 3.2 Trigger Mechanism

1. **SVB collapse**: On March 10, 2023, Silicon Valley Bank was shut down by California regulators after a bank run
2. **Circle exposure**: Circle (USDC issuer) disclosed $3.3B of its ~$40B reserves were held at SVB
3. **Panic selling**: USDC traded down to $0.87 on some exchanges as users feared insolvency
4. **DAI correlation**: DAI (which held ~50% USDC in its reserves) also depegged to ~$0.90

### 3.3 Contagion Path

1. **Direct**: USDC holders sold at a loss, with $6B in redemptions over 48 hours
2. **DAI/compound stablecoins**: Protocols holding USDC as reserves faced cascading depegs
3. **DeFi liquidations**: USDC-denominated loans became undercollateralized as USDC dropped
4. **DEX pools**: Concentrated liquidity pools experienced significant slippage

### 3.4 Recovery

The recovery was remarkably swift:

- **March 12**: Federal Reserve announces Bank Term Funding Program (BTFP); Circle confirms all USDC reserves are safe
- **March 13**: USDC re-pegs to ~$0.99
- **March 15**: Full recovery to $1.00; Circle publishes detailed reserve breakdown

### 3.5 BIS/NBER Analysis

NBER Working Paper No. 31484 (2023) characterized this event as a "digital-age bank run" where:

- **Information asymmetry**: Users reacted to partial information (knowing SVB exposure existed but not its severity)
- **Coordination failure**: No mechanism existed to reassure holders in real-time
- **Regulatory response as circuit breaker**: The BTFP announcement effectively stopped the run

Key lesson: Even well-collateralized stablecoins face run risk when reserve transparency is insufficient.

## 4. BUSD Shutdown (February-August 2023)

### 4.1 Overview

Paxos-issued BUSD (Binance USD) was forced to wind down following a SEC Wells notice and NYDFS order.

| Metric | Value |
|--------|-------|
| Peak market cap | $23B (Nov 2022) |
| Current status | Winding down |
| Recovery | N/A (orderly wind-down) |

### 4.2 Trigger Mechanism

1. **Regulatory action**: SEC classified BUSD as an unregistered security
2. **NYDFS order**: Paxos ordered to cease minting new BUSD
3. **Binance distancing**: Binance auto-converted BUSD holdings to other stablecoins
4. **Redemption queue**: Large redemptions caused temporary redemption delays

### 4.3 Contagion

- Minimal broader market impact due to orderly wind-down
- Users migrated to TUSD, FDUSD, and USDT
- Demonstrated regulatory risk as a depeg trigger

## 5. Comparative Analysis

### 5.1 Trigger Mechanisms

| Event | Primary Trigger | Secondary Factors | Predictability |
|-------|----------------|-------------------|----------------|
| UST | Algorithmic death spiral | Low reserve, concentrated liquidity | Moderate (design flaw visible) |
| USDC | Banking system failure | Reserve concentration, information gap | Low (black swan) |
| BUSD | Regulatory action | Compliance failures | Moderate (regulatory trend) |

### 5.2 Contagion Path Analysis

| Event | Direct Losses | DeFi Impact | CeFi Impact | Market-Wide |
|-------|--------------|-------------|-------------|-------------|
| UST | $60B | Severe (Anchor, Mirror) | Severe (3AC, Celsius) | Severe |
| USDC | ~$2B (temporary) | Moderate (liquidations) | Minimal | Moderate |
| BUSD | Minimal | Minimal | Minimal | Minimal |

### 5.3 Recovery Timelines

| Event | Time to Trough | Time to Recovery | Recovery Mechanism |
|-------|---------------|------------------|-------------------|
| UST | 4 days | Never | Protocol abandoned |
| USDC | 2 days | 7 days | Fed intervention + reserve transparency |
| BUSD | N/A | N/A | Orderly wind-down (by design) |

### 5.4 Protocol Mitigations Implemented

After these events, several mitigations were adopted:

1. **Reserve diversification**: USDC reduced single-bank concentration; now held across 10+ institutions
2. **Real-time attestation**: Both USDC and USDT now provide near-real-time reserve reports
3. **Liquidity monitoring**: DEX pools now have circuit breakers for extreme depegs
4. **Stablecoin regulation**: EU MiCA (2024) and proposed US stablecoin bills mandate reserve requirements
5. **DeFi risk management**: Aave, Compound added stablecoin-specific parameters (lower LTV, higher liquidation thresholds)

## 6. Run-Risk Framework Application

### 6.1 Diamond-Dybvig Model for Stablecoins

The BIS Working Paper "Regulating Stablecoins: The Search for Safety" (2024) adapts the Diamond-Dybvig model:

```
Normal State:
  Depositors → Stablecoin Issuer → Reserve Assets (bonds, cash)
  
Stress State:
  Depositors panic → Mass redemptions → Issuer must sell assets at discount → Losses → More panic
  
Resolution:
  - Lender of last resort (e.g., Fed for USDC)
  - Redemption throttling
  - Reserve insurance
```

### 6.2 NBER Framework: Three Pillars of Stability

NBER's analysis identifies three stability pillars:

1. **Asset quality**: Reserves must be high-quality, liquid, and free of credit risk
2. **Operational transparency**: Real-time disclosure of reserve composition
3. **Redemption mechanism**: Clear, enforceable redemption rights

UST failed all three. USDC passed #1 and #2 but faced #3 challenges during SVB crisis. BUSD had all three but faced regulatory rather than economic risk.

## 7. Lessons and Predictions

1. **Algorithmic stablecoins remain fragile** — Any mechanism relying on endogenous collateral (its own token) is inherently unstable.

2. **Banking risk is stablecoin risk** — Even fully-backed stablecoins face depeg risk when banking partners fail.

3. **Transparency is the best defense** — Circle's rapid disclosure during the SVB crisis was critical to recovery.

4. **Regulation reduces tail risk** — MiCA-style regulation with mandatory reserves and audits reduces depeg probability.

5. **Contagion is faster in DeFi** — Automated liquidations and DEX swaps propagate depegs instantly compared to traditional bank runs.

6. **Recovery depends on exogenous support** — All successful stablecoin recoveries involved external intervention (regulatory, institutional, or governmental).

## References

1. Diamond, D.W., & Dybvig, P.H. (1983). "Bank Runs, Deposit Insurance, and Liquidity." *Journal of Political Economy*, 91(3), 401-419.
2. BIS Working Paper No. 1013 (2022). "Regulating Stablecoins: The Search for Safety." Bank for International Settlements.
3. NBER Working Paper No. 31484 (2023). "Runs on Crypto Stablecoins." National Bureau of Economic Research.
4. Circle Internet Financial (2023). "USDC Reserve Transparency Report — March 2023." Circle.com.
5. Federal Reserve Board (2023). "Bank Term Funding Program." Federal Reserve Press Release.
6. Liu, J., et al. (2023). "An Empirical Analysis of Depegging Events in Crypto Markets." *Journal of Financial Economics*.
7. Terraform Labs (2022). "Terra Whitepaper." (Archived.)
8. NYDFS (2023). "Order to Paxos Trust Company." New York Department of Financial Services.
9. European Parliament (2024). "Markets in Crypto-Assets Regulation (MiCA)." Official Journal of the EU.
10. MakerDAO (2023). "Post-USDC Depeg Risk Assessment." MakerDAO Forum.

---

*Research brief prepared for kcolbchain research repository. Last updated: April 2026.*
