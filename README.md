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
