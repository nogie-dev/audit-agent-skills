Title: ABCCApp fixedDay no auth boosts claim (2025-08-18)

Summary
- Type: missing access control on reward accrual parameter.
- Preconditions: `addFixedDay(uint256)` is public; `fixedDay` is used in claimable USDT/DDDD calculation; contract holds USDT/DDD balances; flashloan available.
- Impact: attacker flash-loaned BUSD, deposited small amount, set `fixedDay` to huge value, then claimed DDDD rewards worth ~10k BUSD and swapped to BUSD.
- Representative tx: https://bscscan.com/tx/0xee4eae6f70a6894c09fda645fb24ab841e9847a788b1b2e8cb9cc50c1866fb12.

Signal
- Reward parameter setter (addFixedDay) callable by anyone; no onlyOwner/role check.
- Claim uses `fixedDay` directly in payout computation.

Exploit Steps
- 1) Flash-loan ~12.5k BUSD from proxy.
- 2) Approve and call `deposit` with small number to enable claiming.
- 3) Call `addFixedDay(1_000_000_000)` to inflate accrual.
- 4) Call `claimDDDD()` to receive DDDD tokens.
- 5) Swap DDDD→WBNB→BUSD via router; repay flash loan; keep profit (~10k BUSD).

Key Code/Storage
- ABCCApp token/contract: 0x1bC016C00F8d603c41A582d5Da745905B9D034e5 (`addFixedDay` public).
- DDDD token: 0x422cBee1289AAE4422eDD8fF56F6578701Bb2878.
- Flashloan: IERC1967Proxy 0x8F73b65B4caAf64FBA2aF91cC5D4a2A1318E5D8C.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-08/ABCCApp_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1959457212914352530
- Related pattern: treasury-swap-no-access-control.md
