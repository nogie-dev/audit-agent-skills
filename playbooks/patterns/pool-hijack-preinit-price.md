Title: Pool hijack via pre-initialization with extreme price

Summary
- AMM pool is created by whoever calls `createAndInitializePoolIfNecessary` first; project later adds liquidity to existing pool without verifying creator or initial price.
- Impact: attacker pre-creates pool with extreme `sqrtPrice`, project migrates liquidity into hijacked pool, enabling attacker to trade small holdings for outsized base asset.

Signal
- Liquidity migration uses `createAndInitializePoolIfNecessary` and trusts existing pool address without checking origin/salt/price.
- No guard that project-controlled pool exists before migration; trading restrictions lifted during migration.

Exploit Steps
- 1. Acquire small amount of project token (from presale/launchpad).
- 2. Front-run and call pool creation with extreme price.
- 3. Let project add liquidity into this pool.
- 4. Swap small token amount for large base asset at skewed price.

Key Code/Storage
- Pool creation via router factory calls; `addLiquidity` accepts existing pool without ownership/price validation.

Refs
- Example: Four.meme pool hijack 2025-02-11 (playbooks/cases/dex/fourmeme-pool-hijack-price-skew-2025.md)
