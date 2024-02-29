## Findings Summary

| Label | Description |
| - | - |
| [L-01](#) | Precision Loss in Calculating delta_hub_reserve in calculate_add_liquidity_state_changes()|
| [L-02](#) | calculate_add_one_asset() The returned fee was calculated incorrectly|
| [L-03](#) | calculate_out_given_in() Incorrect Rounding Direction for amount_in|
| [L-04](#) | Stableswap::add_liquidity() Lacks Slippage Protection  min_mint_amount|
| [L-05](#) | Insufficient Iterations for `MAX_Y_ITERATIONS` / `MAX_D_ITERATIONS`|
| [L-06](#) | Unreasonable Restriction in calculate_add_one_asset()|
| [L-07](#) | remove_liquidity() Missing Restriction: MinimumPoolLiquidity|
| [L-08](#) | omnipool.remove_liquidity() Missing get_price_weight()|

## [L-01] Precision Loss in Calculating `delta_hub_reserve` in `calculate_add_liquidity_state_changes()`

In the function `calculate_add_liquidity_state_changes()`, the current calculation for `delta_hub_reserve` is as follows:

```rust
let delta_hub_reserve = asset_state.price()?.checked_mul_int(amount)?;
```

After conversion, this is equivalent to:

```
delta_hub_reserve = (total_hub_reserve * 1e18 / total_reserve) * amount / 1e18
```

However, there is a precision loss issue in the expression `(total_hub_reserve * 1e18 / total_reserve)`.

recommend avoiding dividing `total_reserve` first and then multiplying by `amount`. Instead, use the following formula:

```
delta_hub_reserve = amount * total_hub_reserve / total_reserve
```

This approach is similar to the algorithm used for `delta_shares_hp`.

```diff
pub fn calculate_add_liquidity_state_changes(
	asset_state: &AssetReserveState<Balance>,
	amount: Balance,
	imbalance: I129<Balance>,
	total_hub_reserve: Balance,
) -> Option<LiquidityStateChange<Balance>> {
-	let delta_hub_reserve = asset_state.price()?.checked_mul_int(amount)?;

-	let (amount_hp, shares_hp, reserve_hp) = to_u256!(amount, asset_state.shares, asset_state.reserve);
+	let (amount_hp, shares_hp, reserve_hp,hub_reserve_hp) = to_u256!(amount, asset_state.shares, asset_state.reserve,asset_state.hub_reserve);

+       let delta_hub_reserve = hub_reserve_hp
+		.checked_mul(amount_hp)
+		.and_then(|v| v.checked_div(reserve_hp))?; 

	let delta_shares_hp = shares_hp
		.checked_mul(amount_hp)
		.and_then(|v| v.checked_div(reserve_hp))?; //@info round down is right

	let delta_imbalance = calculate_delta_imbalance(delta_hub_reserve, imbalance, total_hub_reserve)?;

	let delta_shares = to_balance!(delta_shares_hp).ok()?;


```


## [L-02] calculate_add_one_asset() The returned fee was calculated incorrectly
in `calculate_add_one_asset()` The calculated position is as follows：

| ____________________ | ____________________ | ____________________|
y1　　　　　　　　　　y　　　　　　reserves[asset_index]　　　　　asset_reserve

Correct should be：
amount_in = y1 - asset_reserve
dy_0  = y - reserves[asset_index]  　　　　　(The current implementation code is：dy_0 = y-asset_reserve）
fee = amount_in - dy_0

Suggested modification:
```diff
pub fn calculate_add_one_asset<const D: u8, const Y: u8>(
	reserves: &[AssetReserve],
	shares: Balance,
	asset_index: usize,
	share_asset_issuance: Balance,
	amplification: Balance,
	fee: Permill,
) -> Option<(Balance, Balance)> {
...


	let y1 = calculate_y_internal::<Y>(&reserves_reduced, Balance::try_from(d1).ok()?, amplification)?;
	let dy = y1.checked_sub(asset_reserve)?;
-      let dy_0 = y.checked_sub(asset_reserve)?;
+      let dy_0 = y.checked_sub(reserves[asset_index])?;
	let fee = dy.checked_sub(dy_0)?;
	let amount_in = normalize_value(dy, TARGET_PRECISION, asset_in_decimals, Rounding::Up);
	let fee = normalize_value(fee, TARGET_PRECISION, asset_in_decimals, Rounding::Down);
	Some((amount_in, fee))
}
```
## [L-03] calculate_out_given_in() Incorrect Rounding Direction for amount_in

In the function `calculate_out_given_in()`, when converting `amount_in` to `TARGET_PRECISION`, it currently uses `Rounding::Up`. However, the resulting value is used for calculating `amount_out`. Therefore, it is more advantageous for the protocol to use `Rounding::Down` for `amount_in`.

Recommendation:
```diff
pub fn calculate_out_given_in<const D: u8, const Y: u8>(
	initial_reserves: &[AssetReserve],
	idx_in: usize,
	idx_out: usize,
	amount_in: Balance,
	amplification: Balance,
) -> Option<Balance> {
	if idx_in >= initial_reserves.len() || idx_out >= initial_reserves.len() {
		return None;
	}
	let reserves = normalize_reserves(initial_reserves);
	let amount_in = normalize_value(
		amount_in,
		initial_reserves[idx_in].decimals,
		TARGET_PRECISION,
-		Rounding::Up,
+		Rounding::Down,
	);
	let new_reserve_out = calculate_y_given_in::<D, Y>(amount_in, idx_in, idx_out, &reserves, amplification)?;
	let amount_out = reserves[idx_out].checked_sub(new_reserve_out)?;
	let amount_out = normalize_value(
		amount_out,
		TARGET_PRECISION,
		initial_reserves[idx_out].decimals,
		Rounding::Down,
	);
	Some(amount_out.saturating_sub(1u128))
}
```

## [L-04] Stableswap::add_liquidity() Lacks Slippage Protection  min_mint_amount

The `add_liquidity()` function can result in liquidity amounts different from what users expect due to slippage. 
To mitigate this, it is advisable to introduce a minimum mint amount (`min_mint_amount`).


```diff
		pub fn add_liquidity(
			origin: OriginFor<T>,
			pool_id: T::AssetId,
			assets: Vec<AssetAmount<T::AssetId>>,
+                      min_mint_amount: Balance,
		) -> DispatchResult {

```


## [L-05] Insufficient Iterations for `MAX_Y_ITERATIONS` / `MAX_D_ITERATIONS`

For calculating Y and D, both currently use Newton's formula. 
recommend adjusting the iteration count to be similar to Curve's 255 iterations, which would yield more accurate results:

```diff
-pub const MAX_Y_ITERATIONS: u8 = 128;
-pub const MAX_D_ITERATIONS: u8 = 64;

+pub const MAX_Y_ITERATIONS: u8 = 255;
+pub const MAX_D_ITERATIONS: u8 = 255;
```

## [L-06] Unreasonable Restriction in `calculate_add_one_asset()`

The limitation that `shares < share_asset_issuance` is not logical. Users should be able to increase shares beyond `share_asset_issuance`. Typically, this restriction is used in `remove_liquidity()`. I recommend removing this constraint.

```diff
pub fn calculate_add_one_asset<const D: u8, const Y: u8>(
	reserves: &[AssetReserve],
	shares: Balance,
	asset_index: usize,
	share_asset_issuance: Balance,
	amplification: Balance,
	fee: Permill,
) -> Option<(Balance, Balance)> {
	if share_asset_issuance.is_zero() {
		return None;
	}
	if asset_index >= reserves.len() {
		return None;
	}

-	if shares > share_asset_issuance {
-		return None;
-	}
```


## [L-07] `remove_liquidity()` Missing Restriction: MinimumPoolLiquidity

The `remove_liquidity()` function lacks a restriction to prevent the amount reduced after removal from being less than the `MinimumPoolLiquidity`. 

recommend adding this restriction.

```diff
		pub fn remove_liquidity(
			origin: OriginFor<T>,
			position_id: T::PositionItemId,
			amount: Balance,
		) -> DispatchResult {

...
			} else {
				Self::deposit_event(Event::PositionUpdated {
					position_id,
					owner: who.clone(),
					asset: asset_id,
					amount: updated_position.amount,
					shares: updated_position.shares,
					price: updated_position
						.price_from_rational()
						.ok_or(ArithmeticError::DivisionByZero)?,
				});

				<Positions<T>>::insert(position_id, updated_position);

+                                 ensure!(
+                                      updated_position.amount >= T::MinimumPoolLiquidity::get(),
+                                      Error::<T>::MissingBalance
+                                  );
			}
```

## [L-08] `omnipool.remove_liquidity()` Missing `get_price_weight()`

suggest adding the `get_price_weight()` function to `omnipool.remove_liquidity()`.

```diff
		#[pallet::call_index(3)]
-		#[pallet::weight(<T as Config>::WeightInfo::remove_liquidity().saturating_add(T::OmnipoolHooks::on_liquidity_changed_weight()))]
+		#[pallet::weight(<T as Config>::WeightInfo::remove_liquidity().saturating_add(T::OmnipoolHooks::on_liquidity_changed_weight())).saturating_add(T::ExternalPriceOracle::get_price_weight())]
		#[transactional]
		pub fn remove_liquidity(
			origin: OriginFor<T>,
			position_id: T::PositionItemId,
			amount: Balance,
		) -> DispatchResult {
```








