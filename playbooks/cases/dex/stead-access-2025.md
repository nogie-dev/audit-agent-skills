Title: Stead public drain function (2025-06-20)

Summary
- Type: missing access control on token transfer function.
- Preconditions: Contract 0xf9ff... exposes selector 0x16fb27ce callable by anyone; parameters allow draining STEAD tokens held by the contract.
- Impact: attacker called the function with arbitrary params to transfer out STEAD, gaining ~14.5k USD.
- Representative tx: https://arbiscan.io/tx/0x32dbfce2253002498cd41a2d79e249250f92673bc3de652f3919591ee26e8001.

Signal
- Public external function that moves contract-owned tokens without owner/role check.
- No validation on caller or transfer amount.

Exploit Steps
- 1) Call function `0x16fb27ce(1,1,1)` on contract 0xf9FF933f51bA180a474634440a406c95DfB27596.
- 2) Function transfers STEAD from contract to caller.

Key Code/Storage
- Vulnerable contract: 0xf9FF933f51bA180a474634440a406c95DfB27596 (Arbitrum).
- Token: STEAD 0x42F4e5Fcd12D59e879dbcB908c76032a4fb0303b.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-06/Stead_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1939508301596672036
- Related pattern: treasury-swap-no-access-control.md
