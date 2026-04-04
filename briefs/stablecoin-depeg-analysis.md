# Comparative Analysis of Stablecoin Depeg Events (2022–2025)

**Research Brief — July 2025**

---

## Abstract

Stablecoins have emerged as critical infrastructure in digital asset markets, with aggregate market capitalization exceeding $250 billion by mid-2025. Yet the period from 2022 to 2025 exposed fundamental fragilities across every stablecoin design paradigm—algorithmic, fiat-backed, crypto-collateralized, and regulated custodial. This brief presents a comparative analysis of five major depeg events: the catastrophic collapse of TerraUSD (UST) in May 2022 (~$40–60 billion in value destroyed), the USDC banking-contagion depeg during the Silicon Valley Bank failure in March 2023 (trough at $0.87), episodic USDT deviations, the regulatory-driven wind-down of BUSD in February 2023, and the cascading depeg of DAI due to concentrated USDC collateral exposure. Drawing on the run-risk frameworks developed by the Bank for International Settlements (BIS) and the National Bureau of Economic Research (NBER), we identify common vulnerability patterns, compare trigger mechanisms and recovery dynamics, and synthesize protocol-level and regulatory mitigations that have since reshaped the stablecoin landscape.

---

## 1. Introduction: The Stability Premise

Stablecoins are digital tokens designed to maintain a fixed exchange rate—typically 1:1 with the U.S. dollar—through varying combinations of asset reserves, algorithmic mechanisms, and arbitrage incentives. By early 2022, total stablecoin market capitalization reached approximately $168 billion, accounting for a significant share of on-chain transaction volume and serving as the primary unit of account in decentralized finance (DeFi) (CoinGecko, 2023).

The stability premise rests on three pillars:

1. **Reserve adequacy** — sufficient liquid assets to honor redemptions at par.
2. **Arbitrage efficiency** — market participants who profit from correcting price deviations, thereby restoring the peg.
3. **Confidence equilibrium** — the belief among holders that the peg will hold, which is self-reinforcing until it is not.

When any pillar fractures, the result mirrors dynamics well-studied in traditional finance: bank runs, money market fund (MMF) "breaking the buck," and sovereign currency crises. The Federal Reserve Bank of New York's Staff Report No. 1073 (2023, revised 2024) formally establishes the parallel between stablecoin runs and MMF runs, noting that stablecoins exhibit "flight-to-safety" dynamics during market stress, with flows shifting from riskier to safer stablecoins in patterns analogous to the 2008 and 2020 MMF crises (Cipriani et al., 2023).

The following case studies trace how these pillars failed—through different mechanisms and with vastly different outcomes—across the 2022–2025 period.

---

## 2. Case Studies

### 2.1 TerraUSD / Luna (May 2022) — Algorithmic Death Spiral

**Background.** TerraUSD (UST) was an algorithmic stablecoin that maintained its dollar peg through a mint-and-burn arbitrage mechanism with its companion token LUNA. If UST traded below $1, holders could redeem 1 UST for $1 worth of newly minted LUNA, reducing UST supply and theoretically restoring the peg. The Anchor Protocol, Terra's flagship lending application, attracted over $14 billion in UST deposits by offering an unsustainable ~20% annual yield, which became the dominant source of UST demand (Liu, Makarov & Schoar, 2023).

**Trigger and timeline.**

| Date | Event |
|------|-------|
| May 7, 2022 | ~$150M UST withdrawn from Curve Finance pool; liquidity thins |
| May 7–8 | Two large traders swap ~$185M UST for USDC on Curve; UST slips to ~$0.98 |
| May 9–10 | Panic accelerates; mass redemptions mint billions of LUNA, crashing its price |
| May 11–13 | LUNA hyperinflates from ~450M to 6.5T supply; UST falls to $0.08 |
| May 13 | Terra blockchain halted; ecosystem value near zero |

