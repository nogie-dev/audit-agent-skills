Title: Public ERC4626 mint without asset value check

Summary
- ERC4626 `mint(shares)` is exposed without override/access control, not payable, and does not verify assets/value provided. Internal `_deposit` can mint shares even when no assets received or pool liquidity exhausted.
- Impact: attacker mints vault/LST shares for free when swap-from-pool path fails/empty, draining backing assets via swaps/redemptions.

Signal
- Inheriting ERC4626 (upgradeable) without overriding `mint` to enforce payable/value/allowlist.
- `_deposit` has fallback minting path even if `_getAssetsFromPool` returns zero.
- Liquid pool can be drained first, then mint produces full shares.

Exploit Steps
- 1. Drain internal pool/liquidity so swap path yields nothing.
- 2. Call public `mint(shares)` with zero value (non-payable); fallback mints full shares.
- 3. Swap/redeem minted shares for base asset; profit.

Examples
- playbooks/cases/dex/metapool-mint-no-asset-2025.md
