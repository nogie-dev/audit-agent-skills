Title: BankrollStack buy/sell/withdraw overcredit (2025-06-10)

Summary
- Type: dividend/accounting flaw; buy/sell cycle overcredits balances.
- Preconditions: `buy`, `sell`, `withdraw` all public; dividend/share accounting lets a flash-loan-funded buy/sell in same block extract dividends.
- Impact: attacker flash-borrowed BUSD, bought BankrollStack tokens, immediately sold and withdrew, repaid loan and kept ~5k USD.
- Representative tx: https://bscscan.com/tx/0x0706425beba4b3f28d5a8af8be26287aa412d076828ec73d8003445c087af5fd.

Signal
- No lockup between buy and dividend withdrawal; donations/fees redistributed instantly.
- Public withdraw callable after immediate buy/sell round-trip.

Exploit Steps
- 1) Flashloan ~28.3k BUSD from Pancake V3 pool.
- 2) Approve BankrollStack; `buy` with full amount.
- 3) `sell(myTokens)` to get BUSD back.
- 4) `withdraw()` to collect dividends/fees credited during round-trip.
- 5) Repay flashloan + fee; keep remainder (~5k USD).

Key Code/Storage
- BankrollStack: 0x16d0a151297a0393915239373897bCc955882110 (`buy`, `sell`, `withdraw` public).
- Flash pool: 0x4f3126d5DE26413AbDCF6948943FB9D0847d9818 (Pancake V3).
- Token accounting via `myTokens`, `myDividends`.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-06/BankrollStack_exp.sol
- Alerts: https://x.com/Phalcon_xyz/status/1943518566831296566 , https://x.com/TenArmorAlert/status/1935618109802459464
- Related pattern: overcredited-internal-balance-vs-transfer.md