**Mechanism.** The depeg exposed the reflexive fragility of algorithmic stablecoins: as UST fell below $1, rational actors redeemed for LUNA and immediately sold, depressing LUNA's price, which in turn reduced the backing capacity for UST—a classic death spiral. The Luna Foundation Guard (LFG) deployed approximately $3.5 billion in Bitcoin reserves in a failed defense, but the redemption velocity overwhelmed all intervention capacity.

**Losses.** Total ecosystem value destruction exceeded $40–60 billion. Anchor Protocol alone lost ~$14 billion in deposits. On-chain forensics by Nansen revealed that larger, more sophisticated wallets exited earlier with smaller losses, while retail participants suffered disproportionately—a pattern consistent with informed-trader advantages in traditional bank runs (Nansen, 2022; Liu et al., 2023).

**Contagion.** The collapse triggered liquidation cascades across DeFi, contributed to the insolvencies of Three Arrows Capital, Celsius Network, and Voyager Digital, and accelerated the 2022 crypto bear market. USDT briefly fell to $0.95 on some exchanges in the immediate aftermath.

---

### 2.2 USDC / Silicon Valley Bank (March 2023) — Banking Contagion

**Background.** USD Coin (USDC), issued by Circle, was the second-largest stablecoin with approximately $40 billion in circulation. Reserves were held as ~77% in short-dated U.S. Treasuries (managed by BlackRock at BNY Mellon) and ~23% in cash deposits across multiple banking partners, including Silicon Valley Bank (SVB).

**Trigger and timeline.**

| Date | Event |
|------|-------|
| March 9, 2023 | Circle initiates wire transfer to withdraw funds from SVB; transfer not processed |
| March 10 | SVB collapses; FDIC takes receivership; Circle confirms $3.3B (8% of reserves) stranded |
| March 11 | USDC drops to $0.87 on secondary markets; Coinbase pauses USDC-USD conversions |
| March 12 | U.S. Treasury, Fed, and FDIC announce all SVB depositors will be made whole |
| March 13 | USDC returns to $1.00; automated minting/redemption resumes |

**Mechanism.** This event demonstrated that even well-collateralized fiat-backed stablecoins carry counterparty risk through their banking relationships. The 8% reserve exposure to a single failed bank was sufficient to break market confidence, triggering a weekend of panic selling on secondary markets where direct redemption was unavailable. The depeg was resolved not by Circle's own actions but by extraordinary government intervention—the joint FDIC/Fed/Treasury guarantee of all SVB deposits.

**Depth and duration.** The depeg reached a trough of $0.87 (13% deviation) and lasted approximately 60 hours—from Friday evening to Monday morning—a timeline constrained by banking hours and government decision-making rather than any on-chain mechanism.

---

### 2.3 USDT / Tether (Various Minor Depegs)

**Background.** Tether (USDT) is the largest stablecoin by market capitalization (~$83B in mid-2023, ~$140B+ by 2025). Despite persistent transparency concerns regarding its reserve composition, USDT has maintained relative peg stability with only brief, minor deviations.

**Notable events.**

| Period | Trigger | Trough Price | Duration |
|--------|---------|-------------|----------|
| May 2022 | Terra/Luna contagion, flight-to-safety | $0.9493 | ~24–48 hours |
| Nov 2022 | FTX collapse, general market panic | ~$0.98 | <24 hours |
| Dec 2023 | Tether wallet-freezing policy announcement | ~$0.985 | <24 hours |

**Mechanism.** USDT depegs have typically been demand-side shocks—moments when holders rush to exit crypto-denominated assets into fiat—rather than reserve adequacy crises. Ironically, during the March 2023 SVB event, USDT traded at a *premium* (~$1.02–$1.06 on some exchanges) as holders fled USDC into USDT, illustrating the flight-to-safety dynamic within the stablecoin ecosystem itself.

