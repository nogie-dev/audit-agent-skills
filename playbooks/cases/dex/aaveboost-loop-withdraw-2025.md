Title: AaveBoost proxyDeposit loop drains stkAAVE (2025-06-06)

Summary
- Type: logic flaw in deposit accounting; repeated zero-deposit calls unlock withdraw.
- Preconditions: `proxyDeposit` callable by anyone; internal limit based on AaveBoost token balance misused; no stake/unstake auth; `withdraw` callable after threshold.
- Impact: attacker looped `proxyDeposit` ~163 times to satisfy limit, then withdrew stkAAVE from Aave pool and transferred to self (~14.8K USD).
- Representative tx: https://app.blocksec.com/explorer/tx/eth/0xc4ef3b5e39d862ffcb8ff591fbb587f89d9d4ab56aec70cfb15831782239c0ce.

Signal
- Public `proxyDeposit(address token, address onBehalf, uint128 amount)` callable with amount=0.
- Loop/limit derived from token balance; no per-user accounting; withdraw lacks auth.
- Uses upgradeable proxy token (AAVE) balances as gating but not locking funds.

Exploit Steps
- 1) Compute `limit = balanceOf(AaveBoost) / (3e17)` (~163).
- 2) Loop `proxyDeposit(AAVE token, attacker, 0)` until `limit` reached; no funds sent.
- 3) Call `withdraw(token=AAVE, to=attacker)` via AavePool to pull stkAAVE held.
- 4) Transfer received AAVE to attacker.

Key Code/Storage
- AaveBoost: 0xd2933c86216dC0c938FfAFEca3C8a2D6e633e2cA (`proxyDeposit` no auth, uses token balance as loop counter).
- Token: stkAAVE proxy 0x7Fc66500c84A76Ad7e9c93437bFc5Ac33E2DDaE9.
- Pool: 0xf36F3976f288b2B4903aca8c177efC019b81D88B (`withdraw` callable by attacker).

Refs
- PoC: https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/2025-06/AAVEBoost_exp.sol
- Alert: https://x.com/CertiKAlert/status/1933011428157563188
- Related pattern: overcredited-internal-balance-vs-transfer.md
