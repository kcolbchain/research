# Citation Standards

Citation guidelines for kcolbchain research. All contributors must cite sources using the formats below.

## General Principles

- **Prefer primary sources**: protocol documentation, governance forums, whitepapers, audits, legal/regulatory publications, public datasets, or official incident reports.
- **Use Markdown links inline** for web sources with enough context to identify the source without opening it.
- **Include access dates** for volatile sources: dashboards, market data, governance posts, issue threads, forum discussions.
- **For papers**: include author(s) or organization, title, year, and a DOI/arXiv/SSRN/canonical URL.
- **Do not cite** screenshots, social posts, or secondary summaries when a durable primary source is available.

## Citation Formats

### Academic Papers / Preprints

```
Author(s). "Title." Venue/Journal, Year. DOI/arXiv/SSRN URL.
```

Example:

```
Daian, P., et al. "Flash Boys 2.0: Frontrunning in Decentralized Exchanges, Miner Extractable Value, and Consensus." IEEE S&P, 2020. https://arxiv.org/abs/1904.05234
```

### Whitepapers / Protocol Documentation

```
Project Name. "Title." Version, Date. URL. Accessed YYYY-MM-DD.
```

Example:

```
Arbitrum Foundation. "Timeboost: A New Approach to Transaction Ordering." v1, 2024. https://research.arbitrum.io/t/decentralized-timeboost-specification/9676. Accessed 2026-05-20.
```

### Governance Posts / Forum Discussions

```
Platform. "Post Title." Author (if known). Date. URL. Accessed YYYY-MM-DD.
```

Example:

```
Arbitrum DAO Forum. "Constitutional AIP: Proposal to adopt Timeboost." 2024. https://forum.arbitrum.foundation/t/constitutional-aip-proposal-to-adopt-timeboost-a-new-transaction-ordering-policy/25167. Accessed 2026-05-20.
```

### Code Repositories / Smart Contracts

```
Owner/Repo. "File or description." Commit hash or version. URL.
```

Example:

```
kcolbchain/stablecoin-toolkit. "ComplianceModule.sol." commit a1b2c3d. https://github.com/kcolbchain/stablecoin-toolkit/blob/main/contracts/ComplianceModule.sol
```

### Dashboards / Market Data

```
Platform. "Dashboard Name." URL. Accessed YYYY-MM-DD.
```

Example:

```
L2BEAT. "Scaling Risk Analysis." https://l2beat.com/scaling/risk. Accessed 2026-05-20.
```

### Regulatory / Legal Sources

```
Authority. "Document Title." Date. URL.
```

Example:

```
NYDFS. "Notice Regarding Paxos-Issued BUSD." 2023. https://www.dfs.ny.gov/consumers/alerts/Paxos_and_Binance
```

## Inline Citation Style

Use Markdown links inline. Format: `[Source Description](URL)`. For paper references, include author-year parenthetical.

```
Optimism documents Timeboost as an optional sequencing mechanism (Optimism Docs, 2025).
```

When the citation is a specific claim:

```
As shown in the BIS Working Paper No. 1164, reserve transparency can increase run risk when holders believe quality is low [BIS 2025].
```

## Reference List

Every research brief must include a "References" section at the end. Sort alphabetically by author/organization.

## Code of Conduct

All contributors are expected to:
- Cite sources accurately and completely
- Not misrepresent what a source says
- Acknowledge competing viewpoints
- Provide access dates for dynamic content
- Flag potential conflicts of interest

Violations may result in removal of contributions and loss of contributor privileges.
