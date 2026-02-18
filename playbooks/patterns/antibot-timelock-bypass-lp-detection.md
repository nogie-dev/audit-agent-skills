Title: Anti-bot timelock bypass via flawed LP detection

Summary
- Buy/sell detection relies on reserve/balance heuristics (e.g., `_isRemoveLP`) and only sets timelock on buys; attackers can spoof liquidity removal to skip the lock.
- Impact: immediate sells despite intended lock; if sell path burns pair tokens or adds extra fees, pool can be drained quickly.

Signal
- `_isRemoveLP`/`_isAddLP` compares reserves vs balances to classify LP events; relies on token ordering checks like `USDT > address(this)`.
- Timelock applied only when `sender == pair && !_isRemoveLP(pair)`; no default `transferTime` set for all pair-origin transfers.
- Sell path includes destructive actions (burn pair balances) or high fees.

Exploit Steps
- 1. Flash swap both tokens to perturb reserves so `_isRemoveLP` returns true.
- 2. Buy without triggering timelock (buy branch skipped).
- 3. Immediately sell; timelock check bypassed; destructive sell logic executes.

Key Code/Storage
- Reserve/balance-based LP detection, e.g., `_isRemoveLP(pair)` that infers removal from `balanceOf(pair) < reserve` with token-order assumptions.
- Timelock state `transferTime[addr]` only set in buy branch.

Refs
- Example: IPC timelock bypass 2025-01 (playbooks/cases/token/ipc-timelock-bypass-2025.md)
