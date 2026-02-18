Title: Corkprotocol access control/flow abuse (2025-05-28)

Summary
- Type: complex access control/flow flaw across proxies/hooks allowing asset drain.
- Preconditions: multiple modules (ERC1967 proxies, CorkHook, PoolManager) callable/configurable by attacker; approvals set; lack of auth on critical functions.
- Impact: attacker manipulated module initialization, swaps, and returns to drain wstETH (~12M USD loss reported).
- Representative tx: https://app.blocksec.com/explorer/tx/eth/0xfd89cdd0be468a564dd525b222b728386d7c6780cf7b2f90d2b54493be09f64d.

Signal
- Critical init/config functions callable by anyone; complex hook/swap flows without auth.
- Multiple proxies/hooks interacting with user-controlled parameters.

Exploit Steps
- 1. Transfer tokens into proxy; set up swap assets via `getDeployedSwapAssets`.
- 2. Manipulate reserves via CorkHook swap with minimal minOut.
- 3. Initialize module core/issue assets; deposit/return functions called by attacker.
- 4. Abuse PoolManager unlock/callback to move assets; withdraw wstETH.

Key Code/Storage
- Proxies: 0xCCd90F6435dd78C4ECCED1FA4db0D7242548a2a9, 0x96E0121D1cb39a46877aaE11DB85bc661f88D5fA.
- Tokens: LiquidityToken 0x05816980fAEC123dEAe7233326a1041f372f4466, wstETH 0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0.
- Hook/manager: CorkHook 0x5287E8915445aee78e10190559D8Dd21E0E9Ea88; PoolManager 0x000000000004444c5dc75cB358380D2e3dE08A90.
- Attacker: 0xea6f30e360192bae715599e15e2f765b49e4da98; contract: 0x9af3dce0813fd7428c47f57a39da2f6dd7c9bb09.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/Corkprotocol_exp.sol
- Alert: https://x.com/SlowMist_Team/status/1928100756156194955
