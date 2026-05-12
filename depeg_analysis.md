# Comparative Analysis of Stablecoin Depeg Events (2022-2025)

## Executive Summary

Stablecoin depegs between 2022 and 2025 show that "peg risk" is not a single failure mode. The major events fall into five categories:

1. **Endogenous-collateral death spiral**: TerraUSD (UST) in May 2022.
2. **Reserve counterparty run**: USDC during the Silicon Valley Bank crisis in March 2023.
3. **Regulatory wind-down risk**: BUSD after the NYDFS/Paxos action in February 2023.
4. **Reserve-transparency and liquidity run**: TrueUSD (TUSD) in January 2024.
5. **Venue-local liquidity/oracle dislocation**: Ethena USDe on Binance in October 2025.

The common thread is run risk. BIS work on stablecoins emphasizes reserve quality, transparency, redemption guarantees, and the failure of many stablecoins to maintain continuous parity. NBER work adds two useful mechanisms: first-mover advantage in runs and the way concentrated arbitrage can both stabilize secondary prices and amplify redemption pressure.

## Case Selection

| Event | Date | Peg stress | Recovery outcome | Core failure mode |
|-------|------|------------|------------------|-------------------|
| TerraUSD (UST) | May 2022 | Fell from $1 to near zero | No recovery | Algorithmic mint-burn loop with endogenous LUNA collateral |
| USDC | Mar 2023 | Traded around $0.86-$0.88 at the trough | Recovered after U.S. depositor guarantee and Circle liquidity updates | Reserve counterparty concentration at SVB |
| BUSD | Feb-Mar 2023 | No sustained market depeg | Orderly wind-down and redemption | Regulatory/issuer continuity shock |
| TrueUSD (TUSD) | Jan 2024 | Fell to roughly $0.97-$0.984 | Partial recovery after reserve-attestation response | Reserve transparency concerns and exchange liquidity outflows |
| Ethena USDe | Oct 2025 | Fell to about $0.65 on Binance only | Rapid recovery; on-chain venues reportedly stayed close to peg | Centralized exchange oracle/liquidity dislocation |

## Event Analysis

### 1. TerraUSD (UST) - May 2022

**Trigger mechanism**

UST depended on a mint-burn arbitrage with LUNA: one UST could be exchanged for $1 worth of LUNA. Once confidence fell, redemptions minted more LUNA, LUNA's price fell, and each further UST redemption required more LUNA issuance. This made the collateral base endogenous and reflexive.

**Contagion path**

- Anchor withdrawals weakened the primary demand sink for UST.
- UST selling moved across Curve, centralized exchanges, and other chains.
- LUNA inflation destroyed collateral value.
- Losses fed into broader crypto credit stress, including lenders and funds exposed to the Terra ecosystem.

**Recovery timeline**

There was no functional recovery. The peg defense failed within days, and the ecosystem was later relaunched without restoring UST's original claim.

**Protocol mitigations implied**

- Avoid reserve designs where the stabilizing asset is a volatile native token.
- Maintain exogenous, liquid, bankruptcy-remote reserves.
- Publish reserve composition and stress tests before a crisis.
- Cap circular leverage that turns a peg defense into new collateral dilution.

### 2. USDC - Silicon Valley Bank Crisis, March 2023

**Trigger mechanism**

Circle disclosed that $3.3 billion of USDC reserves, roughly 8% of the reserve base at the time, remained at Silicon Valley Bank after SVB entered receivership. The stablecoin was still asset-backed, but holders did not know whether uninsured bank deposits would be made whole.

**Contagion path**

- USDC holders sold in secondary markets instead of waiting for bank reopening.
- DAI and other stablecoins with USDC-heavy reserves also traded below peg.
- DeFi pools with USDC pairs suffered large imbalances and slippage.
- The event connected traditional-bank counterparty risk directly to on-chain money markets.

**Recovery timeline**

USDC recovered after U.S. regulators announced that SVB depositors would be protected and Circle announced that the $3.3 billion reserve deposit would be fully available. The recovery was fast because the shock was a reserve-access shock, not a permanent asset shortfall.

**Protocol mitigations implied**

- Diversify cash across banking partners.
- Hold more reserves in short-duration Treasury instruments and custodial arrangements less exposed to single-bank failure.
- Maintain emergency mint/redeem rails across multiple banks.
- Communicate reserve exposures with enough speed and precision to prevent information vacuums.

### 3. BUSD - Regulatory Wind-Down, 2023

**Trigger mechanism**

