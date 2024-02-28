# Low / non-critical findings

## Omnipool

1. In the `add_token` extrinsic, both the `amount` and `reserve` variables are the same value query from `T::Currency::free_balance(asset, &Self::protocol_account())`, the `reserve` can be replaced by `amount` to avoid one database read.

2. In the `add_liquidity` extrinsic, when calculating the `hub_reserve_ratio`, the `current_hub_asset_liquidity` variable can be used to avoid one database read.



## Stableswap

1. Missing `InsufficientLiquidityRemaining` check in the `withdraw_asset_amount` extrinsic

The user can use `withdraw_asset_amount` to withdraw shares from a pool and leave the pool with `0 < remaining_liquidity < MinPoolLiquidity`.

2. Missing `MinTradingLimit` check in the `add_liquidity_shares` extrinsic

3. The `added_amounts` variable in `fn do_add_liquidity` is never used

4. The `InvalidAmplification` check in the `update_amplification` extrinsic can move to the beginning of the function as a short-circuit.



### EMA Oracle

1. The oracle data of a removed token of the Omnipool stays forever in the runtime state

As a result, the `Oracles` storage in the `ema-oracle` pallet will only grow and never decrease. Recommended adding another hook like `on_token_remove` to remove the oracle date for the removed token.




