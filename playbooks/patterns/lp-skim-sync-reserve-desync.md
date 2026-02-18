Title: LP reserve desync via skim/sync on imbalanced token

Summary
- UniswapV2-style pair holds more tokens than recorded reserves (fee-on-transfer/residuals); public `skim`/`sync` let attackers reset reserves at a distorted balance.

Signal
- Public `skim(address)` and `sync()` on pair; token balance can exceed reserves (fee-on-transfer, prior residuals).
- Small token transfer after `skim` before `sync` to misalign reserves.

Exploit Steps
- 1. Acquire token (dust swap).
- 2. `skim(pair)` to collect excess/residual tokens.
- 3. Send minimal token back to pair; call `sync()` to set reserves to inflated balance.
- 4. Swap token→base asset at distorted reserves to drain counter-asset.

Key Code/Storage
- UniswapV2 pair with fee/residual-prone token; public skim/sync; reserves derived from balances on sync.

Refs
- Example: UNI skim/sync reserve manipulation 2025-03-07 (playbooks/cases/dex/uni-skim-sync-reserve-2025.md)