BUSD was not a classic market depeg. The shock came from NYDFS directing Paxos to stop minting new Paxos-issued BUSD and Paxos ending its relationship with Binance for the branded token. Paxos stated that existing BUSD remained fully backed and redeemable.

**Contagion path**

- Users rotated from BUSD into USDT, TUSD, FDUSD, and USDC.
- Binance reduced reliance on BUSD pairs and incentives.
- The broader stablecoin market learned that regulatory continuity can matter even when reserve collateral is sound.

**Recovery timeline**

The token entered an orderly wind-down rather than a re-peg cycle. Paxos later reported redeeming $7.9 billion of BUSD in 32 days without market disruption.

**Protocol mitigations implied**

- Separate reserve solvency from issuer/regulatory continuity risk.
- Maintain clear redemption rights after minting stops.
- Minimize dependence on one exchange sponsor or brand relationship.
- Prepare migration paths before regulatory actions force them.

### 4. TrueUSD (TUSD) - January 2024

**Trigger mechanism**

TUSD fell below peg after heavy selling on Binance and renewed concerns around reserve attestations. Reports also linked the episode to changes in exchange incentives and launch-pool demand, which reduced natural buyers while holders shifted into USDT.

**Contagion path**

- Selling pressure was concentrated on exchange order books.
- The incident damaged confidence in reserve transparency more than it threatened the full stablecoin market.
- Liquidity moved toward larger stablecoins with deeper redemption/arbitrage channels.

**Recovery timeline**

TUSD moved back toward peg after the issuer deployed a daily reserve-audit/attestation response, but the event showed that reserve credibility can be a live market variable, not just a compliance document.

**Protocol mitigations implied**

- Make reserve reports timely, verifiable, and resilient to auditor or service interruptions.
- Avoid overdependence on one exchange's incentive program for demand.
- Monitor net outflow pressure in major trading pairs as an early run signal.

### 5. Ethena USDe - October 2025 Binance Dislocation

**Trigger mechanism**

USDe traded as low as about $0.65 on Binance during an extreme market liquidation cascade. Available reporting and Ethena governance updates describe the event as a venue-local dislocation: Binance's internal oracle and order book diverged from deeper on-chain liquidity, while mint/redeem functionality and decentralized venues reportedly remained much closer to peg.

**Contagion path**

- The damage was concentrated in Binance collateral and liquidation systems.
- ENA, Ethena's governance token, sold off sharply during the event.
- The episode highlighted CeFi-DeFi integration risk: even if the protocol is solvent, a large venue can create forced selling through oracle design and collateral haircuts.

**Recovery timeline**

The dislocation normalized within roughly 90 minutes according to Ethena's later governance update. This was not a Terra-style structural collapse, but it was still economically important for users liquidated on the affected venue.

**Protocol mitigations implied**

- Use weighted, multi-venue oracle indices rather than a single venue order book.
- Apply conservative collateral haircuts for synthetic-dollar assets in leveraged accounts.
- Maintain direct mint/redeem or authorized participant access for large venues.
- Publicly document how secondary-market dislocations differ from protocol insolvency.

## Comparative Findings

### Trigger Mechanisms

| Category | UST | USDC | BUSD | TUSD | USDe |
|----------|-----|------|------|------|------|
| Endogenous collateral | High | None | None | None | Low |
| Reserve counterparty risk | Low | High | Low | Medium | Medium, via custodians/exchanges |
| Reserve transparency risk | High | Medium | Low | High | Medium |
| Regulatory/issuer continuity risk | Medium | Medium | High | Medium | Medium |
| Oracle/liquidity venue risk | Medium | Medium | Low | Medium | High |

### Contagion Paths

| Event | Main contagion channel | Why it spread |
|-------|------------------------|---------------|
| UST | DeFi deposits, LUNA dilution, CeFi credit exposure | Peg defense created new LUNA supply and destroyed collateral value |
| USDC | Banking reserves to DeFi pools | USDC was a reserve asset for other stablecoins and lending markets |
| BUSD | Exchange liquidity and pair migration | Minting stopped, so users rotated to substitute stablecoins |
| TUSD | Exchange order-book outflows | Confidence in attestation and demand incentives weakened together |
| USDe | Binance collateral/oracle system | A venue-specific price dislocation triggered forced liquidations |

### Recovery Timelines

