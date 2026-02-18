Title: YBToken no-slippage protection reserve targeting (2025-04-16)

Summary
- Type: missing slippage protection; attacker computes exact swap amounts to target constant product.
- Preconditions: Pancake V2 pair YB/BUSD; swap calls accept any amount (no minOut check); attacker can flashloan BUSD; reserves and balance used in custom math.
- Impact: attacker flashloaned BUSD, split into many sub-swaps (66), computed outputs to reach target K with no slippage check, drained ~15,261 BUSD.
- Representative tx: https://bscscan.com/tx/0xe1e7fa81c3761e2698aa83e084f7dd4a1ff907bcfc4a612d54d92175d4e8a28b.

Signal
- Swaps executed without `amountOutMin`/slippage enforcement; attacker can set output amounts (compute to target K).
- Pair uses fee factors but caller supplies exact amounts to satisfy invariant.
- Flashloan-funded multi-iteration swaps.

Exploit Steps
- 1. Flashloan BUSD from Pancake V3 pool.
- 2. Loop 66 times: transfer BUSD chunk to YB/BUSD LP; compute `amount0Out` to push K to target; swap to receive YB via child contracts.
- 3. Loop to swap YB back to BUSD with computed `amount1Out` targeting K, extracting BUSD.
- 4. Repay flashloan; keep ~15.26k BUSD.

Key Code/Storage
- Token: YB 0x04227350eDA8Cb8b1cFb84c727906Cb3CcBff547.
- Pair: YB/BUSD LP 0x38231F8Eb79208192054BE60Cb5965e34668350A (Pancake V2 style).
- Flashloan pool: 0x36696169C63e42cd08ce11f5deeBbCeBae652050.
- Attacker: 0x00000000b7da455fed1553c4639c4b29983d8538; contract 0xbdcd584ec7b767a58ad6a4c732542b026dceaa35.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-04/YBToken_exp.sol
- Alert: https://x.com/CertiKAlert/status/1912684902664782087
