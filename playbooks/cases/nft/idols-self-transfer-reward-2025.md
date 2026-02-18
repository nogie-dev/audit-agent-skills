Title: The Idols self-transfer reward farming (2025-01-14)

Summary
- Type: reward on NFT transfer misapplied when sender==receiver.
- Preconditions: `safeTransferFrom` rewards both sender and receiver; when sender==receiver, claimed snapshot is cleared then reward is claimed for same address; no guard against self-transfer loops; constructor bypasses `Address.isContract` check.
- Impact: attacker self-transferred the same NFT repeatedly to claim stETH rewards each iteration; ~97 stETH (~$330K) drained over repeated txs.
- Representative tx: https://etherscan.io/tx/0x5e989304b1fb61ea0652db4d0f9476b8882f27191c1f1d2841f8977cb8c5284c.

Signal
- Transfer hook pays rewards to both from/to without checking from!=to.
- Claimed snapshot for sender is deleted before claiming, allowing re-claim when from==to.
- No limit on transfer count per block/tx; constructor mint/transfer bypasses contract check.

Exploit Steps
- 1. Deploy attack contract; receive target NFT via `safeTransferFrom`.
- 2. Loop `safeTransferFrom(address(this), address(this), tokenId)` many times; each loop clears snapshot then reclaims reward for same address.
- 3. Collect stETH rewards and send NFT back to EOAs; repeat across txs.

Key Code/Storage
- Contract: 0x439cac149b935ae1d726569800972e1669d17094 (The Idols NFT).
- Functions: transfer logic that clears claimed snapshots and pays rewards to both sender/receiver; lacks `from != to` guard.
- State: `claimedSnapshots` per address; `allocatedStethRewards`, `rewardPerGod` used to compute rewards.

Refs
- Alert: https://x.com/TenArmorAlert/status/1879376744161132981
- Post-mortem: https://rekt.news/theidolsnft-rekt
- Attacker: https://etherscan.io/address/0xe546480138d50bb841b204691c39cc514858d101
- Attack contract: https://etherscan.io/address/0x22d22134612c0741ebdb3b74a58842d6e74e3b16