**Resilience factors.** Tether's restricted redemption architecture—where only authorized institutional arbitrageurs can redeem directly—creates a centralized arbitrage structure. As the NBER's Ma, Zeng, and Zhang (2024) demonstrate, this centralization enhances day-to-day price stability but may amplify run risk in tail scenarios by concentrating exit capacity among a small number of actors.

---

### 2.4 BUSD / Binance USD (February 2023) — Regulatory Shutdown

**Background.** Binance USD (BUSD) was a fiat-backed stablecoin issued by Paxos Trust Company under a New York Department of Financial Services (NYDFS) trust charter, reaching a peak market capitalization of approximately $23 billion in late 2022.

**Trigger and timeline.**

| Date | Event |
|------|-------|
| Feb 13, 2023 | NYDFS orders Paxos to cease minting new BUSD tokens |
| Feb 13, 2023 | SEC issues Wells Notice to Paxos, alleging BUSD may be an unregistered security |
| Feb–Dec 2023 | Orderly wind-down; market cap declines from $16B to <$1B |
| Jul 2024 | SEC closes investigation without enforcement action |

**Mechanism.** Unlike other depeg events, BUSD's demise was not a market-driven run but a regulatory kill-switch. NYDFS cited "unresolved issues related to Paxos' oversight of its relationship with Binance," including anti-money laundering compliance gaps. Because Paxos continued to honor 1:1 redemptions throughout the wind-down, BUSD never experienced a significant market price deviation—it maintained its peg while shrinking to irrelevance.

**Significance.** The BUSD case demonstrates a fourth vector of stablecoin failure: regulatory risk. Even a fully-reserved, peg-maintaining stablecoin can be effectively terminated by regulatory action, destroying its utility without a single dollar of holder loss. Market cap fell from $23B to under $1B within 12 months—a 95%+ supply contraction.

---

### 2.5 DAI / MakerDAO (March 2023) — Cascading Collateral Contagion

**Background.** DAI is a crypto-collateralized, governance-managed stablecoin issued by MakerDAO. By March 2023, approximately 40–51% of DAI's backing consisted of USDC held through the Peg Stability Module (PSM), which enabled 1:1 swaps between DAI and USDC to maintain peg stability.

**Trigger.** The USDC depeg during the SVB crisis immediately transmitted to DAI through the PSM mechanism. As USDC fell to $0.87, the market repriced DAI's collateral pool accordingly.

**Depth.** DAI dropped to approximately $0.88–$0.89 on March 11, 2023—closely tracking USDC's deviation due to the high collateral concentration.

**Response.** MakerDAO governance executed emergency actions within 48 hours:

- Slashed USDC-related debt ceilings to zero
- Reduced daily DAI minting limits via the USDC-backed PSM
- Increased swap fees to slow USDC dumping into the protocol
- Initiated proposals to diversify collateral toward ETH, real-world assets, and U.S. Treasuries

**Outcome.** DAI recovered its peg within 48 hours following the government guarantee of SVB deposits. In the months after, MakerDAO reduced USDC collateral concentration from >50% to under 10%, significantly expanding allocations to U.S. Treasury instruments and crypto-native assets (The Defiant, 2023).

---

## 3. Comparative Analysis

