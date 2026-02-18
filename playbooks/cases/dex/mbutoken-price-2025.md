Title: MBU deposit mint mispricing (2025-05-11)

Summary
- Type: public deposit mints tokens at absurd fixed rate (no price feed).
- Preconditions: ERC1967 proxy `deposit(token, amount)` accepts any token and mints MBU using a mis-scaled conversion; callable by anyone.
- Impact: attacker deposited 0.001 WBNB and received ~30,000,000 MBU, swapped to BUSD for ~2.16M BUSD.
- Representative tx: https://bscscan.com/tx/0x2a65254b41b42f39331a0bcc9f893518d6b106e80d9a476b8ca3816325f4a150.

Signal
- Deposit function lacks auth/whitelist and uses hardcoded or stale rate to mint reward token.
- Huge token mint relative to tiny deposit; immediately swapped out via router.
- Upgradeable proxy with `deposit` exposed on the implementation.

Exploit Steps
- 1) Wrap 0.001 BNB to WBNB and approve proxy 0x95e9….
- 2) Call `deposit(WBNB, 0.001 ether)` to mint ~30M MBU due to bad rate math.
- 3) Approve Pancake router and swap MBU→BUSD.
- 4) Pay MEV protection tip; keep remaining BUSD.

Key Code/Storage
- Proxy: 0x95e92B09b89cF31Fa9F1Eca4109A85F88EB08531 (`deposit` method).
- Token: MBU 0x0dFb6Ac3A8Ea88d058bE219066931dB2BeE9A581.
- Router: Pancake V2 0x10ED… (swap route MBU→BUSD).

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/MBUToken_exp.sol
- Alerts: https://x.com/TenArmorAlert/status/1921474575965065701 , https://x.com/CertiKAlert/status/1921483904483000457
- Related pattern: unchecked-pricing-overflow.md
