Title: Balancer V2 Composable Stable Pool invariant rounding exploit (2025-11-03)

Summary
- Type: precision/rounding loss in stable invariant misprices BPT.
- Preconditions: Stable math upscales with mulDown and downscales with divUp/divDown; `_swapGivenOut` rounds down amountOut before solving; precision loss shrinks invariant D; BPT price = D/totalSupply.
- Impact: attacker crafted batchSwap with tuned amounts/iterations to deflate D, then redeemed at deflated BPT price; coordinated attacks drained ~$125M+ across chains.
- Representative tx: https://app.blocksec.com/explorer/tx/arbitrum/0x7da32ebc615d0f29a24cacf9d18254bea3a2c730084c690ee40238b1d8b55773 (stage 1), profits realized in follow-up txs.

Signal
- Stable pool math uses one-directional rounding on upscale; comment says impact minimal.
- GivenOut swaps under-charge input due to rounding; invariant decreases.
- BPT price derived from invariant without sanity bounds.

Exploit Steps
- 1) Compute crafted amounts near scaling/rounding boundaries (aux contract).
- 2) Batch swap to push balances to boundary and trigger mulDown precision loss, lowering invariant D.
- 3) Swap back/redeem BPT at deflated price; withdraw profits in later tx.

Key Code/Storage
- ComposableStablePool/BaseGeneralPool `_swapGivenOut` → `_upscale` (mulDown) → solve for amountIn.
- Rounding inconsistency (mulDown vs divUp/Down) underestimates amountIn/out; deflates D.
- BPT price depends on D/totalSupply.

Refs
- Analysis: https://blocksec.com/blog/in-depth-analysis-the-balancer-v2-exploit
- Alerts: https://x.com/Phalcon_xyz/status/1985262010347696312 , https://x.com/Balancer/status/1985390307245244573
- Related pattern: invariant-precision-rounding-loss.md
