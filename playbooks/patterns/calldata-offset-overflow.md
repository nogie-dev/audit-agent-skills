Title: Calldata length/offset overflow in dynamic suffix handling

Summary
- Assembly builds new calldata buffers using attacker-controlled length/offset fields (e.g., interactionLength) without bounds, allowing under/overflow to reposition appended data.
- Impact: appended fields (resolver/fee/tokens) overwritten; pointers move into attacker-controlled padding; enables resolver hijack or arbitrary target execution.

Signal
- `add(add(ptr, interactionOffset), interactionLength)` style calculations on untrusted calldata lengths; mstore to computed offsets; no sanity checks.
- Dynamic suffix copied/appended after calldatacopy with attacker-supplied lengths; 32-byte interactionLength used directly.

Exploit Steps
- 1. Craft calldata with extreme length (e.g., 0xffff...fe00) and padding.
- 2. Cause append logic to write suffix over attacker-chosen region (e.g., fake interaction).
- 3. Overwrite sensitive fields (resolver, target) and trigger call.

Key Code/Storage
- Assembly copy + suffix append using unchecked length/offset; resolver/target fields appended from attacker-controlled section.

Refs
- Example: 1inch Settlement calldata offset overflow 2025-03-05 (playbooks/cases/dex/1inch-settlement-calldata-overflow-2025.md)
