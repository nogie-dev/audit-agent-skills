Title: Virtual balance/rate manipulation with unchecked math

Summary
- Pools track virtual balances and allow external rate updates; arithmetic uses unchecked multiplications/divisions not tied to real reserves.
- Impact: crafted add/remove cycles, zero-amount updates, and rebases can skew accounting and mint/redeem disproportionate shares.

Signal
- Functions `update_rates` or similar callable externally; uses virtual_balance arrays separate from on-chain reserves.
- Large products/sums (`vb_prod_sum`) without overflow checks or bounds; zero-amount removes trigger recalculation on manipulated state.
- Interaction with rebasing tokens inside the pool.

Exploit Steps
- 1. Seed positions and perform add/remove liquidity sequences to tilt virtual balances.
- 2. Call rate update on chosen indices; use zero-amount removes to recompute state.
- 3. Apply rebase (if present) to further skew accounting; perform final remove to drain.

Key Code/Storage
- Virtual balance arrays and product/sum tracking; externally callable rate update functions; add/remove paths relying on these virtual values instead of reserves.

Refs
- Example: yETH virtual balance unsafe math (playbooks/cases/lending/yeth-virtual-balance-unsafe-math-2025.md)
