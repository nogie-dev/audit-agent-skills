Title: LeverageSIR storage slot collision & callback abuse (2025-03-30)

Summary
- Type: storage slot collision + arbitrary callback via crafted swap data.
- Preconditions: vault `mint` writes to storage slot 1 with user-supplied amount; address collisions allow deploying a contract at that slot; `uniswapV3SwapCallback` trusts calldata `data` to route transfers; vault holds USDC/WBTC/WETH.
- Impact: attacker manipulated slot 1 to point to a CREATE2 contract, then used swap callbacks to drain USDC/WBTC/WETH (~$353.8K total: ~17.8k USDC, 1.4085 WBTC, 119.87 WETH).
- Representative tx: https://etherscan.io/tx/0xa05f047ddfdad9126624c4496b5d4a59f961ee7c091e7b4e38cee86f1335736f.

Signal
- Vault stores critical address in low slot using user-controlled value (e.g., `tstore(1, amount)` in mint).
- Allows arbitrary callback (`uniswapV3SwapCallback`) with attacker-controlled `data` leading to transfers.
- Uses CREATE2/deploy to collide with stored slot address to gain control of callback target.

Exploit Steps
- 1. Create UniV3 pool and positions with attacker dummy token to set up vault interactions.
- 2. Call `initialize` and `mint` to write crafted value into slot 1 (vaultParameters.collateral?); use large amount to set slot to chosen address (0x…d4281).
- 3. Deploy contract at that CREATE2 address to hijack callback logic.
- 4. Invoke vault functions (e.g., swap callbacks) to transfer USDC/WBTC/WETH to attacker-controlled contract; forward to EOA.

Key Code/Storage
- Vault: 0xb91ae2c8365fd45030aba84a4666c4db074e53e7.
- Slot collision: mint uses `tstore(1, amount)` from user-supplied amount.
- Callback: `uniswapV3SwapCallback` uses `data` addresses for transfers without validation.
- Attack contracts: main 0xea55fffae1937e47eba2d854ab7bd29a9cc29170; CREATE2 0x00000000001271551295307acc16ba1e7e0d4281.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/LeverageSIR_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1906268185046745262
