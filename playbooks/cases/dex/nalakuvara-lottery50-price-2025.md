Title: Nalakuvara LotteryTicket50 burn payout flaw (2025-05-09)

Summary
- Type: unchecked redemption loop drains USDC.
- Preconditions: `transferToken` mints LotteryTicket50; `DestructionOfLotteryTickets` pays out USDC without verifying ticket ownership/allowance, allowing repeated burns for the same minted tickets.
- Impact: attacker flash-loaned USDC on Base and drained ~105,470 USDC.
- Representative tx: https://basescan.org/tx/0x16a99aef4fab36c84ba4616668a03a5b37caa12e2fc48923dba4e711d2094699.

Signal
- Redemption/burn function lacks balance checks; only reduces an internal counter.
- Large looped calls to `DestructionOfLotteryTickets` in a single tx.
- Payout token (USDC) held by the swap contract rather than the ticket token.

Exploit Steps
- 1) Flash-loan ~2.93M USDC from a Uniswap V3 pool.
- 2) Swap a portion for Nalakuvara tokens via a V2 pair to satisfy path requirements.
- 3) Call `transferToken(130_000_000_000)` to mint tickets, then call `DestructionOfLotteryTickets(20_000_000)` about 130 times to receive USDC each iteration.
- 4) Swap Nalakuvara back to USDC, repay flash loan, keep the surplus.

Key Code/Storage
- Lottery swap: 0x172119155a48DE766B126de95c2cb331D3A5c7C2 (`transferToken`, `DestructionOfLotteryTickets`).
- Ticket token: 0xF9260Bb78d16286270e123642ca3DE1F2289783b.
- Pair: 0xaDcaaB077f636d74fd50FDa7f44ad41e20A21FEE (V2 Nalakuvara/USDC).

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-05/Nalakuvara_LotteryTicket50_exp.sol
- Alert: https://x.com/TenArmorAlert/status/1920816516653617318
- Related pattern: reward-replay-missing-claim-tracking.md
