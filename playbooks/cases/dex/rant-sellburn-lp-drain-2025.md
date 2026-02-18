Title: RANT sellBurn LP drain via transfer-to-self (2025-07-05)

Summary
- Type: deflationary sell burn misuses user amount to pull from LP.
- Preconditions: `_sellBurnLiquidityPairTokens(amount)` called when sender unwhitelisted and recipient is token contract; uses `amount` (user input) to remove tokens from LP and send to burn/node; then syncs reserves.
- Impact: attacker flash-loaned WBNB, bought RANT, then transferred RANT to token contract to trigger sell burn that drained almost all RANT from LP; swapped at inflated price, profiting ~311.4 BNB (~$204k).
- Representative tx: https://bscscan.com/tx/0x2d9c1a00cf3d2fda268d0d11794ad2956774b156355e16441d6edb9a448e5a99.

Signal
- Token contract treats transfer-to-self as “sell” and burns based on user `amount`, pulling directly from LP.
- `_sellBurnLiquidityPairTokens` removes `amount` from LP instead of applying burn rate on pool size.
- Calls `sync()` after draining LP, leading to extreme price distortion.

Exploit Steps
- 1) Flash-loan WBNB from Pancake V3.
- 2) Swap WBNB→RANT on Pancake pair; LP holds RANT.
- 3) Transfer RANT to token contract (unwhitelisted sender) → triggers `_sellBurnLiquidityPairTokens(amount)`, removing `amount` from LP to dead/node, nearly emptying RANT side.
- 4) `sync()` updates reserves; swap back to WBNB at inflated price.
- 5) Repay flash loan; keep profit (~311.4 BNB).

Key Code/Storage
- Token: RANT 0xc321AC21A07B3d593B269AcdaCE69C3762CA2dd0 (`_transfer` branch for recipient == address(this) calls `_sellBurnLiquidityPairTokens(amount)`; drains LP and calls sync).
- Pair: RANT/WBNB 0x42A93C3aF7Cb1BBc757dd2eC4977fd6D7916Ba1D.
- Flash pool: Pancake V3 0x172fcD41E0913e95784454622d1c3724f546f849.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/RANTToken_exp.sol
- Analysis: https://verichains.substack.com/p/rant-exploit-analysis
- Alerts: https://x.com/Phalcon_xyz/status/1941788315549946225 , https://x.com/AgentLISA_ai/status/1942162643437203531
- Related pattern: lp-skim-sync-reserve-desync.md
