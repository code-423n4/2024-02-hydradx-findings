# Low
## [L-01] Formula for $\Delta b$ is not implemented according to documentation
[OmipoolMath Line 344](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/math.rs#L344)

**Issue Description:**

According to the Omnipool [documentation](https://docs.hydradx.io/omnipool_impermanent_loss), the formula for calculating $\Delta b$ (the amount of protocol shares) in the case of removed liquidity with the variables Pi (current price), $P\alpha$ (old price) and $S\alpha$ (shares removed) is:

$$\Delta B = max(\frac{Pi-P\alpha}{Pi+P\alpha} * (- S\alpha),0)$$

This is not the way the function is implemented in the code. The function is implemented in code as:

$$\Delta b = \frac{P\alpha * currentReserveOfAsset - currentHubReserves}{P\alpha * currentReserveOfAsset + currentHubReserves} * S\alpha + 1$$

This could be further simplified by dividing the upper and lower term of the fraction by `currentReserveOfAsset` and then splitting at the -/+.

$$\Delta b = \frac{P\alpha * \frac{currentReserveOfAsset}{currentReserveOfAsset} - \frac{ currentHubReserves}{currentReserveOfAsset}}{P\alpha * \frac{currentReserveOfAsset}{currentReserveOfAsset} + \frac{ currentHubReserves}{currentReserveOfAsset}} * S\alpha + 1$$
Now this can be simplified by removing the $\frac{currentReserveOfAsset}{currentReserveOfAsset}$ as it is equal to 1 and replacing $\frac{ currentHubReserves}{currentReserveOfAsset}$ with Pi as it would be the way to calculate the current price.

$$\Delta b = \frac{P\alpha  - Pi}{P\alpha + Pi} * S\alpha + 1$$
If one now adds the negation added in the original form one ends up with a formula pretty close to the documented one.

$$\Delta b = \frac{Pi - P\alpha }{Pi + P\alpha} * (- S\alpha) + 1$$
As we know that $Pi < P\alpha$ and that $S\alpha$ is a positive value, the result of $\frac{Pi - P\alpha }{Pi + P\alpha} * - S\alpha$  could potentially go very close to zero, but could never be negative. So by adding the +1 at the end, the minimum we could reach (with rounding) is 1 not 0. 

**Recommended Mitigation Steps**

As rounding in the favor of the protocol (by adding 1) should be intended behavior, the documentation of the formula is incorrect and needs to be adapted. The correct implemented formula is:

$$\Delta B = max(\frac{Pi-P\alpha}{Pi+P\alpha} * (- S\alpha),1)$$

This formula should also be documented.

## [L-02] Missing check for swap to 0 when selling hub asset
[omnipool/src/lib.rs Line 1704](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs#L1704)

**Issue Description:**

The omnipool swapping mechanism uses the `sell()` function so that a user can swap tokens and wants to specify how many token he wants to sell. One of the safeguards in this functions is the check if the user would get 0 tokens out and throwing an `ZeroAmountOut` error in that case, as this would mean a 100% loss on the swap for the user.

```rust
ensure!(
	*state_changes.asset_out.delta_reserve > Balance::zero(),
	Error::<T>::ZeroAmountOut
);
```

When the user wants to sell the Hub Asset Token for another token, the underlying `sell_hub_asset()` function is used. Unfortunately this function misses to do the mentioned check and does not return the `ZeroAmountOut` error on `state_changes.asset.delta_reserve == 0`. This leads to potential 100% losses for users on swaps.

**Recommended Mitigation Steps**

The issue can be mitigated by adding the same check that is used in the `sell()` function to the `sell_hub_asset()` function. 

```rust
ensure!(
	*state_changes.asset.delta_reserve > Balance::zero(),
	Error::<T>::ZeroAmountOut
);
```

## [L-03] Missing balance check in add_liquidity_shares()
[StableSwap Line 512](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/lib.rs#L512)

**Issue Description:

When a user deposits funds into the protocol, either for a swap or to add liquidity, the users balance must first be checked. In the `add_liquidty()` function this is checked as follows.

```rust
ensure!(
	T::Currency::free_balance(asset.asset_id, who) >= asset.amount,
	Error::<T>::InsufficientBalance
);
```

Unfortunately the `do_add_liquidity_shares()` function does not implement any check on the balance.

**Recommended Mitigation Steps

The issue can be mitigated by implementing the same check in the `do_add_liquidity_shares()` function.

## [L-04] Incorrect error returned for double adding of liquidity
[StableSwap Line 1017](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/lib.rs#L1017)

**Issue Description:

The StableSwap allows users to add liquidity for multiple assets at once. This is done by passing an array of assets and amount tuples to the `do_add_liquidity()` function. When an asset is more than once in that array, an error is correctly thrown. Unfortunately the error that is thrown is `IncorrectAssets` which is described as "Creating a pool with same assets or less than 2 assets is not allowed.". 

**Recommended Mitigation Steps:**

The issue can be mitigated by adding a custom error called `DuplicateEntry` for this error case.

## [L-05] Incorrect access for Authority
[Omnipool/lib.rs Line156](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs#L156)

**Issue Description:**

The Omnipools documentation states that the Authorities callable functions as

```rust
// Origin that can add token, refund refused asset and withdraw protocol liquidity.
```

Unfortunately by implementation the AuthorityOrigin is also able to call the function to remove tokens using `remove_token().

**Recommended Mitigation Steps:**

If it is intended that the AuthorityOrigin can remove tokens it must be added to the documentation. Otherwise the requirement needs to be removed/changed.

# Non-Critical

## [NC-01] Incorrect documentation of Error in `add_liquidity`
[omnipool/src/lib.rs Line 560](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs#L560)

**Issue Description:**

The documentation of the `add_liquidity()` function states:
```
//Asset weight cap must be respected, otherwise `AssetWeightExceeded` error is returned.
```
Unfortunately this is incorrect as the error is called `AssetWeightCapExceeded` not `AssetWeightExceeded`.

**Recommended Mitigation Steps**

The issue can be mitigated by correcting the documentation:

```Solidity
//Asset weight cap must be respected, otherwise `AssetWeightCapExceeded` error is returned.
```

## [NC-02] Typo in documentation of `calculate_remove_liquidity_state_changes`
[math/omnipool/math.rs Line 310](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/math.rs#L310)

**Issue Description:**

The documentation of the `calculate_remove_liquidity_state_changes()` function includes a typo on the word `liqudiity`. 

```rust
/// Calculate delta changes of remove liqudiity given current asset state and position from which liquidity should be removed.
```

**Recommended Mitigation Steps**

The issue can be mitigated by correcting the documentation:

```rust
/// Calculate delta changes of remove liquidity given current asset state and position from which liquidity should be removed.```
```

## [NC-03] Incorrect ratio mentioned in `buy_asset_for_hub_asset()`
[omnipool/src/lib.rs Line 1833](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs#L1833)

**Issue Description:

In the calculation if the `MaxOutRatio` is followed in the `buy_asset_for_hub_asset()` function, the comment states the wrong case for the error as failing if the `MaxInRatio` is zero. 

```rust
ensure!(
	amount
		<= asset_state
			.reserve
			.checked_div(T::MaxOutRatio::get())
			.ok_or(ArithmeticError::DivisionByZero)?, // Note: this can only fail if MaxInRatio is zero.
	Error::<T>::MaxOutRatioExceeded
);
```


**Recommended Mitigation Steps**

The Issue can be mitigated by correcting the comment.

```rust
ensure!(
	amount
		<= asset_state
			.reserve
			.checked_div(T::MaxOutRatio::get())
			.ok_or(ArithmeticError::DivisionByZero)?, // Note: this can only fail if MaxOutRatio is zero.
	Error::<T>::MaxOutRatioExceeded
);
```


## [NC-04] Incorrect ratio mentioned in `sell_hub_asset()`
[omnipool/src/lib.rs Line 1756](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs#L1756)

**Issue Description:

In the calculation if the `MaxOutRatio` is followed in the `sell_hub_asset()` function, the comment states the wrong case for the error as failing if the `MaxInRatio` is zero. 

```rust
ensure!(
	*state_changes.asset.delta_reserve
		<= asset_state
			.reserve
			.checked_div(T::MaxOutRatio::get())
			.ok_or(ArithmeticError::DivisionByZero)?, // Note: this can only fail if MaxInRatio is zero.
	Error::<T>::MaxOutRatioExceeded
);
```

**Recommended Mitigation Steps**

The Issue can be mitigated by correcting the comment.

```rust
ensure!(
	*state_changes.asset.delta_reserve
		<= asset_state
			.reserve
			.checked_div(T::MaxOutRatio::get())
			.ok_or(ArithmeticError::DivisionByZero)?, // Note: this can only fail if MaxOutRatio is zero.
	Error::<T>::MaxOutRatioExceeded
);
```

## [NC-05] Incorrect documentation of  `calculate_shares_for_amount()`
[math/src/stableswap/math.rs Line 172](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/math.rs#L172)

**Issue Description:**

The documentation of the `calculate_shares_for_amount()` incorrectly states:

```rust
/// Calculate amount of shares to be given to LP after LP provided liquidity of one asset with given amount.
```

This is incorrect as the function is used to calculate the shares that a user will have to burn when withdrawing liquidity.

**Recommended Mitigation Steps**

The issue can be fixed by adapting the comment as follows:

```rust
/// Calculate amount of shares to be burned from LP after LP withdraws liquidity.
```

