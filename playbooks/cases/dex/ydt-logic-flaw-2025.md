Title: YDT backdoor transfer drains LP (2025-05-26)

Summary
- Type: logic/backdoor function callable by anyone to move LP funds.
- Preconditions: YDT token exposes function `0xec22f4c7` that transfers tokens from an arbitrary address (LP) to caller-specified address without auth; paired on Pancake V2 with USDT.
- Impact: attacker pulled almost all YDT from the LP, then swapped to USDT for ~41k USD.
- Representative tx: https://bscscan.com/tx/0x585d8be6a0b07ca2f94cfa1d7542f1a62b0d3af5fab7823cbcf69fb243f271f8.

Signal
- Token exposes non-standard function that moves tokens from arbitrary address with no access control.
- LP address passed as `from`, attacker as `to`, amount near full LP balance.

Exploit Steps
- 1) Call YDT function `0xec22f4c7(from=LP, to=attacker, amount=LP balance - 1000e6, taxModule=0x013E...f0)` to yank tokens from the pair.
- 2) Call pair selector `0xfff6cae9` (pair maintenance).
- 3) Approve router and swap stolen YDT→USDT on Pancake.

Key Code/Storage
- Token: YDT 0x3612e4Cb34617bCac849Add27366D8D85C102eFd.
- Pair: 0xFd13B6E1d07bAd77Dd248780d0c3d30859585242 (Pancake V2 YDT/USDT).
- Tax module param: 0x013E29791A23020cF0621AeCe8649c38DaAE96f0.
- Attacker: 0x2ae2f691642bb18cd8deb13a378a0f95a9fee933; attack contract: 0xf195b8800b729aee5e57851dd4330fcbb69f07ea.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/YDTtoken_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1926587721885040686
