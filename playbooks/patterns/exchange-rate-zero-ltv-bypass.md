Title: Exchange rate floors to zero, solvency check bypass

Summary
- Oracle/exchange-rate computation divides by an inflated price and floors to 0; solvency/health checks multiply by this rate, making any position appear fully solvent.
- Impact: attacker can borrow up to caps with minimal collateral when `_exchangeRate` hits 0.

Signal
- Exchange rate code like `exchangeRate = A / price` without min bound; price can be attacker-inflated.
- Solvency uses `_amount * exchangeRate` directly; no check for zero/underflow.
- Vault empty or price function uses `convertToAssets(1e18)` on extreme price, producing overflowed or huge values.

Exploit Steps
- Seed vault to make price extreme (e.g., empty vault with huge donation; mint 1 wei share).
- Force `exchangeRate` to floor to 0 in oracle.
- Call borrow; `_ltv` becomes 0 so check passes; drain up to debt cap.

Key Code/Storage
- Oracle math: `price = convertToAssets(1e18)`; `exchangeRate = 1e36 / price` (floors to 0).
- Solvency: `_ltv = borrowerAmount * exchangeRate / collateral; return _ltv <= maxLTV` with no zero guard.

Examples
- playbooks/cases/lending/resupplyfi-exchange-rate-zero-2025.md
