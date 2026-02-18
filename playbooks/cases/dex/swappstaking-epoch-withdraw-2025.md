Title: SWAPP Staking epoch init + emergencyWithdraw drains cUSDC (2025-07-15)

Summary
- Type: staking accounting flaw; attacker can withdraw contract-held cUSDC without depositing.
- Preconditions: `manualEpochInit` callable; `deposit` trusts caller’s `amount` and credits balance after epoch init; `emergencyWithdraw` returns staked amount without verifying actual contribution beyond internal accounting.
- Impact: attacker initialized epochs, “deposited” an amount equal to staking contract’s own cUSDC balance, then called `emergencyWithdraw` to pull those tokens (~32k USD).
- Representative tx: https://app.blocksec.com/explorer/tx/eth/0xa02b159fb438c8f0fb2a8d90bc70d8b2273d06b55920b26f637cab072b7a0e3e.

Signal
- Epoch init is public; no guard to prevent attackers from initializing past epochs.
- `deposit` uses caller-supplied amount and only checks epoch init; no invariant tying user balance to net inflow (can use contract’s existing token balance).
- `emergencyWithdraw` returns recorded balance without additional checks.

Exploit Steps
- 1) Call `manualEpochInit` for all epochs on cUSDC to satisfy deposit precondition.
- 2) Read staking’s current cUSDC balance; call `deposit(cUSDC, stakingBalance, 0)` after approving (caller may have zero actual cUSDC).
- 3) Call `emergencyWithdraw(cUSDC)` to transfer the recorded balance from staking to attacker.

Key Code/Storage
- Staking: 0x245a551ee0F55005e510B239c917fA34b41B3461 (`manualEpochInit`, `deposit`, `emergencyWithdraw` public).
- Token: cUSDC 0x39AA39c021dfbaE8faC545936693aC917d5E7563.
- Exploit relies on staking’s pre-existing cUSDC holdings being withdrawable.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/SWAPPStaking_exp.sol
- Alert: https://x.com/deeberiroz/status/1947213692220710950
- Related pattern: overcredited-internal-balance-vs-transfer.md
