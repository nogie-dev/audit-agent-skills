Title: User-supplied swap route seeds price with attacker pool

Summary
- Deposit/mint flow lets caller provide arbitrary swap calldata or router/path, and uses resulting output to set share price or mint amount.
- If the price source is a fresh pool the attacker controls (e.g., Uniswap V3 pool just initialized), they can set an arbitrary exchange rate, mint overpriced shares, then unwind.

Signal
- `deposit`/`mint` accepts opaque calldata for a DEX/aggregator (ParaSwap/Augustus, 1inch, custom router) without allowlist.
- Protocol does not verify pool existence/liquidity or price bounds; trusts returned amount as fair value.
- Ability to create/initialize the referenced pool immediately before deposit.

Exploit Steps
- Create/initialize a tiny pool for the deposit/quote pair at an extreme `sqrtPriceX96`.
- Flash-loan the deposit token.
- Call `deposit` with swap calldata routing through the attacker pool, minting shares at the fake rate.
- Redeem/withdraw and repay loan, keeping the spread.

Key Code/Storage
- Deposit function parameters: router address, swapData/path provided by caller.
- Pool initialization call in the same tx trace before deposit.

Refs
- UsualMoney USD0+/sUSDS price mis-sync (2025-05-27): ParaSwap calldata + attacker-seeded V3 pool used to mint overpriced vault shares.
