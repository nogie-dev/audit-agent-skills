Title: IPC sell timelock bypass via flawed LP detection (2025-01)

Summary
- Type: anti-bot sell timelock bypass; sell path also burns pair tokens.
- Preconditions: buy sets a 30m sell lock via `transferTime`, but `_isRemoveLP` mis-detects buys when both tokens are flash-swapped; sell burns pair tokens.
- Impact: attacker bypassed timelock, sold immediately, and repeated swaps while burning pair-side tokens, draining ~590k USDT from liquidity.
- Representative txs: front-run 0x5ef1edb9749af6cec511741225e6d47103e0b647d1e41e08649caaff66942a91 (BSC); victim 0x3a3683119e1801821faa15c319cb9c8fb3fcf6ee92b1904a829d82c432e09a44.

Signal
- Transfer hook distinguishes buy/sell via `_isRemoveLP`/`_isAddLP`; uses reserve comparison to detect liquidity removal but can be fooled by flash swaps.
- Timelock (`TRANSFER_LOCK`) enforced only when buy branch runs; buy branch can be skipped if `_isRemoveLP` returns true.
- Sell path burns pair tokens (`_destroy(destroyNum)`), further draining paired asset.

Exploit Steps
- 1. Flash swap both tokens from the pair so `_isRemoveLP(pair)` returns true.
- 2. Perform buy without triggering timelock (buy branch skipped), so `transferTime` not set.
- 3. Immediately sell; sell path burns pair tokens and transfers out USDT.
- 4. Loop swaps/flashloans to maximize drain.

Key Code/Storage
- `_isRemoveLP` compares pair reserves vs balances; flawed logic using `USDT > address(this)` to pick reserve index, misclassifies flash swaps as liquidity removal.
- `_transfer` buy path sets `transferTime[recipient]` only if `sender == pair && !_isRemoveLP(pair)`.
- Sell path requires `block.timestamp >= transferTime[sender] + TRANSFER_LOCK`; if buy path skipped, lock is unset and sell passes.
- Pair: 0xDe3595a72f35d587e96d5C7B6f3E6C02ed2900AB (IPC/USDT on BSC). Token: 0xEAb0d46682Ac707A06aEFB0aC72a91a3Fd6Fe5d1.

Fix/Detect
- Correct LP add/remove detection; avoid reserve/balance heuristic; track per-address buy timestamps unconditionally on pair-origin transfers.
- Do not burn pair balances; avoid custom AMM mutations.
- Monitoring: buys without timelock entries, immediate sells after buys, flash swaps around buys, pair balance burns.

Refs
- Alert: https://x.com/TenArmorAlert/status/1876669657866015230
- Attack tx (front-run): https://app.blocksec.com/explorer/tx/bsc/0x5ef1edb9749af6cec511741225e6d47103e0b647d1e41e08649caaff66942a91
- Victim tx: https://app.blocksec.com/explorer/tx/bsc/0x3a3683119e1801821faa15c319cb9c8fb3fcf6ee92b1904a829d82c432e09a44
