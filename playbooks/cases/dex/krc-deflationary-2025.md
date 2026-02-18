Title: KRC LP reserve desync via transfer+skim spam (2025-05-18)

Summary
- Type: reserve desync on a fee/deflationary token pair.
- Preconditions: Pancake V2 pair for KRC/USDT exposes `skim`/`swap` without guard; KRC takes fees on transfer so balances drift from cached reserves.
- Impact: attacker pulled ~355k USDT from the pair; net profit ~7k USD after flash loan fees (small pool).
- Representative tx: https://bscscan.com/tx/0x78f242dee5b8e15a43d23d76bce827f39eb3ac54b44edcd327c5d63de3848daf.

Signal
- Fee-on-transfer token paired with standard AMM; repeated `transfer(pair); skim()` calls in the trace.
- Large `swap` after many alternating transfer/skim calls; reserves out of sync with balances.
- No `sync` before the final swap.

Exploit Steps
- 1) Flash-loan USDT from DODO and Pancake V3.
- 2) Buy KRC via router (two swaps) to get working balance.
- 3) Perform ~17 sequences of `KRC.transfer(pair, X); pair.skim(attacker);` to push fee-deducted amounts into the pair while skimming out the credited amounts, forcing reserves to lag real balances.
- 4) Call `pair.swap(0, 355,361.9345 USDT, attacker, data)` using distorted reserves to receive excess USDT.
- 5) Repay flash loans; keep remainder.

Key Code/Storage
- Pair: 0xdBEAD75d3610209A093AF1D46d5296BBeFFd53f5 (Pancake V2 KRC/USDT).
- Token: KRC 0x1814…b58 (fee-on-transfer).
- Flash loans: DODO 0x6098…B476 and Pancake V3 pool 0x3669…2050.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/KRCToken_pair_exp.sol
- Alert: https://x.com/CertikAIAgent/status/1924280794916536765
- Related pattern: lp-skim-sync-reserve-desync.md
