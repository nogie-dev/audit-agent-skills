Title: Price Oracle Manipulation

Summary
- Oracles derived from manipulable sources (AMM spot, thin liquidity, staleness).
- Preconditions: attacker can trade to skew price or feed custom oracle without sanity bounds.
- Impact: under/over-collateralization, cheap borrows, inflated minting, liquidations avoided/forced.
- Representative: bZx 2020, Mango 2022, multiple lending forks using AMM spot.

Signal
- On-chain pricing from single pool without TWAP; no liquidity floor checks.
- Missing staleness/heartbeat checks; oracle read writable via governance/admin.
- Price used mid-transaction after attacker-driven swap; lack of cap/floor bounds.

Exploit Steps
- 1. Manipulate reference market (swap large, seed pool, flash loan).
- 2. Read oracle in vulnerable function (borrow/mint/liquidate).
- 3. Realize profit (withdraw inflated collateral, borrow cheap) before price normalizes.

Key Code/Storage
- Oracle reader functions; TWAP config; decimals conversion; scaling factors.
- Pools/feeds addresses; ability to override via admin or upgrade.
- Loan-to-value math using spot price without bounds.

Refs
- https://rekt.news/mango-rekt/
- https://samczsun.com/taking-undercollateralized-borrows-for-fun-and-for-profit/
