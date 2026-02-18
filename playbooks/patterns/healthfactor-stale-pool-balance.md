Title: Health factor uses stale pool balance during redeem

Summary
- Health factor or collateral balance uses current pool token balance after burning shares, so pending redemptions remain in the balance and inflate lend value.
- Impact: borrowers pass health checks and redeem collateral/borrowed assets they shouldn't, draining pools.

Signal
- `userBalanceOf` = `poolTokenBalance * userShare / totalShares` using `balanceOf(pool)` even after shares are burned.
- Redeem flow burns shares, then reads pool balance without subtracting pending withdrawal; health check before transfer relies on stale balance.
- Flashloan-friendly paths allowing borrow + redeem in one tx.

Exploit Steps
- 1. Inflate shares (possibly with flashloan) and borrow target asset.
- 2. Redeem collateral; health factor computed with pre-withdraw balance so check passes.
- 3. Redeem remaining assets, repay flashloan, keep borrowed asset.

Key Code/Storage
- `userBalanceOftoken*` using raw pool `balanceOf` instead of tracked total assets minus pending redemption.
- Health check functions called after burning shares but before balances update.

Refs
- Example: UniLend stale balance health factor 2025-01-12 (playbooks/cases/lending/unilend-healthfactor-stale-balance-2025.md)
