Title: AST token faulty transfer logic with LP skim (2025-01-21)

Summary
- Type: nonstandard ERC20 transfer bug causing LP skim to overpay.
- Preconditions: AST proxy implements custom transfer that misaccounts balances; sending AST into LP then calling `skim` returns more than deposited; fee-on-transfer/logic interacts poorly with skim/sync.
- Impact: attacker flashloaned BUSD, swapped to AST, injected AST into LP, called `skim` to pull out excess AST, swapped back to BUSD for ~$65K profit.
- Representative tx: https://bscscan.com/tx/0x80dd9362d211722b578af72d551f0a68e0dc1b1e077805353970b2f65e793927.

Signal
- Token uses proxy (ERC1967) with custom transfer logic; not vanilla ERC20.
- LP interactions rely on `transfer` then `skim`/`sync`; token miscalculates balances leading to over-withdrawal.
- Fee-on-transfer or balance adjustments not mirrored in LP reserve accounting.

Exploit Steps
- 1. Flashloan 30M BUSD.
- 2. Swap all BUSD→AST to the token proxy.
- 3. Manually transfer AST (and small BUSD) into BUSD/AST pair.
- 4. Call `skim` to pull out extra AST due to faulty transfer accounting; `sync` reserves.
- 5. Swap drained AST back to BUSD; repay flashloan; keep profit.

Key Code/Storage
- Token: 0xc10E0319337c7F83342424Df72e73a70A29579B2 (proxy at 0xc8B9817eB65B7d7e85325f23A60D5839d14F9Ce4) with custom transfer logic.
- LP: 0x5ffEc8523A42BE78B1Ad1244fA526f14B64bA47a (Pancake BUSD/AST pair).
- Flashloan pool: 0x36696169C63e42cd08ce11f5deeBbCeBae652050.

Refs
- Analysis: https://medium.com/@joichiro.sai/ast-token-hack-how-a-faulty-transfer-logic-led-to-a-65k-exploit-da75aed59a43
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-01/Ast_exp.sol
- Attacker: https://bscscan.com/address/0x56f77AdC522BFfebB3AF0669564122933AB5EA4f
- Attack contract: https://bscscan.com/address/0xaaE196b6E3f3Ee34405e857e7bfb05D74c5cf775