| Event | Time to stress | Time to stabilization | Recovery type |
|-------|----------------|-----------------------|---------------|
| UST | Days | Never | Failed peg and abandoned design |
| USDC | Weekend crisis | Days | External banking guarantee and issuer liquidity response |
| BUSD | Regulatory announcement | Months-long redemption window | Orderly wind-down |
| TUSD | Days | Days to partial normalization | Attestation and liquidity response |
| USDe | Minutes to hours | About 90 minutes on affected venue | Market microstructure normalization |

## BIS/NBER Run-Risk Framing

The events map well onto three run-risk mechanisms:

1. **First-mover advantage**: Holders who exit early receive par or near-par value; late holders absorb losses or illiquidity. UST is the strongest example, but USDC, TUSD, and USDe also showed rapid secondary-market exit incentives.

2. **Public information shocks**: BIS Working Paper No. 1164 argues that reserve information can either reduce or increase run risk depending on perceived reserve quality and conversion costs. USDC's SVB disclosure was stabilizing only after regulators removed the depositor-loss uncertainty.

3. **Concentrated arbitrage and redemption access**: NBER work on stablecoin runs shows that concentrated arbitrage can support secondary-market price stability, but it also affects run dynamics because only some agents can redeem directly at par. This distinction explains why retail holders often sell below peg while authorized or institutional actors arbitrage through primary redemption.

The BIS "Will the real stablecoin please stand up?" study of 68 stablecoins is a useful baseline: continuous parity has been the exception, not the rule, and full on-demand redemption cannot be assumed without legal, operational, and reserve evidence.

## Practical Risk Checklist

For future stablecoin reviews, classify peg risk across seven axes:

1. **Collateral type**: fiat cash, Treasury bills, crypto collateral, endogenous token, delta-neutral derivatives.
2. **Reserve location**: bank deposits, custodians, money market funds, centralized exchanges, on-chain smart contracts.
3. **Redemption rights**: who can redeem, minimum size, fees, timing, and legal enforceability.
4. **Arbitrage structure**: number of direct redeemers, market-maker dependence, venue depth.
5. **Oracle design**: single order book vs weighted multi-venue index.
6. **Issuer/regulatory continuity**: license status, supervisory authority, and wind-down plan.
7. **Composability exposure**: use as collateral in lending, liquidity pools, structured products, and leveraged accounts.

## References

- BIS CPMI Paper No. 141, "Will the real stablecoin please stand up?" (2023): https://www.bis.org/publ/bppdf/bispap141.htm
- BIS Working Paper No. 1164, "Public information and stablecoin runs" (2024; revised 2025): https://www.bis.org/publ/work1164.htm
- BIS Bulletin No. 73, "Stablecoins versus tokenised deposits: implications for the singleness of money" (2023): https://www.bis.org/publ/bisbull73.htm
- NBER Working Paper No. 31160, "Anatomy of a Run: The Terra Luna Crash" (2023): https://www.nber.org/papers/w31160
- NBER Working Paper No. 33882, "Stablecoin Runs and the Centralization of Arbitrage" (2025): https://www.nber.org/papers/w33882
- NBER Working Paper No. 30796, "Leverage and Stablecoin Pegs" (2022): https://www.nber.org/papers/w30796
- Circle, "$3.3 Billion of USDC Reserve Risk Removed, Dollar De-peg Closes" (2023): https://www.prnewswire.com/news-releases/3-3-billion-of-usdc-reserve-risk-removed-dollar-de-peg-closes-301769820.html
- Paxos, "Paxos Will Halt Minting New BUSD Tokens" (2023): https://www.paxos.com/newsroom/paxos-will-halt-minting-new-busd-tokens
- Paxos, "Paxos Manages the Safe Redemption of $7.9B BUSD in One Month Without Market Disruption" (2023): https://www.paxos.com/newsroom/paxos-manages-the-safe-redemption-of-7-9b-busd-in-one-month-without-market-disruption
- NYDFS, "Notice Regarding Paxos-Issued BUSD": https://www.dfs.ny.gov/consumers/alerts/Paxos_and_Binance
- Cointelegraph, "TrueUSD stablecoin depegs as holders dump $330M in TUSD" (2024): https://cointelegraph.com/news/tusd-stablecoin-depeg-justin-sun-binance-launchpool-true-usd
- Ethena Governance, "Ethena's October 2025 Governance Update" (2025): https://gov.ethenafoundation.com/t/ethenas-october-2025-governance-update/711
- CoinDesk, "No, Ethena's USDe Didn't De-peg" (2025): https://www.coindesk.com/markets/2025/10/13/no-ethena-s-usde-didn-t-de-peg
