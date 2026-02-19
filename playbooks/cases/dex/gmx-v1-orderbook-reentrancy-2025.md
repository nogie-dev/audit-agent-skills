Title: GMX V1 OrderBook reentrancy skews short avg price (2025-07-09)

Summary
- Type: reentrancy into Vault `increasePosition` bypasses avg price calc.
- Preconditions: OrderBook `createIncreaseOrder`/execution uses nonReentrant only within OrderBook; external call path lets attacker call Vault directly; average short price not updated when bypassing PositionRouter/Manager.
- Impact: attacker reentered Vault.increasePosition, set BTC short avg price to ~1,913.70 (vs 109,505.77), opened $15.39M short, PnL calc blew up GLP price → minted/sold GLP for ~$40M.
- Representative tx: https://arbiscan.io/tx/0x03182d3f0956a91c4e4c8f225bbc7975f9434fab042228c7acdc5ec9a32626ef.

Signal
- `nonReentrant` only guards same-contract functions; external call flow allows direct Vault calls.
- Vault `increasePosition` callable outside PositionRouter/Manager; avg price calc depends on routed path.
- GLP price depends on shorts’ avg price; manipulating avg price inflates GLP.

Exploit Steps
- 1) Use OrderBook entry point to start increase order.
- 2) Reenter and call Vault.increasePosition directly during execution, bypassing avg price update.
- 3) Set BTC short avg price to an abnormally low value (~$1.9k).
- 4) Open large short (~$15.39M); PnL calc makes GLP price spike.
- 5) Buy GLP at normal price, then redeem/sell inflated GLP; profit.

Key Code/Storage
- OrderBook: `createIncreaseOrder`/execute path (https://github.com/gmx-io/gmx-contracts/blob/master/contracts/core/OrderBook.sol#L874) nonReentrant only intra-contract.
- Vault: `increasePosition` callable directly; avg price calc expected via PositionRouter/Manager.
- GLP price uses shorts’ avg price; manipulated via reentrancy.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/gmx_exp.sol
- Announcement: https://x.com/GMX_IO/status/1927697775071801600
- Alert: https://x.com/GMX_IO/status/1927697775071801600
- Related pattern: cross-contract-reentrancy-price-skew.md
