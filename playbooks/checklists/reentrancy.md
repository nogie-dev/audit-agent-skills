Title: Reentrancy (single/recursive/cross-function)

Summary
- Typical in external calls before state updates or missing reentrancy guards.
- Preconditions: attacker-controlled callback or ERC777/flash-loan hooks, unprotected external calls, shared state.
- Impact: drain balances, bypass accounting, double-claim rewards, mint inflation.
- Representative: TheDAO, Curve pool reward drains (2022), multiple staking vaults.

Signal
- External calls before state change; mixed view+write functions callable externally.
- Missing/non-canonical reentrancy guard; shared storage across functions called externally.
- Withdraw/mint/redemption paths with callbacks (ERC777 hooks, onFlashLoan, onERC721Received).

Exploit Steps
- 1. Acquire position/token enabling callback (deposit or borrow).
- 2. Trigger function that makes external call before accounting finalization.
- 3. Callback re-enters sibling function or same function to repeat effect until cap hit.

Key Code/Storage
- Functions doing `call`/`delegatecall`/token transfers pre-state update.
- Shared balances mapping, debt accounting, reward indices, cumulative sums.
- Non-atomic math on shares/price-per-share during callbacks.

Refs
- https://rekt.news/daorekt/
- https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/
