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

Fix/Detect
- Use robust oracle (Chainlink/TWAP with liquidity floor); cap per-block price change.
- Require dual-source confirmation; apply collateral factor caps for low-liquidity assets.
- Monitoring: price delta vs reference, borrow spikes during thin liquidity windows.

Refs
- https://rekt.news/mango-rekt/
