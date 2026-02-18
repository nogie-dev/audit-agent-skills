Helper scripts for PoC and data pulls. Keep them minimal and parameterized.

- `poc/`: Foundry/Hardhat PoCs or test harnesses. Use env vars for RPC/keys.
- `queries/`: RPC/subgraph/Dune-style query snippets; note dependencies.

Usage
- Prefer reproducible commands (e.g., `forge test -m <name>`).
- Document required env vars and expected outputs inline.
