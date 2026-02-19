Title: NGP spot-price oracle + fee sync manipulation (2025-09-18)

Summary
- Type: AMM spot price oracle manipulation combined with faulty fee/sync logic.
- Preconditions: `getPrice()` uses instantaneous USDT/NGP reserves from a single Pancake pair; `_update` deducts fees and calls `sync()` before completing seller transfer; DEAD address whitelisted to bypass buy/cooldown limits.
- Impact: attacker flash-loaned stablecoins, manipulated pool reserves to skew price, bypassed max buy limits via whitelisted DEAD, then sold with inflated price after premature fee/sync, draining ~ $2M liquidity.
- Representative tx: 0xc2066… (per incident report; victim 0xd2F26… on BSC).

Signal
- Oracle uses spot `reserveUSDT * 1e18 / reserveNGP` without TWAP/bounds.
- Sell path deducts fees and calls `sync()` before transferring user tokens to pair.
- Whitelist includes DEAD, bypassing buy/cooldown checks.

Exploit Steps
- 1) Flash-loan large USDT; manipulate NGP/USDT pool reserves to lower reported price, bypassing `maxBuyAmountInUsdt`.
- 2) Accumulate NGP (using DEAD whitelist to skip limits).
- 3) Sell NGP: fee logic mutates reserves and calls `sync` before user transfer, inflating price.
- 4) Extract USDT/BUSD liquidity at distorted price; repay loans, keep profit (~$2M).

Key Code/Storage
- `getPrice()` and `getTokenPrice()` read direct reserves from NGP-USDT pair.
- `_update()` on sell: fee deductions to market/burn/treasury/reward, `pair.sync()` before final transfer.
- Whitelist: DEAD address included; buy cooldown/max buy enforced except for whitelist.

Refs
- Analysis: https://solidityscan.substack.com/p/ngp-token-hack-analysis
- Attack tx: 0xc2066… (BSC)
- Victim token: NGP on BSC (0xd2F26…)
- Related pattern: permissionless-oracle-update.md, lp-skim-sync-reserve-desync.md
