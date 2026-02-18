Title: Signature Replay/Reuse

Summary
- Signatures accepted multiple times or across domains/contracts without nonce/domain separation.
- Preconditions: missing per-user nonce, weak permit implementation, chainID/domain mismatch.
- Impact: drain approvals, duplicate executions, unauthorized transfers/mints.
- Representative: Permit replay issues on various ERC20s; EIP-2612 mis-implementations.

Signal
- No per-user nonce increment; shared nonce across actions; nonces not stored on-chain.
- EIP-712 domain uses static chainId or missing contract-specific salt.
- Signature verified but no state change preventing reuse; off-by-one nonce checks.

Exploit Steps
- 1. Obtain signed message (legit intent or phishing).
- 2. Submit to target function where replay is possible (same or different chain).
- 3. Repeat execution or route to different contract that shares domain.

Key Code/Storage
- `permit`, meta-tx executors, multicall permits; nonce mapping; domain separator.
- Upgradeable contracts rebuilding domain separator incorrectly.
- Deadline logic without enforcement or unchecked after execution.

Refs
- https://eips.ethereum.org/EIPS/eip-2612
- https://mirror.xyz/samczsun.eth/uw2ZCenZDP7FpC18JNpD-sB4hRu9mY4FRbMa80N1V6c
