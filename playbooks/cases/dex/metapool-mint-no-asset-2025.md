Title: Meta Pool mpETH mint without assets (2025-06-17)

Summary
- Type: access control/economic invariant breach in ERC4626 mint.
- Preconditions: Staking (mpETH) inherits ERC4626Upgradeable; `mint()` left public, non-payable, no ETH/value check; internal `_deposit` fallback mints shares even if pool swap fails; attacker drained liquidUnstakePool first.
- Impact: attacker minted ~9,702 mpETH with zero ETH, swapped part for ETH, across chains total ~56.35 ETH loss (main tx ~25k USD).
- Representative tx: https://etherscan.io/tx/0x57ee419a001d85085478d04dd2a73daa91175b1d7c11d8a8fb5622c56fd1fa69.

Signal
- ERC4626 `mint(uint256 shares, address receiver)` exposed without override/access control.
- `mint` not payable and lacks `msg.value`/asset transfer validation.
- `_deposit` splits swap-from-pool then mint remainder; if pool empty or no value, mints full amount anyway.

Exploit Steps
- 1) Flash-loan 200 WETH; unwrap ~107 WETH to ETH.
- 2) Call `depositETH()` to drain mpETH from internal liquidUnstakePool (so swap path yields nothing later).
- 3) Call unrestricted `mint(~9702 mpETH, receiver)` with zero ETH; swap bypassed, fallback mint mints all shares.
- 4) Swap minted mpETH for ETH (pool + Uniswap V3), repay flash loan, keep profit.

Key Code/Storage
- Staking contract inherits ERC4626Upgradeable; `mint` exposed (victim 0x48afbbd342f64ef8a9ab1c143719b63c2ad81710).
- `_deposit` calls `_getmpETHFromPool()` then mints remainder; no check for received ETH.
- Liquid pool drained first to force full mint.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-06/MetaPool_exp.sol
- Analysis: https://www.coindesk.com/business/2025/06/17/liquid-staking-protocol-meta-pool-suffers-usd27m-exploit , https://medium.com/@SolidityScan/meta-pool-hack-analysis
- Alert: https://x.com/peckshield/status/1934895187102454206
- Related pattern: public-erc4626-mint-no-asset.md
