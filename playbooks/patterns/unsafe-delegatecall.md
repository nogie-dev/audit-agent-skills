Title: Unsafe delegatecall entrypoints

Summary
- Delegatecall into user-controlled target without strict allowlist or selector filtering.
- Preconditions: contract holds value/state; target address controllable; storage layout compatible enough to clobber.
- Impact: storage takeover, fund drain, role escalations.

Signal
- `delegatecall` on user param; upgradeable proxies exposing fallback without whitelist; plugin systems lacking auth.

Exploit Steps
- 1. Supply crafted target contract with malicious logic.
- 2. Invoke entrypoint that delegatecalls into target.
- 3. Modify storage (owner, balances) or transfer funds.

Key Code/Storage
- Fallbacks/plug-ins; storage slots for owner/admin/implementations; call data routing.

Refs
- https://blog.openzeppelin.com/proxy-patterns
