Title: WETC LP skim/sync price manipulation (2025-07-09)

Summary
- Type: LP reserve desync via manual transfers + skim/sync.
- Preconditions: Pancake V2 BUSD/WETC pair with public `skim`/`sync`; attacker can flash-loan large BUSD/WETC; token allows direct transfers to pair.
- Impact: attacker used flash-loaned BUSD, transferred huge WETC into the pair, used `skim` to misalign reserves, `sync` to record inflated reserves, then swapped out BUSD for profit (~101k USD).
- Representative tx: https://bscscan.com/tx/0x2b6b411adf6c452825e48b97857375ff82b9487064b2f3d5bc2ca7a5ed08d615.

Signal
- Multiple large transfers to pair followed by `skim` and `sync`.
- Fee-on-transfer/deflationary token or manual balance injection causing reserve vs balance drift.
- Final swap after reserve inflation to pull base asset.

Exploit Steps
- 1) Flash-loan ~1,000,000 BUSD (1e24) from Pancake V3.
- 2) Small swap on BUSD/WETC pair to enable callback; transfer flash-loaned BUSD to pair in pancakeCall.
- 3) Transfer large WETC amounts to pair; call `skim` to send excess elsewhere; call `sync` to set reserves to inflated balances. Repeat.
- 4) Swap manipulated reserves to extract BUSD.
- 5) Repay flash loan; keep remaining BUSD.

Key Code/Storage
- Pair: 0x8e2cc521b12dEBA9A20EdeA829c6493410dAD0E3 (Pancake V2 BUSD/WETC).
- Token: WETC 0xE7f12B72bfD6E83c237318b89512B418e7f6d7A7.
- Flash pool: Pancake V3 0x92b7807bF19b7DDdf89b706143896d05228f3121.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/WETC_Token_exp.sol
- Analysis: https://t.me/evmhacks/78?single
- Related pattern: lp-skim-sync-reserve-desync.md
