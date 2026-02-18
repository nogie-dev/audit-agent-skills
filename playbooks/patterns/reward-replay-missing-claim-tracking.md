Title: Reward replay due to missing claim tracking

Summary
- Rewards are computed fresh on every call without subtracting previously claimed amounts or updating last-claim checkpoints.
- Impact: repeated claims/withdraws can pay the same full reward multiple times, draining reward pools.

Signal
- `getPendingRewards` sums deposits/rewards with no claimed flag or reward debt; no per-deposit/per-user checkpoint.
- Withdraw/claim callable multiple times without state mutation that reduces future payouts.
- Uses `block.timestamp` directly each call; no accrual difference between calls.

Exploit Steps
- 1. Make a deposit that accrues rewards (possibly after vesting period).
- 2. Once eligible, call claim/withdraw repeatedly to receive the same reward each time.
- 3. Exit/swap rewards to profit.

Key Code/Storage
- Reward calc functions reading deposits/amounts and time elapsed, but not persisting `claimed`, `rewardDebt`, or `lastClaimedAt`.

Fix/Detect
- Track claimed rewards and subtract from pending; add `rewardDebt` or per-deposit claimed flags; update on every claim/withdraw.
- Monitoring: bursts of repeated claims from one address without new deposits; payout per call equals full accrued amount.

Refs
- Example: SOR staking reward replay 2025-01-04 (playbooks/cases/staking/sor-reward-replay-2025.md)
