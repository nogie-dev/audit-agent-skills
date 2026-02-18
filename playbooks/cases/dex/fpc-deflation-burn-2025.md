Title: FPC burn-on-transfer drains USDT (2025-07-05)

Summary
- Type: deflationary transfer breaks AMM pricing; outputs overpaid.
- Preconditions: FPC burns tokens on transfer to the pair; swap math assumes full amount in, so effective amountIn < expected, inflating price and causing overpayment on swapExactTokensSupportingFee.
- Impact: attacker flash-borrowed USDT, bought FPC, then sold FPC back for more USDT (~4.7M USDT stolen).
- Representative tx: https://bscscan.com/tx/0x3a9dd216fb6314c013fa8c4f85bfbbe0ed0a73209f54c57c1aab02ba989f5937.

Signal
- Token has burn/fee on transfer that triggers when sending to the LP.
- swapExactTokensSupportingFeeOnTransferTokens used; reserves not synced to burned amount.
- Large price discrepancy between getAmountsOut and actual post-burn balances.

Exploit Steps
- 1) Flash-loan 23.02M USDT from Pancake V3 pool.
- 2) Swap USDT→FPC via Pancake pair.
- 3) Transfer ~247k FPC to helper; helper swaps FPC→USDT with `swapExactTokensSupportingFeeOnTransferTokens`, triggering burn on transfer to pair and overpaying USDT.
- 4) Repay flash loan + fee; keep remaining USDT (~4.7M).

Key Code/Storage
- Token: FPC 0xB192D4A737430AA61CEA4Ce9bFb6432f7D42592F (burn on transfer to pair).
- Pair: 0xa1e08E10Eb09857A8C6F2Ef6CCA297c1a081eD6B (Pancake V2).
- Flash pool: 0x92b7807bF19b7DDdf89b706143896d05228f3121 (Pancake V3).

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/FPC_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1940423393880244327
- Related pattern: buggy-transfer-lp-skim-overpay.md
