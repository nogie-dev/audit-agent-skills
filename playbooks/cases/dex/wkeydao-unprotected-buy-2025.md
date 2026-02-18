Title: wKeyDAO unprotected buy drains contract funds (2025-03-16)

Summary
- Type: access control/logic flaw in buy function; contract BUSD used to mint tokens for caller.
- Preconditions: `buy()` callable by anyone; uses contract-held BUSD to purchase/mint wKeyDAO without requiring caller payment; attacker can loop and then swap tokens to BUSD on Pancake.
- Impact: attacker flashloaned BUSD, looped `buy` then sold wKeyDAO for BUSD, extracting ~767 USD.
- Representative tx: https://app.blocksec.com/explorer/tx/bsc/0xc9bccafdb0cd977556d1f88ac39bf8b455c0275ac1dd4b51d75950fb58bad4c8.

Signal
- Public `buy()` with no amount/price input, no payment transfer from msg.sender; relies on contract balances.
- Token sale contract holds stablecoins and dispenses tokens without auth/limit.

Exploit Steps
- 1. Flashloan BUSD.
- 2. Approve sell contract and router.
- 3. Loop `buy()` to receive wKeyDAO funded by contract BUSD.
- 4. Swap wKeyDAO→BUSD on Pancake; repay flashloan; keep profit.

Key Code/Storage
- Sale contract: 0xD511096a73292A7419a94354d4C1C73e8a3CD851 (`buy()` public).
- Token: wKeyDAO 0x194B302a4b0a79795Fb68E2ADf1B8c9eC5ff8d1F.
- Router: Pancake V2 0x10ED43C718714eb63d5aA57B78B54704E256024E.
- Attack contract: 0x3783c91ee49a303c17c558f92bf8d6395d2f76e3; attacker 0x3026c464d3bd6ef0ced0d49e80f171b58176ce32.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/wKeyDAO_exp.sol
- Alert: https://x.com/Phalcon_xyz/status/1900809936906711549
- Related pattern: unprotected-sale-contract-buy.md
