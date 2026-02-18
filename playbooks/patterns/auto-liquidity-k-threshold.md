Title: Auto-liquidity k-threshold hooks on tokens

Summary
- Tokens implement hooks that shrink/adjust AMM pair balances when x*y=k exceeds a threshold; callable by anyone.
- Impact: attackers can manipulate reserves via flashloan, trigger the hook, and drain paired asset on unwinding liquidity/swaps.

Signal
- Functions like `removeLiquidityWhenKIncreases` that mutate pair balances directly and call `sync()`.
- Thresholds based on k without caller checks, TWAP, or rate limits.
- Uses internal balance tracking rather than LP tokens; adjusts only one side of the pair.

Exploit Steps
- 1. Flashloan to skew reserves and push k over threshold.
- 2. Call the auto-liquidity hook to reduce one reserve while preserving the other.
- 3. Remove liquidity or swap back to capture the imbalance.

Key Code/Storage
- Token-side hook altering pair balances, e.g., `_balances[uniswapV2Pair] -= ...; pair.sync();` when `currentK > threshold`.

Refs
- Example: LAURA auto-liquidity exploit 2025-01-01 (playbooks/cases/token/laura-k-removal-2025.md)
