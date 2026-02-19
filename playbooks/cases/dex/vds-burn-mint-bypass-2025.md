Title: VDS burn-to-mint AVD loop (2025-07-09)

Summary
- Type: burn/mint redemption flaw; sending VDS to token contract returns underlying 1:1.
- Preconditions: VDS contract accepts `deposit(token, amount)` to mint VDS from AVD, and `transfer(VDS, VDS_contract)` burns VDS and returns AVD at same rate; no fee/check; flashloan available.
- Impact: attacker flash-loaned USDT, swapped to AVD, minted VDS, then transferred VDS to its own contract to reclaim AVD, netting profit when swapping back (~13k USD).
- Representative tx: https://bscscan.com/tx/0x0e01fd8798f970fd689014cb215e622aca8b7c8c243176c5b504e0043402e31f.

Signal
- Token contract honors transfers to self as redemption of underlying at fixed 1:1.
- No restriction on `deposit`/`burn` path; supports fee-on-transfer but redeems full amount.

Exploit Steps
- 1) Flash-loan 20k USDT from Moolah.
- 2) Swap USDT→AVD via Pancake.
- 3) Call `deposit(VDS, avdBalance)` to mint VDS.
- 4) Transfer VDS to VDS contract address to burn and receive equal AVD.
- 5) Swap AVD→USDT, repay flash loan; keep remainder.

Key Code/Storage
- VDS token: 0x6ce69d7146dbaae18c11c36d8D94428623B29D5A (`deposit` mints; transfer-to-self returns AVD 1:1).
- AVD token: 0x4Ec93ee81f25dA3C8e49F01533cfB734545190A8.
- Flash loan: Moolah 0x8F73b65B4caAf64FBA2aF91cC5D4a2A1318E5D8C.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/VDS_exp.sol
- Alert: https://x.com/SlowMist_Team/status/1945672192471302645
- Related pattern: self-transfer-redeem-underlying.md