| Dimension | UST/Luna | USDC/SVB | USDT | BUSD | DAI |
|-----------|----------|----------|------|------|-----|
| **Date** | May 2022 | Mar 2023 | Various | Feb 2023 | Mar 2023 |
| **Type** | Algorithmic | Fiat-backed | Fiat-backed | Fiat-backed (regulated) | Crypto-collateralized |
| **Trigger** | Loss of confidence in algorithmic peg; large Curve withdrawals | Banking counterparty failure (SVB collapse) | Market-wide panic events; contagion | Regulatory order (NYDFS + SEC Wells Notice) | Cascading contagion from USDC collateral exposure |
| **Mechanism** | Reflexive death spiral: UST redemption → LUNA hyperinflation → further UST depeg | Reserve inaccessibility (8% stranded at SVB) → market panic on secondary markets | Demand-side shocks; flight-from-crypto dynamics | Regulatory halt on minting; orderly redemption wind-down | PSM collateral repricing; mechanical transmission of USDC depeg |
| **Depeg Depth** | $0.08 (92% loss) | $0.87 (13% loss) | $0.9493 worst case (5% loss) | No significant depeg (peg maintained during wind-down) | $0.88 (12% loss) |
| **Duration** | Permanent (no recovery) | ~60 hours | <48 hours per event | N/A (orderly wind-down over 12 months) | ~48 hours |
| **Recovery** | None — total collapse | Full peg recovery via government intervention | Self-correcting via arbitrage | Peg maintained; supply went to zero | Full recovery; collateral restructured |
| **Value Destroyed** | ~$40–60B | ~$0 (temporary mark-to-market) | Negligible | $0 (holder losses); $22B+ supply reduction | ~$0 (temporary mark-to-market) |
| **Contagion** | Global crypto bear market; CeFi insolvencies | DAI depeg; DeFi liquidity stress | Minimal | Binance ecosystem rebalancing | Contained to DAI holders |

---

## 4. Run-Risk Framework: BIS and NBER Perspectives

The depeg events of 2022–2025 can be systematically understood through the run-risk frameworks developed in recent academic and regulatory research.

### 4.1 The BIS Framework: Public Information and Reserve Quality

Ahmed, Aldasoro, and Duley (BIS Working Paper No. 1164, 2024) model stablecoin peg stability as a function of public information about reserve asset quality. Their key findings include:

- **Transparency paradox**: Greater public information about reserves reduces run risk *only when* market confidence in reserve quality is high. When reserves are perceived as low-quality or illiquid, increased transparency can *accelerate* runs by coordinating pessimistic beliefs.
- **Transaction cost buffer**: Higher fiat-conversion transaction costs serve as a natural run deterrent, slowing outflows during stress. This helps explain why USDT—with its restricted redemption access—has proven more peg-resilient than USDC during acute stress episodes.
- **Crypto-collateralized fragility**: For stablecoins backed by volatile crypto assets, small shocks are absorbed, but large adverse shocks can rapidly break the peg even with initial overcollateralization—precisely the UST/Luna dynamic.

### 4.2 The NBER Framework: Centralized Arbitrage and Run Dynamics

Two NBER papers provide complementary analytical lenses:

**Liu, Makarov, and Schoar (NBER WP 31160, 2023) — "Anatomy of a Run: The Terra Luna Crash."** This paper documents the Terra collapse as a classical bank run accelerated by blockchain transparency. Key findings include:

- The crash was not triggered by a single coordinated attack but by deteriorating fundamentals and a cascade of rational exits.
- Blockchain transparency, which is often cited as a stabilizing feature, actually *accelerated* the run by allowing all participants to observe large outflows in real-time, coordinating panic.
- Wealthier and more sophisticated wallets exited 2–3 days earlier than retail participants, capturing significantly more residual value.

**Ma, Zeng, and Zhang (NBER WP 33882, 2024) — "Stablecoin Runs and the Centralization of Arbitrage."** This paper formalizes the tradeoff between price stability and run vulnerability:

- Stablecoins that restrict direct redemption to a small set of institutional arbitrageurs (e.g., Tether) achieve tighter day-to-day pegs through more efficient arbitrage.
- However, this centralization *amplifies* run risk: in a crisis, secondary market prices become more sensitive to sell-offs because the arbitrage channel is capacity-constrained.
- The implication is a fundamental tension: **policies designed to improve normal-state stability may increase tail-risk fragility**.

### 4.3 The Federal Reserve Parallel: Stablecoins as Digital MMFs

The Federal Reserve Bank of New York's Staff Report No. 1073 (Cipriani et al., 2023, revised 2024) extends the MMF-run literature to stablecoins, identifying:

