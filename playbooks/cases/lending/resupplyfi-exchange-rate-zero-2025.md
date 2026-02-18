Title: ResupplyFi reUSD exchange rate zero LTV bypass (2025-06-26)

Summary
- Type: oracle/pricing logic bug sets exchangeRate to 0, bypassing solvency checks.
- Preconditions: crvUSD-wstUSR vault empty; `convertToAssets(1e18)` returns huge value; exchangeRate computed as `1e36 / price` floors to 0; `_isSolvent` uses `_exchangeRate` directly.
- Impact: attacker inflated vault price with 2e36 crvUSD/wei, exchangeRate became 0, LTV calc returned 0 so every borrow passed solvency; borrowed up to 10M reUSD debt cap, leaving bad debt (~$9.6M).
- Representative tx: https://etherscan.io/tx/0xffbbd492e0605a8bb6d490c3cd879e87ff60862b0684160d08fd5711e7a872d3.

Signal
- Oracle uses hardcoded 1e18 shares: `_price = IERC4626(vault).convertToAssets(1e18)` on empty vault.
- Exchange rate computed as `1e36 / _price` without min bound; can floor to 0.
- Solvency check multiplies borrower amount by `_exchangeRate` then divides by collateral → 0 → always solvent.

Exploit Steps
- 1) With empty CurveLend vault, donate 2,000 crvUSD to inflate price.
- 2) Deposit 2e18 crvUSD to mint 1 wei of vault shares; `convertToAssets(1e18)` returns 2e36.
- 3) Oracle computes `exchangeRate = 1e36 / 2e36` → 0; `_isSolvent` returns true for any borrow.
- 4) Borrow reUSD up to 10M cap; swap to profit.

Key Code/Storage
- Oracle: `_price = IERC4626(_vault).convertToAssets(1e18); exchangeRate = 1e36 / _price` (floors to 0 on extreme price).
- Solvency: `_ltv = borrowerAmount * exchangeRate / collateral; return _ltv <= maxLTV` → 0 always true when exchangeRate=0.
- Vault: crvUSD-wstUSR pair deployed 2025-06-26 with 10M debt cap; initially empty.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-06/ResupplyFi_exp.sol
- Post-mortem: https://mirror.xyz/0x521CB9b35514E9c8a8a929C890bf1489F63B2C84/ygJ1kh6satW9l_NDBM47V87CfaQbn2q0tWy_rtp76OI
- Alert: https://x.com/peckshield/status/1938061948647817647
- Related pattern: exchange-rate-zero-ltv-bypass.md
