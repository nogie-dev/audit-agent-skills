Title: yETH virtual balance unsafe math manipulation (2025-12-01)

Summary
- Type: virtual balance math/rate update manipulation leading to mispriced LP shares and drain.
- Preconditions: attacker can call `update_rates` with crafted indices; virtual balance product/sum uses unchecked math; add/remove liquidity uses virtual balances rather than actual reserves; OETH rebase alters balances mid-cycle.
- Impact: repeated add/remove cycles plus rate updates and rebase skewed pool accounting, allowing withdrawal of excess assets (~$9M loss reported).
- Representative tx: 0x53fe7ef190c34d810c50fb66f0fc65a1ceedc10309cf4b4013d64042a0331156.

Signal
- Pool tracks `virtual_balance`, `vb_prod_sum`, and `supply` with custom math instead of AMM reserves; relies on `update_rates` callable externally.
- Unchecked large multiplications/divisions on virtual balances; no sanity bounds after rate updates or rebases.
- Complex add/remove sequencing can alter virtual balances without corresponding real reserves.

Exploit Steps
- 1. Seed large balances of pool assets and YETH approvals.
- 2. Perform initial remove to obtain YETH, then repeatedly add/remove liquidity with specific amounts to skew virtual balances and `vb_prod_sum`.
- 3. Call `update_rates` with chosen asset indices; perform zero-amount removes to force recalculation on manipulated state.
- 4. Trigger OETH `rebase()` to alter balances, then further add/remove cycles and rate updates.
- 5. Final remove drains pool supply, then minimal add to settle state.

Key Code/Storage
- Pool: 0xCcd04073f4BdC4510927ea9Ba350875C3c65BF81; functions `virtual_balance`, `vb_prod_sum`, `add_liquidity`, `remove_liquidity`, `update_rates`.
- YETH: 0x1BED97CBC3c24A4fb5C069C6E311a967386131f7; OETH rebase token 0x39254033945AA2E4809Cc2977E7087BEE48bd7Ab.
- Vulnerability hinges on virtual balance arithmetic and externally callable rate updates not reflecting real reserves.

Refs
- Analysis: https://github.com/banteg/yeth-exploit/blob/main/report.pdf
- PoC: https://github.com/johnnyonline/yETH-hack
- Attack tx: https://etherscan.io/tx/0x53fe7ef190c34d810c50fb66f0fc65a1ceedc10309cf4b4013d64042a0331156
