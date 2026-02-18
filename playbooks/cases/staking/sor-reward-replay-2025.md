Title: SOR staking reward replay (2025-01-04)

Summary
- Type: reward calculation without claim checkpointing (repeatable payouts).
- Preconditions: deposits mature (vesting tier met); `getPendingRewards` sums rewards per deposit every call; withdraw claims rewards without marking as claimed or updating last-claim.
- Impact: attacker withdrew rewards hundreds of times from one matured deposit, draining ~8 ETH equivalent (4.8 + 2.4 + 0.8 ETH reported).
- Representative tx: deposit 0x72a252277e30ea6a37d2dc9905c280f3bc389b87f72b81a59aa8f50baebd8eaa; attack 0x6439d63cc57fb68a32ea8ffd8f02496e8abad67292be94904c0b47a4d14ce90d.

Signal
- `getPendingRewards` loops deposits and returns full reward if `timeElapsed >= vestingTime` with no per-deposit claimed flag or lastClaim timestamp.
- Withdraw/claim functions callable repeatedly with no state update to prevent double-claiming.
- Uses `block.timestamp` each call; no decay or accrual tracking across calls.

Exploit Steps
- 1. Deposit SOR once into staking contract.
- 2. Wait until vesting period ends (e.g., 14 days).
- 3. Call withdraw/claim many times (e.g., 800) to receive the same full reward each call because claimed state is never updated.
- 4. Swap drained SOR to ETH.

Key Code/Storage
- `getPendingRewards` / `_calculateRewards` sum reward per deposit when vesting met; no claimed tracking.
- Positions store `deposits[]` with `depositTime`, `tier`, `rewardBps`, but no claimed amount or last claim timestamp.
- Vulnerable staking: 0x5d16b8Ba2a9a4ECA6126635a6FFbF05b52727d50; token SOR: 0xE021bAa5b70C62A9ab2468490D3f8ce0AfDd88dF.

Fix/Detect
- Track claimed/redeemed rewards per deposit or per position (e.g., `rewardDebt`, `lastClaimedAt`, claimed flags) and update on every claim/withdraw.
- Make reward functions idempotent; cap claimable to accrued minus claimed; consider non-reentrant claim flow.
- Monitoring: repeated reward claims from same wallet without new deposits; high-frequency withdraw/claim after vesting.

Refs
- Alert: https://x.com/TenArmorAlert/status/1875582709512188394
- Attack tx: https://app.blocksec.com/explorer/tx/eth/0x6439d63cc57fb68a32ea8ffd8f02496e8abad67292be94904c0b47a4d14ce90d
- Deposit tx: https://app.blocksec.com/explorer/tx/eth/0x72a252277e30ea6a37d2dc9905c280f3bc389b87f72b81a59aa8f50baebd8eaa
