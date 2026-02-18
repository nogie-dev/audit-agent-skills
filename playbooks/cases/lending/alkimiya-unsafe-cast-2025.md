Title: Alkimiya SilicaPools unsafe cast on shares (2025-03-28)

Summary
- Type: unchecked downcast of shares to uint128 allows overflowed mint.
- Preconditions: `collateralizedMint` casts `shares` to uint128 without bounds; attacker can pass value > uint128; pool uses morpho flashloan for liquidity.
- Impact: attacker passed `shares = uint256(type(uint128).max) + 2`, causing wrap and misaccounted ERC1155 short tokens, then manipulated pool lifecycle to redeem excess WBTC (~95.5k USD, 1.14 WBTC) via flashloan.
- Representative tx: https://etherscan.io/tx/0x9b9a6dd05526a8a4b40e5e1a74a25df6ecccae6ee7bf045911ad89a1dd3f0814.

Signal
- Parameters like `shares` downcast to `uint128` without range check.
- Pool minting long/short tokens based on cast value; accepts arbitrary caller inputs.
- Uses flashloanable tokens as collateral.

Exploit Steps
- 1. Flashloan WBTC via Morpho.
- 2. Call `collateralizedMint` with `shares = uint128_max + 2` to trigger overflow in cast.
- 3. Receive misaccounted ERC1155 positions; call `safeTransferFrom`, `startPool`, `endPool`, `redeemShort` to extract WBTC.
- 4. Repay flashloan; keep profit.

Key Code/Storage
- SilicaPools: 0xf3F84cE038442aE4c4dCB6A8Ca8baCd7F28c9bDe.
- Flashloan: Morpho 0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb.
- Token: WBTC 0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599.
- Attack contract: 0x80bf7db69556d9521c03461978b8fc731dbbd4e4; attacker 0xF6ffBa5cbF285824000daC0B9431032169672B6e; MEV 0xFDe0d1575Ed8E06FBf36256bcdfA1F359281455A.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/Alkimiya_io_exp.sol
- Post-mortem: https://blog.decurity.io/yul-calldata-corruption-1inch-postmortem-a7ea7a53bfd9 (note: not Alkimiya-specific; primary analysis in PoC)
- Alert: https://x.com/TenArmorAlert/status/1906371419807568119
