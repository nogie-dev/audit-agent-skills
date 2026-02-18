Title: Buggy token transfer causes LP skim overpay

Summary
- Custom ERC20 transfer misaccounts balances (fee-on-transfer/logic bug), so sending tokens to LP then calling `skim` returns more than deposited.
- Impact: attacker can inflate token balance via flashloan, inject into LP, `skim` to over-withdraw, and swap out for profit.

Signal
- Proxy/token with nonstandard transfer; balances adjusted in ways not aligned with AMM reserves.
- LP interactions involve manual transfer + `skim`/`sync`; over-withdraw possible.
- Fee-on-transfer or dual bookkeeping paths in transfer code.

Exploit Steps
- 1. Obtain tokens (e.g., flashloan + swap into target token).
- 2. Transfer tokens into LP; call `skim`/`sync` to pull extra tokens granted by faulty transfer logic.
- 3. Swap drained tokens back to base asset and exit.

Key Code/Storage
- Token transfer logic with custom deductions/additions; LP pair `skim`/`sync` operations.

Refs
- Example: AST token faulty transfer skim exploit 2025-01-21 (playbooks/cases/token/ast-transfer-bug-2025.md)
- Example: FPC burn-on-transfer mispricing 2025-07-05 (playbooks/cases/dex/fpc-deflation-burn-2025.md)
