Title: UsualMoney USD0+/sUSDS price mis-sync (2025-05-27)

Summary
- Type: oracle/mint pricing abuse via attacker-seeded pool.
- Preconditions: VaultRouter swaps USD0+ into sUSDS using arbitrary ParaSwap calldata; share accounting trusts spot price from a freshly created Uniswap V3 USD0/sUSDS pool.
- Impact: attacker flash-borrowed USD0+, routed through their tiny V3 pool to mint overpriced vault shares, then unwound for ~43k USD profit.
- Representative tx: https://etherscan.io/tx/0x585d8be6a0b07ca2f94cfa1d7542f1a62b0d3af5fab7823cbcf69fb243f271f8.

Signal
- User-supplied swap calldata (ParaSwap/Augustus) determines how vault converts deposits.
- Vault share price derived from an external pool that can be created/seeded by anyone.
- Ability to mint a Uniswap V3 pool and set sqrtPriceX96 immediately before deposit.

Exploit Steps
- 1) Seed a new USD0/sUSDS Uniswap V3 pool with 10 sUSDS at an attacker-chosen price (min liquidity).
- 2) Flash-loan ~1.9m USD0+ from Morpho Blue.
- 3) Call `VaultRouter.deposit` with custom ParaSwap calldata that swaps the loan through the manipulated V3 pool, minting sUSDS at the inflated rate and vault shares against it.
- 4) Burn liquidity/collect from the V3 position, then swap and Curve-exchange USD0/USD0+ to repay the loan; keep the excess as profit.

Key Code/Storage
- VaultRouter: 0xE033cb1bB400C0983fA60ce62f8eCDF6A16fcE09 (`deposit` trusts caller-provided Augustus calldata).
- Uniswap V3 pool: created via `createAndInitializePoolIfNecessary` on USD0 (0x73A1…) / sUSDS (0xa393…).
- Flash loan: Morpho Blue 0xBBBB… used to source USD0+ (0x35D8…).

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/UsualMoney_exp.sol
- Alert: https://x.com/BlockSecTeam/status/1927601457815040283
- Write-up: https://www.quadrigainitiative.com/hackfraudscam/usualmoneyusdssyncvaultpricingarbitrageexploit.php
- Related pattern: user-seeded-pool-swap-route-oracle.md
