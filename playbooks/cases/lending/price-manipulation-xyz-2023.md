Title: Lending Price Manipulation (XYZ 2023-style)

Summary
- Type: oracle spot manipulation to borrow undercollateralized.
- Preconditions: lending uses single AMM spot price; low liquidity pool; no TWAP/bounds.
- Impact: attacker borrows asset with minimal collateral and walks away.
- Representative: similar to Mango 2022 and multiple AMM-based lending forks.

Signal
- Borrow function reads pool reserves directly; no deviation check vs reference oracle.
- Sudden collateral value spike within same block as large borrow.

Exploit Steps
- 1. Flash borrow quote asset; inflate collateral price via AMM swap.
- 2. Borrow target asset using inflated collateral value.
- 3. Reverse swap to repay flash loan, keep borrowed asset as profit.

Key Code/Storage
- Oracle getter function; collateral factor math; per-asset collateral configs.
- Admin or governance-controlled oracle addresses without delay.

Refs
- https://rekt.news/mango-rekt/
