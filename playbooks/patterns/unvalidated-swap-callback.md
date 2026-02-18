Title: Unvalidated swap/callback data allows arbitrary calls

Summary
- Protocol executes user-supplied swap data (e.g., GenericRoute) without restricting target/selector, letting attackers call arbitrary contracts.
- Impact: with existing allowances, attacker can `transferFrom` victim assets or invoke arbitrary functions during leverage/swap flows.

Signal
- Swap params include raw `bytes data` executed as-is; no whitelist of routers/targets.
- Calldata offsets manually tweaked by user; contract trusts caller-provided lengths/targets.
- Function mixes asset movement with arbitrary external call hooks.

Exploit Steps
- 1. Find victim with allowance to the swap/leverage contract.
- 2. Craft swap data to call `transferFrom(victim, attacker, amount)` (or other arbitrary call), adjust offsets if needed.
- 3. Invoke protocol function; contract executes malicious call and transfers victim funds.

Key Code/Storage
- Functions accepting `SwapParams`/`GenericRoute` and dispatching external calls without target/selector validation.

Refs
- Example: SizeCredit leverageUp arbitrary call (playbooks/cases/lending/sizecredit-arbitrary-call-2025.md)
