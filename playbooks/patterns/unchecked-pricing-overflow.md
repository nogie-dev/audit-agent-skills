Title: Unchecked arithmetic in pricing/mint curves

Summary
- Raw multiplication/division on user-controlled amount, reserve, and supply/params without overflow checks (pre-0.8 Solidity or unchecked blocks).
- Impact: price calc wraps to 0 or tiny value, allowing free/cheap mints or redemptions; can drain reserves in one tx.

Signal
- Solidity <0.8 or `unchecked` blocks in pricing; custom math helpers lacking SafeMath/checked ops.
- Large-term products (amount * reserve * supply) with no input caps; return values not sanity-checked (price can be 0).

Exploit Steps
- 1. Choose large input to force overflow in numerator/denominator of price function.
- 2. Call purchase/mint with returned low/zero price to mint cheaply.
- 3. Redeem/sell minted asset for backing collateral; repeat.

Key Code/Storage
- Pricing functions (e.g., `getPurchasePrice`, bonding curves) using raw mul/div; config params (e.g., THETA) compounding multipliers.

Refs
- Example: Truebit 2026 price overflow (playbooks/cases/token/truebit-integer-overflow-2026.md)
