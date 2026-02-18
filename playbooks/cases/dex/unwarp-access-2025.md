Title: Unwarp access control flaw (2025-05-14)

Summary
- Type: lack of access control on sensitive function.
- Impact: ~9K USD loss.

Signal
- Admin-like function callable by anyone.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/Unwarp_exp.sol
Title: Unwarp WETH unwrap without auth (2025-05-14)

Summary
- Type: missing access control on treasury unwrap.
- Preconditions: Contract 0x8bEf… exposes `unwrapWETH(uint256 amount, address to)` callable by anyone; holds WETH balance.
- Impact: attacker flash-loaned WETH from Balancer, invoked `unwrapWETH` to pull ETH to themselves, then repaid loan; ~9k USDT-equivalent taken.
- Representative tx: https://app.blocksec.com/explorer/tx/base/0xac6f716c57bbb1a4c1e92f0a9531019ea2ecfcaea67794bbd27115d400ae9b41.

Signal
- Public function that transfers/unwraps protocol-owned WETH to arbitrary address without owner/role check.
- Attack trace shows Balancer flash loan → transfer WETH to target → external call to `unwrapWETH`.

Exploit Steps
- 1) Flash-loan ~100.85 WETH from Balancer Vault.
- 2) Transfer the loaned WETH to contract 0x8bEf… .
- 3) Call `unwrapWETH(104.833984375 WETH, attacker)` to receive ETH directly.
- 4) Rewrap ETH to WETH and repay the flash loan; keep remainder.

Key Code/Storage
- Target contract: 0x8bEfC1d90d03011a7d0b35b3a00eC50f8E014802 (`unwrapWETH` lacks auth).
- Flash loan: Balancer Vault 0xBA12222222228d8Ba445958a75a0704d566BF2C8.
- Token: WETH on Base 0x4200…0006.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/Unwarp_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1921365719664957728
- Related pattern: treasury-swap-no-access-control.md
