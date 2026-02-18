Title: Storage slot collision combined with callback abuse

Summary
- Critical address/param stored in low storage slot from user-controlled value; attacker collides slot with CREATE2 deployment and then abuses callbacks that trust calldata to move funds.
- Impact: overwrite trusted address, deploy contract at that address, and route swap callbacks or transfer logic to attacker-controlled code to drain assets.

Signal
- `tstore`/storage write of important address/amount directly from user input (e.g., mint/deposit).
- Callback (e.g., `uniswapV3SwapCallback`) uses calldata `data` addresses for transfers without validation.
- CREATE2 used to deploy at predictable address equal to corrupted slot.

Exploit Steps
- 1. Write crafted value to targeted slot via exposed function.
- 2. Deploy contract at collided address using CREATE2.
- 3. Trigger callback that reads slot/callback data, directing asset transfers to attacker contract.

Key Code/Storage
- User-controlled storage writes; callbacks that trust calldata for transfer routing; CREATE2 predictable address collision.

Refs
- Example: LeverageSIR slot collision & callback abuse 2025-03-30 (playbooks/cases/lending/leveragesir-slot-collision-2025.md)
