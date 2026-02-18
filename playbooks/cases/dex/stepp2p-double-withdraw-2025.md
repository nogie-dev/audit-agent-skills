Title: Stepp2p cancel/modify double spend (2025-07-13)

Summary
- Type: state machine flaw allowing double transfer on sale order.
- Preconditions: `createSaleOrder` locks seller funds; `cancelSaleOrder` and `modifySaleOrder` both transfer tokens back without marking sale consumed; can call both on same saleId.
- Impact: attacker flash-loaned USDT, created a sale, then called cancel and modify on the same saleId to withdraw twice, netting ~43k USD.
- Representative tx: https://bscscan.com/tx/0xe94752783519da14315d47cde34da55496c39546813ef4624c94825e2d69c6a8.

Signal
- Sale order lifecycle functions that each transfer funds without mutual exclusion or state update preventing double execution.
- Modify function allows negative adjustment (isPositive=false) performing a refund, even after cancel.

Exploit Steps
- 1) Flash-loan 50k USDT.
- 2) `createSaleOrder(amount)` on Stepp2p.
- 3) Call `cancelSaleOrder(saleId)` (refund 1).
- 4) Call `modifySaleOrder(saleId, amount, false)` (refund 2) using same saleId.
- 5) Repay flash loan; keep remaining USDT.

Key Code/Storage
- Stepp2p: 0x99855380E5f48Db0a6BABeAe312B80885a816DCe (`cancelSaleOrder` and `modifySaleOrder` both transfer funds back).
- Pair: Pancake V3 USDC/USDT 0x4f31Fa980a675570939B737Ebdde0471a4Be40Eb (flash loan source).
- Token: USDT 0x55d398326f99059fF775485246999027B3197955.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/Stepp2p_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1946887946877149520
- Related pattern: overcredited-internal-balance-vs-transfer.md
