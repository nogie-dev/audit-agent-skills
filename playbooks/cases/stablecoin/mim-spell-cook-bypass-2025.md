Title: MIM Spell cook insolvency check bypass (2025-10-05)

Summary
- Type: multi-action executor resets solvency flag.
- Preconditions: `cook(uint8[] actions, bytes[] data)` runs multiple actions sharing a single `status` struct; action 5 (borrow) sets `status.needsSolvencyCheck = true`; action 0 calls `_additionalCookAction` which returns `status.needsSolvencyCheck = false` by default, overwriting prior flag.
- Impact: attacker called `cook` with actions [5 (borrow), 0] on multiple addresses, clearing the solvency flag and borrowing 1,793,755e18 MIM without checks (~$1.7M loss).
- Representative tx: https://app.blocksec.com/explorer/tx/eth/0x842aae91c89a9e5043e64af34f53dc66daf0f033ad8afbf35ef0c93f99a9e5e6.

Signal
- Multi-action executor sharing a status struct; later actions can overwrite flags set by earlier actions.
- `_additionalCookAction` unimplemented/default returns false for `needsSolvencyCheck`.
- Final solvency check guarded by `status.needsSolvencyCheck`.

Exploit Steps
- 1) Prepare `actions = [5 (ACTION_BORROW), 0]`.
- 2) Call `cook` on MIM Spell with these actions; borrow sets flag, additional action clears it.
- 3) Final solvency check skipped; receive MIM.
- 4) Swap/bridge profits; repeat across 6 addresses.

Key Code/Storage
- Function: `cook(actions, data)` shares `status`; `ACTION_BORROW` sets `needsSolvencyCheck = true`.
- `_additionalCookAction` (action 0) returns default status with `needsSolvencyCheck = false`.
- Final check only if `status.needsSolvencyCheck`.

Refs
- Alert: https://x.com/Phalcon_xyz/status/1958062149881540823
- Attack tx: https://app.blocksec.com/explorer/tx/eth/0x842aae91c89a9e5043e64af34f53dc66daf0f033ad8afbf35ef0c93f99a9e5e6
- Related pattern: shared-status-flag-overwrite.md
