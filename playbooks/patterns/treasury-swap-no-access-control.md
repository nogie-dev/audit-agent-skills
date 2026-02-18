Title: Treasury swap/harvest without access control

Summary
- Contract-owned treasury tokens are swapped on DEXes via public functions; no access control or price protection.
- Impact: attacker manipulates pool price, then triggers treasury swap at manipulated rate to extract value; risk amplified by `amountOutMin=0`.

Signal
- Functions like `swapProfitFees` callable by anyone; no onlyOwner/keeper.
- Uses AMM spot price with `amountOutMin=0`; multi-token loops swapping treasury balances.
- No TWAP/bounds; swaps large treasuries in a single call.

Exploit Steps
- 1. Pre-manipulate pool price (buy or sell target token).
- 2. Call public treasury swap/harvest to force contract to trade at manipulated price.
- 3. Unwind manipulation and keep the spread.

Key Code/Storage
- Public swap function calling `swapExactTokensForETH` / `swapExactETHForTokens` with minOut=0 on contract-held balances.

Refs
- Example: FortuneWheel swapProfitFees ACL missing (playbooks/cases/token/fortune-swapprofitfees-acl-2025.md)
- Example: Unwarp WETH unwrap without auth (playbooks/cases/dex/unwarp-access-2025.md)
- Example: MetaPool unrestricted mint with zero assets (playbooks/cases/dex/metapool-mint-no-asset-2025.md)
