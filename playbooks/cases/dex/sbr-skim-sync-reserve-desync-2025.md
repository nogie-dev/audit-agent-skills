Title: SBR skim/sync reserve desync drain (2025-03-07)

Summary
- Type: LP reserve/balance desync exploited via `skim` + `sync`.
- Preconditions: pair holds more SBR than recorded reserves (fee-on-transfer or prior imbalance); `skim` callable by anyone; `sync` resets reserves to balances; swaps use inflated reserves.
- Impact: attacker used tiny ETH to buy minimal SBR, skimmed excess SBR, nudged reserves with a 1 token transfer, synced to inflate recorded reserves, then swapped SBR→ETH for ~8.5 ETH (~$18.4K).
- Representative tx: https://etherscan.io/tx/0xe4c1aeacf8c93f8e39fe78420ce7a114ecf59dea90047cd2af390b30af54e7b9.

Signal
- Pair token balance > reserves; `skim` publicly callable.
- `sync` publicly callable; no reconciliation or guard against repeated skim/sync.
- Fee-on-transfer or token logic can leave residuals in pair.

Exploit Steps
- 1. Swap dust ETH→SBR to get LP participant status.
- 2. Call `skim(pair)` to pull excess SBR held above reserves.
- 3. Send 1 SBR back to pair to create imbalance; call `sync()` to set reserves to inflated balance.
- 4. Swap skimmed SBR→ETH at inflated reserves; profit.

Key Code/Storage
- Pair: 0x3431c535dDFB6dD5376E5Ded276f91DEaA864FF2 (UniswapV2-style) with public `skim`/`sync`.
- Token: SBR 0x460B1AE257118Ed6F63Ed8489657588a326a206D (fee-on-transfer/imbalance caused extra SBR in pair).

Refs
- Alert: https://x.com/TenArmorAlert/status/1897826817429442652
- Thread: https://x.com/0xfirmanregar/status/2019768070810812520
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/SBRToken_exp.sol
