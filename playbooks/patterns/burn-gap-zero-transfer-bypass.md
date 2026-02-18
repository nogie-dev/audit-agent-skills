Title: Burn gap bypass via zero-value transfers

Summary
- Token burn/cooldown logic updates timestamp on any transfer, including amount=0, letting attackers skip burn on next real transfer.
- Impact: repeated zero-value transfers advance or reset burn timer so large swap avoids burn/fee, enabling LP drain.

Signal
- `lastBurnTime`/gap updated unconditionally; burn applied only when gap exceeded; zero transfer allowed.
- Fee-on-transfer token paired in AMM; no check that amount>0 before updating burn state.

Exploit Steps
- 1. Acquire token.
- 2. Loop zero-amount transfers to satisfy/update cooldown.
- 3. Swap out token with burn disabled; drain counter-asset.

Key Code/Storage
- Burn logic with cooldown that runs on any transfer irrespective of amount.

Refs
- Example: BBX burn-gap bypass 2025-03-20 (playbooks/cases/dex/bbx-burn-gap-bypass-2025.md)
