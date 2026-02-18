Title: Mosca join overcredit and withdraw drain (2025-01-13)

Summary
- Type: internal balance overcredit due to `diff` reuse and gross amount math, not tied to actual transfers.
- Preconditions: `join` computes `baseAmount = ((amount+diff)*1000)/1015` where `diff = max(user.balance - 127e18, 0)`; credits `user.balance` by `baseAmount - fee` while only `amount - TAX*3` (or similar) is transferred; no reserve check; repeated joins allowed.
- Impact: attacker looped `join` with flashloaned BUSD/USDC, compounding `diff`-based overcredit to grow `user.balance`, then `withdrawFiat` drained stables (~37.6K loss).
- Representative tx: https://bscscan.com/tx/0xf13d281d4aa95f1aca457bd17f2531581b0ce918c90905d65934c9e67f6ae0ec.

Signal
- `diff = user.balance - 127e18` added into next `baseAmount` calculation; overcredit snowballs across calls.
- Balance credits use `(amount+diff)` while net transfer subtracts fees; balance not reconciled to reserves.
- Mixed fiat paths (`fiat` flag) share balances; `withdrawFiat` honors internal balance only.

Exploit Steps
- 1. Flashloan BUSD/USDC.
- 2. Call `join` repeatedly; each call adds prior `diff` to `amount` in `baseAmount` calc, inflating `user.balance` beyond deposits.
- 3. Call `withdrawFiat` to pull large BUSD/USDC amounts based on inflated balance.
- 4. Repay flashloan; keep the spread.

Key Code/Storage
- Contract: 0xd8791f0c10b831b605c5d48959eb763b266940b9 (Mosca).
- `join`: credits `user.balance += baseAmount - JOIN_FEE` (or enterprise variant) before or after fees; transfers only `amount - TAX*3` to contract; uses `diff`/constants in 1e18 units.
- `withdrawFiat`: allows withdrawing against overcredited balance.

Refs
- Alert: https://x.com/TenArmorAlert/status/1878699517450883407
- Attacker: https://bscscan.com/address/0xe763da20e25103da8e6afa84b6297f87de557419
- Attack contract: https://bscscan.com/address/0xedcfa34e275120e7d18edbbb0a6171d8ad3ccf54
