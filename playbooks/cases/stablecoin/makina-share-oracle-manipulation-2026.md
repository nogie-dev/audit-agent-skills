Title: Makina Share Oracle Manipulation (2026-05-18)

Summary
- Type: permissionless AUM refresh used as price anchor (no delay/TWAP/bounds), consumed in same transaction by external pool.
- Preconditions: anyone can call `updateTotalAum()`; AUM uses live pool balances; share price read immediately by Curve DUSD/USDC pool; no rate limits.
- Impact: AUM/price inflated mid-tx, DUSD swapped for excess USDC; pool loss ~5.1M USDC, attacker profit ~3.0M USDC (flashloan-assisted).
- Representative tx: 0x569733b8016ef9418f0b6bde8c14224d9e759e79301499908ecbcd956a0651f5.

Signal
- AUM/price anchor update callable by anyone, no time delay or TWAP.
- Share price changes significantly within one tx/block and is consumed by swaps/LP withdraws in same tx.
- Downstream pools (Curve, LP) rely on anchor without bounds/circuit breakers.

Exploit Steps
- 1. Flashloan ~280M USDC and skew pools (DUSD/USDC, 3pool, MIM-3pool) to inflate observed AUM.
- 2. Call `updateTotalAum()` (permissionless) to bake manipulated balances into `lastTotalAum` and share price.
- 3. Immediately swap DUSD→USDC in Curve DUSD/USDC pool that uses the inflated share price.
- 4. Repay flashloans and keep the spread (~3.0M USDC profit; ~5.1M pool loss reported).

Key Code/Storage
- Machine (AUM writer): 0x6b006870C83b1Cd49E766Ac9209f8d68763Df721 — `updateTotalAum()` updates `lastTotalAum` permissionlessly.
- Share oracle: 0xFFCBc7A7eEF2796C277095C66067aC749f4cA078 — `getSharePrice()` reads updated AUM directly (no delay/TWAP/bounds).
- Consumer: Curve DUSD/USDC pool 0x32E616F4f17d43f9A5cd9Be0e294727187064cb3 pricing off the share oracle.

Refs
- Attack tx: https://skylens.certik.com/tx/eth/0x569733b8016ef9418f0b6bde8c14224d9e759e79301499908ecbcd956a0651f5
- Attacker: 0x935bfb495e33f74d2e9735df1da66ace442ede48
- Post-mortem: https://x.com/nn0b0dyyy/status/2013472538832314630
- Alerts: https://x.com/TenArmorAlert/status/2013460083078836342 , https://x.com/CertiKAlert/status/2013473512116363734
