Title: Internal balance overcredited vs net transfer

Summary
- Protocol credits user balance from nominal `amount` plus carry-over terms (e.g., `diff = balance - threshold`) while net transfer subtracts fees; no reconciliation between credited balance and actual reserves.
- Impact: repeated deposits mint compounding internal balance exceeding tokens received, enabling withdrawals beyond real holdings (esp. across multi-fiat paths).

Signal
- `balance += f(amount + diff)` (diff derived from existing balance) while transfers use `amount - fee`; no invariant tying balance to reserves.
- Shared balance across multiple token types/fiat flags; withdraw uses internal balance only.
- Uses hardcoded 1e18 constants with tokens of varying decimals.

Exploit Steps
- 1. Deposit/“join” repeatedly to inflate internal balance.
- 2. Withdraw using functions that honor internal balance without checking reserves.
- 3. Use flashloans to front liquidity if needed; repay after withdrawal.

Key Code/Storage
- Balance mutations based on gross input; transferFrom subtracts fee; withdraw uses credited balance.

Refs
- Example: Mosca join overcredit (playbooks/cases/staking/mosca-overcredited-join-2025.md)
