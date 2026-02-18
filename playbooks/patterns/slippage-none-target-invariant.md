Title: Missing slippage allows invariant-targeted swaps

Summary
- Swaps executed without `amountOutMin`/slippage checks; attacker computes outputs to target the constant product and drain value.

Signal
- Swap paths/calls accept zero or lax `amountOutMin`; protocol uses AMM invariant but caller chooses outputs.
- Flashloan + iterative swaps targeting reserve math.

Exploit Steps
- 1. Flashloan base asset.
- 2. Split into sub-swaps; compute out amounts to push K toward target without slippage enforcement.
- 3. Swap back to base asset, extracting value; repay loan.

Key Code/Storage
- Swap functions lacking slippage/minOut validation; AMM reserves accessible for custom calculations.

Refs
- Example: YBToken no-slippage protection 2025-04-16 (playbooks/cases/dex/ybtoken-no-slippage-2025.md)
