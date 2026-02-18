Title: UniLend health factor using stale pool balance (2025-01-12)

Summary
- Type: lending health factor miscalculation (uses pre-withdraw pool balance after burning shares).
- Preconditions: redeem flow burns lend shares then computes health factor using pool balance that still includes redeem amount; no fresh balance update before check.
- Impact: health factor computed too high; attacker redeems both collateral and borrowed asset, walking away with ~60 stETH after only 200 USDC pledged (loss ≈ $197K reported).
- Representative tx: https://etherscan.io/tx/0x44037ffc0993327176975e08789b71c1058318f48ddeff25890a577d6555b6ba.

Signal
- Health factor/check uses pool token balance * user lendShare after burning shares, without excluding pending redemption.
- `userBalanceOf` derives from `balanceOf(pool)` rather than internal accounting; borrow side uses shares, leading to mismatch.
- Redeem flow: `_burnLPposition` then `checkHealthFactor` with stale balances.

Exploit Steps
- 1. Seed position (e.g., deposit 200 USDC) and acquire lend shares.
- 2. Flashloan large USDC + wstETH; wrap/unwrap as needed.
- 3. Lend both USDC and stETH to inflate lend shares; borrow stETH heavily.
- 4. Redeem pledged stETH, then redeem USDC; health factor passes due to stale balance calculation.
- 5. Repay flashloans, keep ~60 stETH profit.

Key Code/Storage
- Vulnerable pool: 0x4e34dd25dbd367b1bf82e1b5527dbbe799fad0d0 (USDC/stETH UniLend V2 pool).
- `userBalanceOftoken0/1` multiply current pool token balance by user lendShare; redeem burns lendShare but uses pre-withdraw balance, inflating lendBalance.
- Health check (`checkHealthFactorLtv1`) uses these inflated lend balances vs borrow shares.

Refs
- Post-mortem: https://slowmist.medium.com/analysis-of-the-unilend-hack-90022fa35a54
- Alert: https://x.com/SlowMist_Team/status/1878651772375572573
- Attacker: https://etherscan.io/address/0x55f5f8058816d5376df310770ca3a2e294089c33
- Attack contract: https://etherscan.io/address/0x3f814e5fae74cd73a70a0ea38d85971dfa6fda21
