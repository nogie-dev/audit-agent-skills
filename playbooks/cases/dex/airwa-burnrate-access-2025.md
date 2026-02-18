Title: AIRWA burnRate unprotected leading to LP drain (2025-04-04)

Summary
- Type: access control flaw; anyone can set burnRate to extreme values and manipulate fee-on-transfer behavior.
- Preconditions: `setBurnRate` external/public; burn applied on transfer; LP holds AIRWA/BUSD/WBNB.
- Impact: attacker set burnRate=980 (98%) then forced transfer to LP (amount=0) to apply high fee logic, then reset burnRate=0 and swapped out tokens for WBNB, profiting ~56.73 BNB.
- Representative tx: https://bscscan.com/tx/0x5cf050cba486ec48100d5e5ad716380660e8c984d80f73ba888415bb540851a4.

Signal
- Admin setter (e.g., `setBurnRate`) callable by anyone; no owner/onlyOwner.
- Fee/burn parameters directly affect transfers to LP; can be toggled mid-tx.

Exploit Steps
- 1. Buy small AIRWA with 0.1 BNB.
- 2. Call `setBurnRate(980)` (public).
- 3. Transfer 0 AIRWA to LP to trigger fee logic with high burn.
- 4. Reset `setBurnRate(0)`.
- 5. Swap all AIRWA→BUSD→WBNB; receive inflated WBNB; withdraw to EOA (~56.73 BNB).

Key Code/Storage
- Token: AIRWA 0x3Af7DA38C9F68dF9549Ce1980eEf4AC6B635223A; has `setBurnRate` public.
- LP: 0xc3551400c032cB0556dee1AD1dC78D1cbC64B7bb.
- Router: Pancake V2 0x10ED43C718714eb63d5aA57B78B54704E256024E.
- Attacker: 0x70f0406e0a50c53304194b2668ec853d664a3d9c; contract 0x2a011580f1b1533006967bd6dc63af7ae5c82363.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-04/AIRWA_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1908086092772900909
