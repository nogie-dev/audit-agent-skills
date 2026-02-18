Title: Lifeprotocol buy/sell loop drains BUSD (2025-04-26)

Summary
- Type: price/fee manipulation via repeated buy/sell without invariant protection.
- Preconditions: `buy/sell` callable repeatedly; pricing/fees allow profit per cycle; flashloanable BUSD.
- Impact: attacker flashloaned 110k BUSD, looped buy/sell 53 times, drained ~15,114 BUSD.
- Representative tx: https://bscscan.com/tx/0x487fb71e3d2574e747c67a45971ec3966d275d0069d4f9da6d43901401f8f3c0.

Signal
- Public buy/sell functions lacking slippage/invariant checks; per-cycle net positive due to fee logic.
- Flashloan not restricted; repeated cycles possible in one tx.

Exploit Steps
- 1. Flashloan BUSD from DPP (0x6098…B476).
- 2. Loop `buy(1000*1e18)` 53 times, then `sell(1000*1e18)` 53 times to extract spread.
- 3. Repay flashloan; keep profit.

Key Code/Storage
- Contract: LifeProtocol 0x42e2773508e2AE8fF9434BEA599812e28449e2Cd; token: LIFE 0x19B2834f99Fb9eB4164CB5b49046Ec207F894197.
- Flashloan pool: 0x6098A5638d8D7e9Ed2f952d35B2b67c34EC6B476.
- Attacker: 0x3026C464d3Bd6Ef0CeD0D49e80f171b58176Ce32; contract: 0xF6Cee497DFE95A04FAa26F3138F9244a4d92f942.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-04/Lifeprotocol_exp.sol
