Title: RICE masterContractApproval bypass (2025-05-24)

Summary
- Type: unauthorized approval → withdraw arbitrary user funds.
- Preconditions: BentoBox-style contract exposes `registerProtocol` and `setMasterContractApproval` with signature params, but accepts zeroed signatures and any `user` address.
- Impact: attacker approved themselves for victim 0x4987… and withdrew all RICE tokens, swapping to USDT/WETH for ~34.5 WETH (~88k USD).
- Representative tx: https://basescan.org/tx/0x8421c96c1cafa451e025c00706599ef82780bdc0db7d17b6263511a420e0cf20.

Signal
- `setMasterContractApproval(user, master, true, v, r, s)` callable by anyone; no `msg.sender` == user check and no signature validation.
- `withdraw(token, from, to, amount, share)` callable after forged approval.
- Registering protocol immediately before approval.

Exploit Steps
- 1) Call `registerProtocol()` on unverified vault 0xcfe0… to enable itself.
- 2) Call `setMasterContractApproval(user=0x4987…, masterContract=self, approved=true, v=0, r=0, s=0)` to forge approval.
- 3) Call `withdraw(RICE, from=user, to=attacker, amount=share=22_189_176_505_973_791_717_313_474)` to drain victim balance.
- 4) Swap stolen RICE to USDT (Uniswap V3) then to WETH via router; profit.

Key Code/Storage
- Vault contract: 0xcfE0DE4A50C80B434092f87e106DFA40b71A5563 (`setMasterContractApproval` missing signature/auth validation).
- Token: RICE 0xf501E4c51dBd89B95de24b9D53778Ff97934cd9c.
- Routers: Uniswap V3 SwapRouter02 0x2626… and custom SwapRouter 0xBE6D8f0d05cC4be24d5167a3eF062215bE6D18a5.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/RICE_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1926461662644633770
- Related pattern: forged-master-approval-bentobox.md
