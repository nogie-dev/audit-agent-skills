Title: Shared status flag overwritten in multi-action executor

Summary
- Multi-action/cook-style functions share a single status struct across actions. A later action resets flags (e.g., needsSolvencyCheck) set by earlier actions, skipping required validation.
- Impact: borrower can sequence actions to clear solvency/permission flags and bypass safety checks, enabling over-borrow or unauthorized operations.

Signal
- Actions array processed with a shared `status` passed through; no guard against later actions overwriting prior flags.
- Default `_additionalAction`/fallback returns status with flags false/zero.
- Final checks conditioned on a status flag instead of always running.

Exploit Steps
- 1. Include a “set flag” action (e.g., borrow sets needsSolvencyCheck=true).
- 2. Follow with an action that resets/returns default status (e.g., no-op additional action), clearing the flag.
- 3. Final validation is skipped; borrow/operation succeeds without solvency check.

Key Code/Storage
- Shared status struct (`needsSolvencyCheck` or similar) propagated across actions; default handler returns false.
- Final guard `if (status.needsSolvencyCheck) { ... }`.

Examples
- playbooks/cases/stablecoin/mim-spell-cook-bypass-2025.md
