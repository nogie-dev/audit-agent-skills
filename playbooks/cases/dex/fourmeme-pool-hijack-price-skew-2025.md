Title: Four.meme pool hijack and price skew (2025-02-11)

Summary
- Type: liquidity migration to attacker-created pool with extreme initial price.
- Preconditions: launchpad creates Pancake pool via `createAndInitializePoolIfNecessary`; attacker can front-run and create the pool first with arbitrarily high `sqrtPriceX96`; launchpad later adds liquidity into attacker’s pool (no auth on pool address choice); trading restrictions lifted during migration.
- Impact: attacker seeded pool at extreme price, launchpad added liquidity into hijacked pool, attacker swapped tiny meme balance for large WBNB (~287 BNB ≈ $186K). Similar method used on ~20 tokens.
- Representative tx: https://bscscan.com/tx/0x2902f93a0e0e32893b6d5c907ee7bb5dabc459093efa6dbc6e6ba49f85c27f61.

Signal
- Pool creation uses `createAndInitializePoolIfNecessary` without verifying caller/expected pool salt; relies on first-come pool creation.
- Liquidity addition accepts existing pool address without validation; no safeguard against hijacked pool with manipulated price.
- Trading lock lifted during migration; attacker can front-run pool creation.

Exploit Steps
- 1. Buy small amount of meme token from launchpad (only source initially).
- 2. Attacker calls `createAndInitializePoolIfNecessary` first with extreme `sqrtPriceX96` to create Pancake pool.
- 3. When launchpad migrates liquidity, it adds into attacker-created pool.
- 4. Swap small meme amount for outsized WBNB due to skewed price; repeat across tokens.

Key Code/Storage
- Launchpad: 0x5c952063c7fc8610FFDB798152D69F0B9550762b (Four meme) — `addLiquidity` uses pool created by `createAndInitializePoolIfNecessary` without verifying ownership/price.
- Attacker pool: created via Pancake router at 0xa610cC0d657bbFe78c9D1eA638147984B2F3C05c with extreme sqrtPrice.

Refs
- Analysis: https://securrtech.medium.com/the-four-meme-exploit-a-deep-dive-into-the-183-000-hack-6f45369029be
- Coverage: https://www.chaincatcher.com/en/article/2167296
- Attacker addresses: see case notes (multiple EOAs and contracts involved)
- Attack tx: https://bscscan.com/tx/0x2902f93a0e0e32893b6d5c907ee7bb5dabc459093efa6dbc6e6ba49f85c27f61
- Related pattern: pool-hijack-preinit-price.md
