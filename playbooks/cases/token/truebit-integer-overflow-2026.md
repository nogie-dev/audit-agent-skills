Title: Truebit Token Sale Price Overflow (2026-01-08)

Summary
- Type: unchecked integer overflow in pricing math (Solidity <0.8.0, no built-in overflow checks).
- Preconditions: user-controlled `amount` flows into multi-term multiplication with `reserve`, `totalSupply`, and config param (THETA) without bounding or SafeMath.
- Impact: purchase price computed as 0, letting attacker mint TRU for 0 ETH and swap back to ETH; ~8,535 ETH stolen by primary attacker, ~224k more by a second.
- Representative tx: 0xcd4755645595094a8ab984d0db7e3b4aabde72a5c87c4f176a030629c47fb014 (AdminUpgradeabilityProxy.buyTRU/sellTRU loop).

Signal
- Pricing functions using raw multiplication on large inputs (amount * reserve * totalSupply) with no overflow checks in pre-0.8 Solidity.
- `getPurchasePrice` returns 0 or unexpectedly tiny for large `amount`.
- Absence of input caps or sanity checks on price output (no revert on zero price).

Exploit Steps
- 1. Call `getPurchasePrice(amount)` with a very large `amount` to force overflow in numerator, returning 0.
- 2. Call `buyTRU()` with returned price (0 wei) to mint massive TRU for free.
- 3. Call `sellTRU()` to swap minted TRU back for ETH; repeat with increasing `amount` to drain reserves.

Key Code/Storage
- Vulnerable contract: 0x764C64b2A09b09Acb100B80d8c505Aa6a0302EF2 (AdminUpgradeabilityProxy → logic at 0xC186e6F0163e21be057E95aA135eDD52508D14d3).
- Pricing function `getPurchasePrice` → private `0x1446` → helper `0x18ef` for multiplication without SafeMath.
- Formula (conceptual): `(200 * amount * reserve * totalSupply + 100 * amount^2 * reserve) / (100 * totalSupply^2 - THETA * totalSupply^2)`; large `amount` makes the numerator overflow to a small value, yielding 0 price.
- Key params: `THETA` = 75 (from `_setParameters`); `_reserve` (ETH in pool) ≈ 8,539 ETH at exploit time; `totalSupply` ≈ 161,753,242,367,424,992,669,183,203.

Fix/Detect
- Use Solidity ≥0.8.0 or SafeMath for critical pricing; bound `amount` inputs and reject zero/underflowed prices.
- Add sanity checks: revert if price == 0 or if multiplication overflows (checked math), and cap max purchasable size per tx.
- Monitoring: alert on price = 0 for nonzero amount, sudden massive mints, repeated buy/sell cycles within one tx.

Refs
- CertiK: https://www.certik.com/zh-CN/resources/blog/truebit-incident-analysis
- Attack tx: https://etherscan.io/tx/0xcd4755645595094a8ab984d0db7e3b4aabde72a5c87c4f176a030629c47fb014
- Attacker wallets: 0x6C8EC8f14bE7C01672d31CFa5f2CEfeAB2562b50, 0xc0454E545a7A715c6D3627f77bEd376a05182FBc
- Attack contract: 0x1De399967B206e446B4E9AeEb3Cb0A0991bF11b8
