Title: NFT transfer rewards allow self-transfer farming

Summary
- NFT transfer hook rewards sender and receiver; when from==to, claimed state is reset then claimed again, enabling infinite self-transfer reward loops.
- Impact: attacker self-transfers same token to claim rewards repeatedly (within one tx via loop), draining reward pool.

Signal
- Transfer logic clears claimed snapshot/state before paying rewards; no `from != to` guard.
- No per-tx/per-block cap on rewards; rewards triggered on transfer even if ownership unchanged.
- `safeTransferFrom` callable by token holder without restrictions beyond ERC721.

Exploit Steps
- 1. Obtain NFT.
- 2. Loop `safeTransferFrom(holder, holder, tokenId)` many times in one tx.
- 3. Collect rewards credited each iteration.

Key Code/Storage
- Reward logic in transfer; claimed snapshots per address reset; rewards credited to both sender/receiver regardless of equality.

Refs
- Example: The Idols self-transfer reward farming 2025-01-14 (playbooks/cases/nft/idols-self-transfer-reward-2025.md)
