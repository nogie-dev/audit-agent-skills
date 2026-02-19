Title: Public multicall executes arbitrary targets

Summary
- Multicall/execute function accepts arbitrary targets/callData/value and is public; no whitelist or owner check. Lets attacker perform any calls using contract context/allowances.
- Impact: transferFrom/drain of held/authorized tokens, reentrancy into other protocols, or arbitrary state changes via the victim contract.

Signal
- Function signature like `multicall(Call[] calls)` or `execute(target, data, value)` with no access control.
- Calls forwarded with `call`/`delegatecall` using msg.sender-supplied target/data.

Exploit Steps
- 1. Craft call(s) to a token `transferFrom(funder, attacker, balance)` or other sensitive function.
- 2. Invoke public multicall, executing crafted calls in victim context.
- 3. Receive tokens/funds drained via victim’s allowances/holdings.

Examples
- playbooks/cases/dex/multicallwitheth-no-auth-2025.md
