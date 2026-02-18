Title: BBX burn-gap bypass to drain LP (2025-03-20)

Summary
- Type: burn mechanism/burn gap logic flaw allowing repeated zero-value transfers to bypass cooldown.
- Preconditions: BBX token burns on transfer with a `lastBurnGapTime` (86400s) cooldown; transferring 0 tokens updates burn timestamp without burning; LP holds BUSD/BBX.
- Impact: attacker spammed ~500 zero-amount transfers to reset burn timer, avoided burn, then swapped BBX→BUSD to drain ~11,902 BUSD.
- Representative tx: https://bscscan.com/tx/0x0dd486368444598610239b934dd9e8c6474a06d11380d1cfec4d91568b5ac581.

Signal
- Token burn logic depends on `lastBurnTime`/gap but updates on any transfer, including amount=0.
- Burn rate not applied when cooldown not met; zero transfers can advance timer without burning.
- LP uses fee-on-transfer token; no additional protections.

Exploit Steps
- 1. Buy minimal BBX with small ETH→BUSD→BBX swap.
- 2. Send 0-value BBX transfers in a loop (~500) to bypass burn cooldown.
- 3. Swap all BBX→BUSD via Pancake router; no burn applied; drain LP of BUSD.

Key Code/Storage
- Token: BBX 0x67Ca347e7B9387af4E81c36cCA4eAF080dcB33E9 with `lastBurnTime`, `lastBurnGapTime`, `burnRate` (300), `liquidityPool` 0x6051428B580f561B627247119EEd4D0483B8D28e.
- LP: PancakeSwap V2 BUSD/BBX 0x6051428B580f561B627247119EEd4D0483B8D28e.
- Attack contract: 0x0489E8433e4E74fB1ba938dF712c954DDEA93898; attacker 0x8aea7516b3b6aabf474f8872c5e71c1a7907e69e.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/BBXToken_exp.sol
- Post-mortem: https://blog.solidityscan.com/bbx-token-hack-analysis-f2e962c00ee5
- Alert: https://x.com/TenArmorAlert/status/1916312483792408688
