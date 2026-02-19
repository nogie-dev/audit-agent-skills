Title: MulticallWithETH arbitrary call drains USDC (2025-07-09)

Summary
- Type: missing access control on multicall executor.
- Preconditions: `multicall(Call[])` (selector 0xc9586258) on victim contract 0x3DA0… is public and forwards arbitrary callData/value to targets; no sender restriction.
- Impact: attacker used multicall to invoke USDC `transferFrom` from a funded address to self, stealing ~10k USDT-equivalent.
- Representative tx: https://bscscan.com/tx/0x6da7be6edf3176c7c4b15064937ee7148031f92a4b72043ae80a2b3403ab6302.

Signal
- Public multicall that forwards arbitrary `target` and `callData`, no whitelist/owner check.
- Victim holds/authorized to spend stablecoin on behalf of another address.

Exploit Steps
- 1) Craft `Call` entry targeting USDC `transferFrom(victim_funder, attacker, balanceOf(funder))`.
- 2) Call victim `multicall` with the crafted Call array.
- 3) USDC transferred to attacker; profit.

Key Code/Storage
- Victim: 0x3DA0F00d5c4E544924bC7282E18497C4A4c92046 (BSC) exposes multicall (0xc9586258) with `Call{target, callData, value, allowFailure}`.
- Token: USDC 0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d.
- Funder address drained: 0xfb0De204791110Caa5535aeDf4E71dF5bA68A581.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/MulticallWithETH_exp.sol
- Alert: https://bscscan.com/tx/0x6da7be6edf3176c7c4b15064937ee7148031f92a4b72043ae80a2b3403ab6302
- Related pattern: public-multicall-arbitrary-target.md
