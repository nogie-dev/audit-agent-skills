Title: FortuneWheel swapProfitFees ACL missing (2025-01-10)

Summary
- Type: treasury swap function lacks access control; attacker manipulates price then triggers contract swap.
- Preconditions: `swapProfitFees()` external (no onlyOwner); swaps contract-held tokens via Pancake with `amountOutMin=0`; uses live pool price; attacker can pre-manipulate LINK/BNB pool.
- Impact: attacker pumped LINK price with own WBNB, then forced contract to dump its LINK for WBNB at inflated rate, profiting ~21.5K USD.
- Representative tx: https://bscscan.com/tx/0xd6ba15ecf3df9aaae37450df8f79233267af41535793ee1f69c565b50e28f7da.

Signal
- Public treasury swap/harvest function callable by anyone.
- Swaps treasury tokens with `amountOutMin=0` on DEX; no price bounds or delay.
- Uses current AMM price; no TWAP/guardrails; combines multiple token swaps inside one call.

Exploit Steps
- 1. Attacker swaps WBNB→LINK to raise LINK price in the pool.
- 2. Calls `swapProfitFees()` to make the contract swap its LINK to WBNB at the pumped price (minOut=0).
- 3. Swaps back/repays, keeping the spread (~21.5K).

Key Code/Storage
- Vulnerable contract: 0x384b9fb6e42dab87f3023d87ea1575499a69998e; function `swapProfitFees()` uses Pancake router swaps without access control.
- Core swaps: `router.swapExactTokensForETH(amount, 0, [token, WBNB], ...)` and `router.swapExactETHForTokens{value: totalBNBForLink}(0, [WBNB, LINK], ...)`.
- Holds treasury tokens per casino (`tokenIdToCasino`) and aggregates into LINK/BNB/BNBP.

Refs
- Alert: https://x.com/CertiKAlert/status/1877662352834793639
- Blog: https://www.cnblogs.com/ACaiGarden/p/18664999
- Attacker: https://bscscan.com/address/0xf1e73123594cb0f3655d40e4dd6bde41fa8806e8
- Attack contract: https://bscscan.com/address/0xe40ab156440804c3404bb80cbb6b47dddd3abfd7
