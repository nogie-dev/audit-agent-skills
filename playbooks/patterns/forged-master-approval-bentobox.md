Title: Bento-style masterContractApproval without signature check

Summary
- Contracts modeled after BentoBox expose `setMasterContractApproval(user, master, approved, v, r, s)` to authorize helpers.
- If signature validation is missing or accepts zeroed sigs, anyone can approve themselves for any user and withdraw that user’s funds.

Signal
- `setMasterContractApproval` callable by anyone; no `msg.sender == user` guard and no ECDSA recovery/verify.
- Zeroed or arbitrary `v/r/s` accepted.
- Subsequent `withdraw(token, from=user, to=attacker, amount, share)` succeeds immediately after forged approval.

Exploit Steps
- Call `registerProtocol()` if required, then `setMasterContractApproval(user=<victim>, master=<self>, approved=true, v/r/s=0)`.
- Call `withdraw` with `from=victim`, `to=attacker` to drain balances.

Key Code/Storage
- Approval function lacking signature verification.
- Withdraw function trusting `masterContractApproval[user][master]`.

Refs
- RICE masterContractApproval bypass (2025-05-24): forged approval with zeroed sigs, then `withdraw` drained victim RICE, swapped to WETH.