- A **"break-the-buck" threshold** effect: once a stablecoin's price dips below $1.00, redemption outflows accelerate non-linearly—a dynamic observed in both the UST and USDC events.
- **Flight-to-safety flows**: during the March 2023 SVB crisis, capital flowed *from* USDC *to* USDT and *from* smaller stablecoins to larger ones, mirroring the institutional-to-government MMF flows observed in 2008.
- The analogy suggests that stablecoin regulation should draw heavily on the post-2008 MMF reform toolkit: liquidity buffers, redemption gates, swing pricing, and enhanced disclosure.

---

## 5. Protocol Mitigations and Lessons Learned

The depeg events catalyzed significant structural changes across the stablecoin ecosystem:

### 5.1 Reserve Diversification and Transparency

- **Circle (USDC)**: Post-SVB, Circle accelerated the shift of cash reserves away from regional banks toward systemically important financial institutions and expanded Treasury holdings. Monthly reserve attestations by Deloitte were supplemented with more granular disclosures.
- **MakerDAO (DAI)**: Reduced USDC concentration from >50% to <10% of collateral within six months, replacing it with U.S. Treasury exposure via real-world asset vaults and increased ETH-based collateral.
- **Tether (USDT)**: Published quarterly reserve breakdowns showing a progressive shift from commercial paper to U.S. Treasury bills, though independent audits remain absent.

### 5.2 Mechanism Design Improvements

- **Circuit breakers**: MakerDAO introduced PSM circuit breakers to limit the rate at which collateral stress can transmit to DAI supply.
- **Redemption architecture**: The industry moved toward redemption mechanisms that balance accessibility (reducing flight-to-safety pressure) with rate-limiting (preventing catastrophic outflows).
- **Overcollateralization requirements**: Crypto-collateralized protocols increased collateralization ratios and tightened liquidation parameters.

### 5.3 Regulatory Evolution

- **NYDFS enforcement**: The BUSD shutdown demonstrated that regulatory bodies can and will intervene directly in stablecoin operations, establishing a precedent for compliance-first issuance.
- **U.S. GENIUS Act (2025)**: The first comprehensive federal stablecoin framework in the United States, mandating reserve requirements, audit standards, and issuer licensing—directly informed by the failures documented in this brief.
- **BIS and FSB coordination**: International standard-setting bodies have incorporated stablecoin run-risk into their financial stability frameworks, with the FSB's 2023 recommendations explicitly referencing the UST collapse and SVB contagion as motivating case studies.

### 5.4 Synthesis of Lessons

1. **No design paradigm is immune**: Algorithmic (UST), fiat-backed (USDC), crypto-collateralized (DAI), and regulated custodial (BUSD) stablecoins all experienced distinct failure modes.
2. **Confidence is non-linear**: Peg deviations beyond a threshold (~1–2%) trigger self-reinforcing exit dynamics that overwhelm arbitrage capacity.
3. **Transparency is a double-edged sword**: Real-time on-chain visibility can coordinate panic as easily as it can reassure holders.
4. **Collateral concentration kills**: DAI's >50% USDC exposure and Circle's 8% SVB exposure both demonstrate that single-counterparty risk can propagate systemic stress.
5. **Recovery depends on external backstops**: USDC recovered because of government intervention, not market mechanisms. UST had no such backstop.

---

## 6. Conclusion

The stablecoin depeg events of 2022–2025 constitute the first comprehensive stress test of digital dollar infrastructure. They revealed that stablecoins, regardless of design, are subject to the same run dynamics that have characterized banking and money market crises for centuries—dynamics now amplified by blockchain transparency, 24/7 markets, and the absence of a lender of last resort.

