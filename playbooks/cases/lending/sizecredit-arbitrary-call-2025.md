Title: SizeCredit leverageUp arbitrary call via swap data (2025-08-15)

Summary
- Type: unvalidated swap data enables arbitrary external call in `leverageUpWithSwap`.
- Preconditions: `leverageUpWithSwap` accepts `GenericRoute` swap data and executes provided calldata without restricting target/selector; victim has approved PT-wstUSR to leverage contract.
- Impact: attacker crafted swap data to call `transferFrom(victim, attacker, amount)` on PT-wstUSR, stealing ~19.7k USD worth of tokens.
- Representative tx: https://etherscan.io/tx/0xc7477d6a5c63b04d37a39038a28b4cbaa06beb167e390d55ad4a421dbe4067f8.

Signal
- Swap params include arbitrary `method: GenericRoute` with raw `data` executed by contract; no whitelist of targets/selectors.
- Function uses caller-supplied bytes to perform external call without sanity checks; relies on user-provided offsets.
- Large allowances from third-party users to contract exist.

Exploit Steps
- 1. Identify victim with PT-wstUSR allowance to `leverageUp` (0xf4a21ac7e51d17a0e1c8b59f7a98bb7a97806f14).
- 2. Craft `SwapParams` (GenericRoute) data to encode `transferFrom(victim, attacker, amount)`, adjust offset byte to point to payload.
- 3. Call `leverageUpWithSwap` with forged swap data; contract executes transferFrom, moving victim tokens to attacker.

Key Code/Storage
- Contract: 0xf4a21ac7e51d17a0e1c8b59f7a98bb7a97806f14 (`leverageUpWithSwap`).
- Token: PT-wstUSR 0x23E60d1488525bf4685f53b3aa8E676c30321066 approved by victim 0x83eCCb05386B2d10D05e1BaEa8aC89b5B7EA8290.
- Attack contract: 0xa6dc1fc33c03513a762cdf2810f163b9b0fd3a71; attacker EOA 0xa7e9b982b0e19a399bc737ca5346ef0ef12046da.

Refs
- Alert/thread: https://x.com/SuplabsYi/status/1956306748073230785
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main/src/test/2025-08/SizeCredit_exp.sol
