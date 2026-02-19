Title: Invariant precision loss via inconsistent rounding

Summary
- Stable/Composable pool math upscales with one-way rounding (mulDown) and downscales with divUp/divDown, causing precision loss that can shrink invariant D and misprice LP/BPT.

Signal
- Upscale uses mulDown for all amounts; downscale uses divUp/divDown.
- GivenOut swap path rounds down user amount before solving swap equation.
- BPT price/invariant not bounded by TWAP or sanity checks.

Exploit Steps
- 1. Push balances to rounding boundary.
- 2. Swap with crafted amounts to trigger mulDown precision loss, underestimating amountIn/out and lowering invariant.
- 3. Swap back/redeem at deflated BPT price.

Key Code/Storage
- `_upscale` mulDown vs `_downscale` divUp/Down in stable math.
- `_swapGivenOut` uses rounded amountOut to solve for amountIn.
- BPT price ≈ D / totalSupply.

Examples
- playbooks/cases/dex/balancer-csp-invariant-rounding-2025.md
