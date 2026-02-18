Title: AMM Spot Manipulation → Sandwich Liquidations (Uniswap V2 style)

Summary
- Type: price manipulation via thin liquidity pool to influence downstream logic.
- Preconditions: victim trade size predictable; downstream uses spot price without TWAP/bounds.
- Impact: extract MEV, trigger bad liquidations or mispriced mints/burns.
- Representative tx: generic sandwich on Uniswap V2-style pools (no specific hash tied).

Signal
- Downstream contract reads AMM reserve ratio directly; no liquidity threshold checks.
- Large swap immediately followed by victim tx then reverse swap in same block.

Exploit Steps
- 1. Attacker flash borrows asset A, swaps to skew A/B price upward.
- 2. Victim executes function relying on skewed spot (mint/borrow/liquidate).
- 3. Attacker reverses swap to capture spread and protocol side effect (borrowed funds, liquidations).

Key Code/Storage
- `getReserves` usage without TWAP; lack of sanity check vs external oracle.
- Slippage configs ignored or too lax in downstream call.

Refs
- https://blog.openzeppelin.com/price-oracle-manipulation
