## [LOW] stableswap: Attackers can prevent the first add_liquidity by transferring one token to the pool directly 

When a user first calls `add_liquidity`, the function will fail in `calculate_shares` if there are non-zero tokens in the pool (initial_reserves should be all zero or all non-zero). So an attacker can front-run the `add_liquidity` after the pool is created, and then the LP will suffer a DoS.

The LP can mitigate this by sending some dust to the zero assets in the pool and then calling the `add_liquidity`, but that's unexpected behavior for the users.

## [LOW] Function-Level Swap DoC invariants should consider the fees

https://github.com/galacticcouncil/HydraDX-security/blob/main/invariants/function-level/Omnipool.md#swap
- $R_iQ_i$ should be invariant, but one is calculated from the other. If e.g. $R_i^+$ is calculated it may have error up to $1$, in which case the product $R_i^+Q_i^+$ may have error up to $Q_i^+$. If $Q_i^+$ is calculated, then the product has error up to $R_i^+$. Thus we should always be able to bound the error by $max(R_i^+,Q_i^+)$, giving us

$$
R_i Q_i + max(R_i^+, Q_i^+) \geq R_i^+ Q_i^+
$$

However, we didn't consider the fees here. For example, there are 100:100 lrna:A in the pool, the asset fee is 20%, and Alice wants to buy 30 A with lrna. In that case, the asset fee will be 7+1=8 A. And Alice needs to sell 61 lrna (100*30/50 + 1).  In `process_trade_fee`, 8-1=7 A fees can be transferred out at most. So the subpool will be 161:63. And 100*100 + 161 > 161*63 > 100*100, which is correct. However, suppose referrals_used is 4, so only 3 A asset fees are transferred out, the subpool will be 161:67, now 100*100 + 161 is not bigger than 161*67.


## [LOW] weight cap can be bypassed by removing other tokens liquidity

A malicious attacker can bypass the weight cap in a block restriction by:

1. Provide liquidity to asset A until it reaches the weight cap.
2. Remove all the liquidity of other assets until the trading volume protection.
3. Now the actual weight of asset A is above the weight cap.

## [INFO] circuit-breaker:validate_limit should make sure numerator <= denominator

There is no check if `numerator <= denominator` in circuit-breaker:validate_limit(), should add one.

## [INFO] stableswap: calculate_out/in_amount should make sure the pool has been initialzed (shares > 0)

Otherwise, it will fail later in calculate_out_given_in.

## [INFO] omnipool: withdraw_protocol_liquidity weight should add on_liquidity_changed_weight()

Only callable by AuthorityOrigin, so not a big problem.
