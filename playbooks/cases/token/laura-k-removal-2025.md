Title: LAURA auto-liquidity K shrink exploit (2025-01-01)

Summary
- Type: token with auto-liquidity adjustment removes LP tokens when x*y=k exceeds threshold.
- Preconditions: public `removeLiquidityWhenKIncreases` callable; token reserves tracked internally and can be skewed via flashloan/AMM ops; no rate limits.
- Impact: attacker shrinks LAURA reserves in WETH/LAURA pair, then withdraws liquidity and swaps back, netting ~12.34 ETH (~$41k) from pool.
- Representative tx: 0xef34f4fdf03e403e3c94e96539354fb4fe0b79a5ec927eacc63bc04108dbf420 (block ~21,529,888).

Signal
- Tokens with custom LP logic that reduces pair reserves when k crosses a threshold.
- Public hook callable by anyone; no checks on caller or rate-limit.
- Uses internal balances instead of LP token accounting; pair.sync after manual balance mutation.

Exploit Steps
- 1. Flashloan WETH (30k), swap a portion to LAURA.
- 2. Add liquidity (LAURA/WETH) with crafted amounts to raise k above threshold.
- 3. Call `removeLiquidityWhenKIncreases()` to reduce LAURA reserve in pair while keeping WETH.
- 4. Remove liquidity and swap LAURA→WETH, repay loan, keep profit.

Key Code/Storage
- Vulnerable token: 0x05641e33fd15baf819729df55500b07b82eb8e89 (LAURA).
- Function `removeLiquidityWhenKIncreases` subtracts tokens from pair balance when `currentK > 1.05 * INITIAL_UNISWAP_K`, then `pair.sync()`.
- Pair: 0xb292678438245Ec863F9FEa64AFfcEA887144240 (LAURA/WETH).

Fix/Detect
- Remove or gate auto-liquidity hooks; require owner/gov-only and rate-limit, or delete manual pair balance edits.
- Avoid manual reserve edits; rely on AMM invariant; add circuit breaker on k-based actions.
- Monitoring: alert on external calls to `removeLiquidityWhenKIncreases`, sudden reserve drops on token side, large flashloan-driven add/remove liquidity cycles.

Refs
- Attack tx: https://etherscan.io/tx/0xef34f4fdf03e403e3c94e96539354fb4fe0b79a5ec927eacc63bc04108dbf420
- Attacker: https://etherscan.io/address/0x25869347f7993c50410a9b9b9c48f37d79e12a36
- PoC author note: https://twitter.com/rotcivegaf
