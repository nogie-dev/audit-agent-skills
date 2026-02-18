Audit playbooks, protocol notes, and tooling to assist an LLM agent during smart contract reviews.

## Structure
- `playbooks/` reusable knowledge
  - `checklists/` vulnerability type checklists
  - `cases/` incident summaries by domain
  - `patterns/` code-level risk patterns
- `protocol-notes/` per-protocol inputs and findings
- `tooling/` PoC and query helpers

## Usage flow
1) Add protocol context to `protocol-notes/<protocol>/summary.md` and list expected risks in `audit-plan.md`.
2) Link relevant items from `playbooks/checklists` (and optionally `cases`/`patterns`) inside the audit plan.
3) When reviewing code, pull only needed sections (Summary/Signal/Exploit Steps) into the agent context.
4) Record outcomes in `protocol-notes/<protocol>/findings.md`, linking back to checklists or cases.

## Files to copy when starting a new protocol
- Duplicate `protocol-notes/protocol-template/` and rename to your protocol.
- Add or extend checklists/cases/patterns as needed; keep them short and bulleted.

## Notes for agent prompts
- Lead with the protocol summary and audit plan links.
- Keep excerpts tight: `Summary`, `Signal`, `Exploit Steps`, plus any key `Key Code/Storage` snippets.
- Include PoC/queries from `tooling/` only when necessary to test a hypothesis.

## Case sources (2025)
| Case | Primary refs |
| --- | --- |
| UsualMoney USD0+/sUSDS price mis-sync (2025-05-27) | [PoC](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/UsualMoney_exp.sol), [BlockSec alert](https://x.com/BlockSecTeam/status/1927601457815040283) |
| KRC LP reserve desync (2025-05-18) | [PoC](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/KRCToken_pair_exp.sol), [Certik alert](https://x.com/CertikAIAgent/status/1924280794916536765) |
| MBU deposit mint mispricing (2025-05-11) | [PoC](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/MBUToken_exp.sol), [TenArmor alert](https://x.com/TenArmorAlert/status/1921474575965065701) |
| Nalakuvara LotteryTicket50 burn flaw (2025-05-09) | [PoC](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/Nalakuvara_LotteryTicket50_exp.sol), [TenArmor alert](https://x.com/TenArmorAlert/status/1920816516653617318) |
| RICE masterContractApproval bypass (2025-05-24) | [PoC](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/RICE_exp.sol), [TenArmor alert](https://x.com/TenArmorAlert/status/1926461662644633770) |
| Unwarp WETH unwrap w/o auth (2025-05-14) | [PoC](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/Unwarp_exp.sol), [Alert](https://x.com/TenArmorAlert/status/1921365719664957728) |
| YDT backdoor transfer drains LP (2025-05-26) | [PoC](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/YDTtoken_exp.sol), [TenArmor alert](https://x.com/TenArmorAlert/status/1926587721885040686) |

## Case sources
| Case | Source |
| --- | --- |
| Truebit price overflow (2026-01) | https://www.certik.com/zh-CN/resources/blog/truebit-incident-analysis |
| Makina share oracle manipulation (2026-01-20) | https://x.com/nn0b0dyyy |
| LAURA auto-liquidity K exploit (2025-01-01) | https://twitter.com/rotcivegaf |
| SOR staking reward replay (2025-01-04) | https://x.com/TenArmorAlert/status/1875582709512188394 |
| IPC timelock bypass (2025-01) | https://x.com/TenArmorAlert/status/1876669657866015230 |
| FortuneWheel swapProfitFees ACL missing (2025-01-10) | https://www.cnblogs.com/ACaiGarden/p/18664999 |
| UniLend stale balance health factor (2025-01-12) | https://slowmist.medium.com/analysis-of-the-unilend-hack-90022fa35a54 |
| yETH virtual balance math manipulation (2025-12-01) | https://github.com/banteg/yeth-exploit/blob/main/report.pdf |
| Mosca join overcredit drain (2025-01-13) | https://x.com/TenArmorAlert/status/1878699517450883407 |
| Idols NFT self-transfer reward farming (2025-01-14) | https://x.com/TenArmorAlert/status/1879376744161132981 |
| AST token faulty transfer skim exploit (2025-01-21) | https://medium.com/@joichiro.sai/ast-token-hack-how-a-faulty-transfer-logic-led-to-a-65k-exploit-da75aed59a43 |
| Four.meme pool hijack price skew (2025-02-11) | https://www.chaincatcher.com/en/article/2167296 |
| 1inch Settlement calldata offset overflow (2025-03-05) | https://blog.decurity.io/yul-calldata-corruption-1inch-postmortem-a7ea7a53bfd9 |
| SBR skim/sync reserve desync (2025-03-07) | https://x.com/0xfirmanregar/status/2019768070810812520 |
| UNI skim/sync reserve manipulation (2025-03-07) | https://x.com/CertiKAlert/status/1897973904653607330 |
| BBX burn-gap bypass (2025-03-20) | https://blog.solidityscan.com/bbx-token-hack-analysis-f2e962c00ee5 |
| wKeyDAO unprotected buy (2025-03-16) | https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/wKeyDAO_exp.sol |
| LeverageSIR slot collision & callback abuse (2025-03-30) | https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/LeverageSIR_exp.sol |
| Alkimiya SilicaPools unsafe cast (2025-03-28) | https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-03/Alkimiya_io_exp.sol |
| YBToken no slippage protection (2025-04-16) | https://x.com/CertiKAlert/status/1912684902664782087 |
| Laundromat unauthenticated withdraw (2025-04-08) | https://x.com/TenArmorAlert/status/1909814943290884596 |
| AIRWA burnRate unprotected (2025-04-04) | https://x.com/TenArmorAlert/status/1908086092772900909 |
| Lifeprotocol buy/sell loop drain (2025-04-26) | https://bscscan.com/tx/0x487fb71e3d2574e747c67a45971ec3966d275d0069d4f9da6d43901401f8f3c0 |
| ImpermaxV3 flashloan oracle manipulation (2025-04-26) | https://medium.com/@quillaudits/how-impermax-v3-lost-300k-in-a-flashloan-attack-35b02d0cf152 |
| BTNFT reward claim replay (2025-04-18) | https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-04/BTNFT_exp.sol |
| Unverified 0x6077 access control (2025-04-11) | https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-04/Unverified_6077_exp.sol |
| SizeCredit leverageUp arbitrary call (2025-08-15) | https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main/src/test/2025-08/SizeCredit_exp.sol |
| Corkprotocol access control abuse (2025-05-28) | https://x.com/SlowMist_Team/status/1928100756156194955 |
