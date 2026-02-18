Title: 1inch Settlement calldata offset overflow (2025-03-05)

Summary
- Type: calldata length/offset overflow in dynamic suffix handling enabling resolver overwrite.
- Preconditions: Settlement `_settleOrder` computes `interactionLength`/offset from calldata without bounds; attacker-controlled 32-byte `interactionLength` underflows offset math; resolver field appended to suffix can be overwritten.
- Impact: attacker crafted nested interactions so resolver became victim resolver contract, causing it to transfer funds; ~4.5M drained (partially returned).
- Representative tx: https://etherscan.io/tx/0x62734ce80311e64630a009dd101a967ea0a9c012fabbfce8eac90f0f4ca090d6.

Signal
- Assembly uses `interactionLength` directly in `add(add(ptr, interactionOffset), interactionLength)` with no range check.
- Calldata copy then mstore on calculated offset; dynamic suffix appended based on unchecked lengths.
- Resolver sanity depends on suffix but suffix location is attacker-influenced.

Exploit Steps
- 1. Craft orders with fake offsets (`interactionLengthOffset`, `interactionLength = -512`) and padding.
- 2. Encode fake interaction/suffix so copy/append overwrites resolver field with victim resolver address.
- 3. Settlement processes orders, calls victim resolver `resolveOrders`, which transfers USDC to attacker.
- 4. Loop nested interactions to satisfy struct expectations.

Key Code/Storage
- Settlement: 0xa88800cd213da5ae406ce248380802bd53b47647 `_settleOrder` assembly (calldata copy + suffix append).
- Resolver (victim): 0xb02F39e382c90160Eb816DE5e0E428ac771d77B5 transfers arbitrary tokens when called by Settlement.
- Attacker contract: 0x019BfC71D43c3492926D4A9a6C781F36706970c9 used to craft orders.

Refs
- Post-mortem: https://blog.decurity.io/yul-calldata-corruption-1inch-postmortem-a7ea7a53bfd9
- Analysis: https://x.com/DecurityHQ/status/1898069385199153610
- Dedaub decompile: https://app.dedaub.com/ethereum/address/0x019bfc71d43c3492926d4a9a6c781f36706970c9/decompiled
