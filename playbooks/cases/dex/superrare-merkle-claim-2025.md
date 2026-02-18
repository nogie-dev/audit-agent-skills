Title: SuperRare Merkle root overwrite claim (2025-07-21)

Summary
- Type: insufficient access control on Merkle drop root.
- Preconditions: ERC1967 proxy exposes `updateMerkleRoot(bytes32)` publicly; `claim(amount, proof)` does not bind caller to proof/owner; contract holds large RARE token balance.
- Impact: attacker overwrote Merkle root with arbitrary value, then called `claim` with empty proof and full contract balance amount, stealing ~730k USD of RARE.
- Representative tx: https://app.blocksec.com/explorer/tx/eth/0xd813751bfb98a51912b8394b5856ae4515be6a9c6e5583e06b41d9255ba6e3c1.

Signal
- Airdrop/claim contract allows public `updateMerkleRoot`.
- Claim lacks binding between msg.sender and proof; accepts empty proof.
- Large token balance in contract.

Exploit Steps
- 1) Deploy/etch attack contract.
- 2) Call `updateMerkleRoot(fakeRoot)` on proxy.
- 3) Call `claim(amount=stakingContractBalance, proof=[])` to transfer all RARE to attacker.

Key Code/Storage
- Proxy: 0x3f4D749675B3e48bCCd932033808a7079328Eb48 (implements updateMerkleRoot, claim).
- Token: RARE 0xba5BDe662c17e2aDFF1075610382B9B691296350.
- Vulnerable contract holds staking/airdrop balance: ~11.9M RARE.

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-07/SuperRare_exp.sol
- Alert: https://x.com/SlowMist_Team/status/1949770231733530682
- Analysis: https://blog.solidityscan.com/superrare-hack-analysis-488d544d89e0
- Related pattern: treasury-swap-no-access-control.md
