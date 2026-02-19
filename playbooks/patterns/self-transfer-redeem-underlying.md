Title: Transfer-to-self redeems underlying 1:1

Summary
- Token implements deposit/mint from underlying, but transfer of the wrapper token to its own contract triggers redemption of underlying at a fixed 1:1 rate.
- Impact: attacker can deposit minimal/flashloaned underlying, mint wrapper, then transfer wrapper to contract to receive underlying back (or more), bypassing fees/logic; used to drain pools.

Signal
- `deposit(token, amount)` minting wrapper; `transfer(wrapper, wrapper)` (recipient == token contract) redeems underlying without checks.
- No fee/slippage on redemption; works even when called by anyone.

Exploit Steps
- 1. Obtain underlying (flashloan).
- 2. `deposit(wrapper, amount)` to mint wrapper tokens.
- 3. `transfer(wrapperContract, amount)` to trigger redemption of underlying at 1:1.
- 4. Swap underlying back, repay loan; keep remainder.

Examples
- playbooks/cases/dex/vds-burn-mint-bypass-2025.md