The BIS and NBER frameworks provide rigorous analytical tools for understanding these events: the reflexive relationship between reserve quality and holder confidence (BIS), the tension between arbitrage efficiency and run vulnerability (NBER), and the parallels with money market fund fragility (Fed NY). Together, they point toward a regulatory equilibrium that combines reserve quality mandates, transparency requirements calibrated to avoid coordination of panic, redemption architecture that balances access with stability, and—ultimately—some form of public backstop for systemically important stablecoins.

The stablecoin market's recovery to over $250 billion by 2025, driven by legislative progress such as the U.S. GENIUS Act and growing institutional adoption, suggests that the market has internalized many lessons from this turbulent period. Whether those lessons prove sufficient for the next stress episode remains an open question.

---

## 7. References

1. Ahmed, R., Aldasoro, I., & Duley, C. (2024). "Public Information and Stablecoin Runs." *BIS Working Paper No. 1164*. Bank for International Settlements. https://www.bis.org/publ/work1164.htm

2. Chainalysis. (2022). "The Trades That Triggered TerraUSD's Collapse." https://www.chainalysis.com/blog/how-terrausd-collapsed/

3. Circle. (2023). "$3.3 Billion of USDC Reserve Risk Removed, Dollar De-peg Closes." Press Release, March 13, 2023. https://www.circle.com/pressroom/3-3-billion-of-usdc-reserve-risk-removed-dollar-de-peg-closes

4. Cipriani, M., et al. (2023, revised 2024). "Runs and Flights to Safety: Are Stablecoins the New Money Market Funds?" *Federal Reserve Bank of New York Staff Report No. 1073*. https://www.newyorkfed.org/research/staff_reports/sr1073.html

5. CoinGecko. (2023). "Stablecoins Statistics: 2023 Report." https://www.coingecko.com/research/publications/stablecoins-statistics

6. Federal Reserve Board. (2025). "In the Shadow of Bank Runs: Lessons from the Silicon Valley Bank Failure and Its Impact on Stablecoins." *FEDS Notes*, December 17, 2025. https://www.federalreserve.gov/econres/notes/feds-notes/in-the-shadow-of-bank-run-lessons-from-the-silicon-valley-bank-failure-and-its-impact-on-stablecoins-20251217.html

7. Liu, J., Makarov, I., & Schoar, A. (2023). "Anatomy of a Run: The Terra Luna Crash." *NBER Working Paper No. 31160*. National Bureau of Economic Research. https://www.nber.org/papers/w31160

8. Ma, Y., Zeng, Y., & Zhang, A. L. (2024). "Stablecoin Runs and the Centralization of Arbitrage." *NBER Working Paper No. 33882*. National Bureau of Economic Research. https://www.nber.org/papers/w33882

9. Nansen. (2022). "On-Chain Forensics: Demystifying TerraUSD De-peg." https://www.nansen.ai/research/on-chain-forensics-demystifying-terrausd-de-peg

10. New York Department of Financial Services. (2023). "Notice Regarding Paxos-Issued BUSD." https://www.dfs.ny.gov/consumers/alerts/Paxos_and_Binance

11. Paxos. (2023). "Paxos Will Halt Minting New BUSD Tokens." Press Release, February 13, 2023. https://www.paxos.com/newsroom/paxos-will-halt-minting-new-busd-tokens

12. S&P Global. (2023). "Stablecoins: A Deep Dive into Valuation and Depegging." https://www.spglobal.com/en/research-insights/special-reports/stablecoins-a-deep-dive-into-valuation-and-depegging

13. The Defiant. (2023). "DAI's Reliance On USDC Drops Below 10% As MakerDAO Expands Bond Holdings." https://thedefiant.io/news/defi/dai-s-reliance-on-usdc-drops-below-10-as-makerdao-expands-bond-holdings

---

*This research brief is intended for informational purposes. Data points are sourced from publicly available on-chain analytics, regulatory filings, and peer-reviewed working papers. All market figures are approximate and reflect conditions at the time of the referenced events.*
