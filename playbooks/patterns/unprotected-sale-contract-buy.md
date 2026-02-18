Title: Unprotected sale contract buy uses contract funds

Summary
- Sale/buy function callable by anyone consumes contract-held funds to mint/dispense tokens; caller pays nothing.
- Impact: attackers loop buy and immediately sell tokens on AMM, draining contract reserves/LP.

Signal
- `buy()` (or similar) with no payment parameter or msg.value use; relies on internal balances; no auth/limits per caller.
- Contract holds stablecoins or base asset used to fund buys.

Exploit Steps
- 1. Fund/flashloan base asset if needed.
- 2. Loop public `buy()` to receive tokens funded by contract.
- 3. Sell tokens for base asset on AMM; repay loan; keep spread.

Key Code/Storage
- Public sale functions lacking payment/authorization; contract maintains payout balance.

Refs
- Example: wKeyDAO unprotected buy drains contract funds 2025-03-16 (playbooks/cases/dex/wkeydao-unprotected-buy-2025.md)
