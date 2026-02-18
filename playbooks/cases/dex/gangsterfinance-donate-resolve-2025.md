Title: Gangsterfinance donate/resolve accounting flaw (2025-06-12)

Summary
- Type: vault accounting miscredit via donate + deposit/resolve/harvest sequence.
- Preconditions: Vault accepts `donate`, `depositTo`, `resolve`, `harvest` publicly; accounting overcredits caller when donating then resolving/harvesting immediately.
- Impact: attacker flash-swapped BTCB, donated, deposited minimal amount, resolved and harvested to withdraw more BTCB than deposited (~16.5k USD).
- Representative tx: https://bscscan.com/tx/0xf34e59e4fe2c9b454d2b73a1a3f3aaf07d484a0c71ff8278b1c068cdedc4b64d.

Signal
- Public donation function that increases pool value; per-user share math fails to exclude caller’s own donation cycle.
- Immediate resolve/harvest after tiny deposit yields outsized payout.

Exploit Steps
- 1) Flashswap ~1.02 BTCB from CAKE LP.
- 2) Approve vault; `donate(1 BTCB)` to boost pool.
- 3) `depositTo(attacker, 0.01572 BTCB)` minimal deposit.
- 4) Call `resolve(myTokens)` then `harvest()` to pull boosted payout.
- 5) Repay flashswap with fee; keep remaining BTCB (~16.5k USD).

Key Code/Storage
- Vault: 0xe968D2E4ADc89609773571301aBeC3399D163c3b (`donate`, `depositTo`, `resolve`, `harvest` public).
- Pair: 0x0b32Ea94DA1F6679b11686eAD47AA4C6bF38cd59 (BTCB/CAKE LP).

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-06/Gangsterfinance_exp.sol
- Alert: (no formal post-mortem; incident tx above)
- Related pattern: overcredited-internal-balance-vs-transfer.md
