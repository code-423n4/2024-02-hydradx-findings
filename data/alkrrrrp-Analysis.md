## Recommendation on the Omnipool `withdraw_protocol_liquidity` extrinsic

The comment states the protocol liquidity is liquidity from sacrificed positions, but when the user withdraws liquidity with the `remove_liquidity` extrinsic if the spot price is smaller than the position price, some amount of the position share will give to the protocol, thus the protocol liquidity comes from both sacrificed positions and the user position.

The `withdraw_protocol_liquidity` takes a `price` argument to reconstruct a temporary position and uses `calculate_remove_liquidity_state_changes` to calculate the asset state change like normal liquidity withdrawal, and there are some potential issues:
- The protocol liquidity comes from both sacrificed positions and the user position, to reconstruct the position with the correct `price` the caller (`AuthorityOrigin`) needs to collect all the price of the liquidity that comes from the user position and call `withdraw_protocol_liquidity` one by one since they may have different price, this requires a lot of manual effort.
- If the given `price` is greater than the current spot price, then some amount of the liquidity will be kept in the protocol account and require another `withdraw_protocol_liquidity` to withdraw.
- If a wrong `price` is given it is possible some amount of Hub asset will be minted to the `dest` account by accident and the exact amount depends on the the difference between `price` and the spot price.

To avoid the manual effort, recommended to remove the `price` argument and use the spot price to reconstruct the position instead. In this way, there won't be any liquidity split and left in the protocol account or any Hub asset minted to the `dest` account, the caller just needs to specify the amount of share to withdraw.

## Recommendation on the Omnipool `MinimumTradingLimit` and `MinimumPoolLiquidity`

Both of these limits apply to the amount of the asset when trading or adding liquidity (removing liquidity doesn't check `MinimumPoolLiquidity`), while the price difference between assets can be large the limit is the same for all assets, e.g. the `MinimumTradingLimit` for BTC maybe $100 while for TKN is $0.001, this essentially making the limit negligible for low price asset.

Recommended to define `MinimumTradingLimit` and `MinimumPoolLiquidity` in the amount of Hub asset so the limit in value is the same for all assets.


### Time spent:
25 hours