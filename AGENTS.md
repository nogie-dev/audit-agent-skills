Agent playbook for adding exploit knowledge (DeFiHackLabs-driven)

Scope
- Use this repo’s structure: playbooks/cases, playbooks/patterns, README case sources.
- Keep content detection-focused (no mitigations), grounded in on-chain/PoC sources.

Workflow
1) Discover incidents
- Fetch index (example): `curl -L https://raw.githubusercontent.com/SunWeb3Sec/DeFiHackLabs/main/past/2025/README.md` (adjust year).
- Search for target incident: `grep -i "<name>"` in the README.
- Note contract/tx links and PoC path (e.g., `src/test/<year>-<mm>/<Name>_exp.sol`).

2) Collect source data
- Download PoC: `curl -L https://raw.githubusercontent.com/SunWeb3Sec/DeFiHackLabs/main/src/test/<year>-<mm>/<Name>_exp.sol`.
- Collect on-chain refs (tx, attacker, contracts) from README/PoC and any linked posts.

3) Add case file
- Path: `playbooks/cases/<domain>/<short-name>-<yyyy>.md`.
- Include: Title, Summary (type/preconditions/impact), Signal, Exploit Steps, Key Code/Storage, Refs (tx/attacker/contract/analysis/PoC).
- No Fix/Detect section.

4) Add pattern (if reusable)
- Path: `playbooks/patterns/<pattern-name>.md`.
- Capture generalized behavior (Summary/Signal/Exploit Steps/Key Code/Storage/Refs to case).
- Only create a new pattern when behavior is not covered by existing ones (e.g., multi-module init without auth, deflationary token LP drift, unauthenticated withdraw signature replay). Otherwise map to an existing pattern.

5) Update README
- Append to Case sources table: `| <case title> (YYYY-MM-DD) | <primary analysis/alert link> |`.

6) Verify
- `find playbooks/cases -maxdepth 3 -type f | sort`
- `find playbooks/patterns -type f | sort`
- Optional: `grep -R "Fix/Detect" playbooks` should return nothing.

Notes
- Keep content concise, ASCII only. No mitigations.
- When network blocked, ask/attempt with justification; report if 404 or denied.
- Prefer incident-by-incident (prioritize high impact) rather than bulk dump.
