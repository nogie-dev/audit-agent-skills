Title: SizeCredit leverageUpWithSwap route injection (2025-08-15)

Summary
- Type: arbitrary call via unvalidated swap route.
- Preconditions: `leverageUpWithSwap(GenericRoute[] routes, ...)` executes caller-provided route data without validation of target/selector; protocol/victim has pre-approved PT-wstUSR to the SizeCredit contract.
- Impact: attacker crafted route that called `transferFrom(victim, attacker, amount)` on PT-wstUSR, stealing ~19.7k USD.
- Representative tx: per PoC for 2025-08-15 (SizeCredit_exp.sol).

Signal
- Swap/route structs contain `target`, `callData` used as-is; no whitelist of functions or tokens.
- Contract holds or has allowance to move third-party tokens (PT-wstUSR).

Exploit Steps
- 1) Prepare GenericRoute with `target=PT-wstUSR` and `callData=transferFrom(victim, attacker, amount)`; mark success allowed.
- 2) Call `leverageUpWithSwap` with crafted route; contract executes transferFrom using victim’s allowance.
- 3) Receive PT-wstUSR at attacker; profit.

Key Code/Storage
- Function: `leverageUpWithSwap` accepts `GenericRoute[]` and forwards calls.
- Token: PT-wstUSR (pre-approved by victim to protocol).

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-08/SizeCredit_exp.sol
- Alert/analysis: internal note on GenericRoute injection (2025-08-15)
- Related pattern: unvalidated-swap-callback.md
