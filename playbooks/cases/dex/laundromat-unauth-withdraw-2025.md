Title: Laundromat unauthenticated withdraw flow (2025-04-08)

Summary
- Type: withdraw flow callable with reused signatures; no binding to caller.
- Preconditions: `withdrawStart/withdrawStep/withdrawFinal` callable by anyone with static signature array; deposits not tied to caller; signatures not bound to msg.sender.
- Impact: attacker replayed known signature values to trigger withdraw sequence and claim funds (~1.5k USD).
- Representative tx: https://etherscan.io/tx/0x08ffb5f7ab6421720ab609b6ab0ff5622fba225ba351119c21ef92c78cb8302c.

Signal
- Withdraw functions accept signature arrays/constants without verifying signer/owner or replay.
- No checks that caller matches depositor; static params can be reused.

Exploit Steps
- 1. Deploy contract matching expected address; call `deposit` multiple times with hardcoded params.
- 2. Call `withdrawStart` with reused signature array and parameters.
- 3. Call `withdrawStep` repeatedly, then `withdrawFinal` to receive funds.
- 4. Selfdestruct/transfer proceeds to attacker EOA.

Key Code/Storage
- Vulnerable contract: 0x934cbbE5377358e6712b5f041D90313d935C501C.
- Functions: `deposit(uint256,uint256)`, `withdrawStart(uint256[],uint256,uint256,uint256)`, `withdrawStep()`, `withdrawFinal()`; signatures not bound to caller.
- Attacker: 0xd6be07499d408454d090c96bd74a193f61f706f4; attack contract: 0x2e95cfc93ebb0a2aace603ed3474d451e4161578.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-04/Laundromat_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1909814943290884596
