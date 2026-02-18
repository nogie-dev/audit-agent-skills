Title: Permissionless price anchor update consumed same-transaction

Summary
- State-changing oracle/AUM anchor can be updated by anyone and is consumed immediately for pricing without TWAP/delay/bounds.
- Impact: attacker skews underlying state (liquidity balances, spot prices), updates anchor, and drains pools relying on the inflated/deflated price in the same tx.

Signal
- `update*` or AUM/oracle refresh callable by anyone; no role checks.
- No TWAP/heartbeat; anchor used immediately by swaps/LP exits.
- Large intra-block price jumps; downstream pools lack bounds/circuit breakers.

Exploit Steps
- 1. Manipulate underlying state (flashloan to skew AMM/pool balances).
- 2. Call permissionless anchor/oracle update to bake manipulated state.
- 3. Consume the new price in swaps/withdrawals before state reverts.

Key Code/Storage
- Anchor update function (e.g., `updateTotalAum`, `poke`, `updatePrice`) writing shared price/AUM.
- Consumers (DEX/Curve pools, LP withdraws) reading anchor in same tx.

Refs
- Example: Makina share oracle manipulation 2026 (playbooks/cases/stablecoin/makina-share-oracle-manipulation-2026.md)
