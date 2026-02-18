Title: BankrollNetwork donate/buy/sell dividend overcredit (2025-06-10)

Summary
- Type: dividend/accounting flaw allowing double collection.
- Preconditions: `donatePool`, `buy`, `sell`, `withdraw` all public; donations boost dividends without proper exclusion of caller’s own deposit/withdraw cycle; flashswap WBNB available.
- Impact: attacker flash-swapped WBNB, donated to pool, bought tokens, then sold and withdrew dividends, repaying loan and keeping ~24.5 WBNB (~$12k+).
- Representative tx: https://bscscan.com/tx/0x7226b3947c7e8651982e5bd777bca52d03ea31d19b515dec123595a4435ae22c.

Signal
- Dividend/share accounting not adjusting for immediate buy/sell within same window.
- Public donation function affecting all holders including attacker’s just-bought balance.
- No lockup/fee preventing instant round-trip.

Exploit Steps
- 1) Flashswap 2,000 WBNB from Pancake pair.
- 2) `donatePool(1000 WBNB)` to inflate dividends.
- 3) `buy(240 WBNB)` to acquire tokens after donation.
- 4) `sell(all)` to cash out; top-up tiny amount to avoid revert; `withdraw()` dividends.
- 5) Repay flashswap; keep profit.

Key Code/Storage
- BankrollNetwork: 0xAdEfb902CaB716B8043c5231ae9A50b8b4eE7c4e (`donatePool`, `buy`, `sell`, `withdraw` public).
- Pair: 0x16b9a82891338f9bA80E2D6970FddA79D1eb0daE (WBNB pair).
- Token/share tracked via `myTokens`, `myDividends`.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-06/BankrollNetwork_exp.sol
- Alerts: https://x.com/Phalcon_xyz/status/1943518566831296566 , https://x.com/TenArmorAlert/status/1935618109802459464
- Related pattern: overcredited-internal-balance-vs-transfer.md
