Title: Cross-contract reentrancy skews price/avg calc

Summary
- Entry function is nonReentrant only within the contract; attacker reenters another contract to call sensitive methods (e.g., increasePosition) directly, bypassing normal routers/managers and their accounting.
- Impact: average price/PnL calculation skipped or manipulated, leading to inflated LP token price or mispriced positions.

Signal
- Multi-contract flow where router/manager is expected to gate calls; core contract callable by anyone.
- nonReentrant on entry contract only; reentrancy into a different contract is possible.
- Price/avg calculations rely on routed path; direct call can set abnormal avg price.

Exploit Steps
- 1. Invoke entrypoint (e.g., OrderBook) to start operation.
- 2. Reenter and call core contract (e.g., Vault.increasePosition) directly during the process.
- 3. Manipulate average price/short price; downstream LP/GLP price inflates.
- 4. Mint/redeem at distorted price; profit.

Examples
- playbooks/cases/dex/gmx-v1-orderbook-reentrancy-2025.md
