Title: UNI (SamPrisonman) skim/sync reserve manipulation (2025-03-07)

Summary
- Type: LP reserve/balance desync via skim + sync on fee-on-transfer token.
- Preconditions: token (SamPrisonman) allows zero/fee transfers; pair has token balance ≠ recorded reserves; `skim` and `sync` are public.
- Impact: attacker performed skim, slight token transfer to pair, sync to inflate reserves, then swapped token→WETH to extract ~14K USD.
- Representative tx: https://etherscan.io/tx/0x6c8aed8d0eab29416cd335038cd5ee68c5e27bfb001c9eac7fc14c7075ed4420.

Signal
- UniswapV2 pair with public `skim`/`sync`; token balance can exceed reserves (fee-on-transfer/imbalance).
- Token transfer of 1 unit after skim to create imbalance before sync.

Exploit Steps
- 1. Buy small amount of SamPrisonman token with ETH.
- 2. Call `skim(pair)` to collect excess tokens.
- 3. Transfer 1 token to pair to misalign reserves vs balances.
- 4. Call `sync()` to set reserves to inflated balance.
- 5. Swap all tokens to WETH via router, profiting from distorted reserves.

Key Code/Storage
- Token: SamPrisonman 0xdDF309b8161aca09eA6bBF30Dd7cbD6c474FF700.
- Pair: 0x76EA342BC038d665e8a116392c82552D2605edA1 (UniswapV2-style).
- Router: 0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D.
- Attacker: 0x97d8170e04771826A31C4c9B81e9f9191a1c8613; attack contract 0x2901c8b8e6d9f2c9f848987ded74b776ab1f973e.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/UNI_exp.sol
- Alert: https://x.com/CertiKAlert/status/1897973904653607330
- Related pattern: lp-skim-sync-reserve-desync.md
