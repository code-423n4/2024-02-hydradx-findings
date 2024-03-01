### Low-01 Incorrect comment - amount should be asset bought, not asset sold
**instances(1)**
In Omnipool -`buy()`, code comment for parameter `amount` is incorrect. `buy()` allows the user to enter the `amount` to buy, not `amount` to sell.
```rust
//HydraDX-node/pallets/omnipool/src/lib.rs
		/// Parameters:
		/// - `asset_in`: ID of asset sold to the pool
		/// - `asset_out`: ID of asset bought from the pool
|>		/// - `amount`: Amount of asset sold //@audit this should be Amount of asset bought
		/// - `max_sell_amount`: Maximum amount to be sold.
		///
		/// Emits `BuyExecuted` event when successful.
		///
		#[pallet::call_index(6)]
		#[pallet::weight(<T as Config>::WeightInfo::buy()
			.saturating_add(T::OmnipoolHooks::on_trade_weight())
			.saturating_add(T::OmnipoolHooks::on_liquidity_changed_weight())
		)]
		#[transactional]
		pub fn buy(
			origin: OriginFor<T>,
			asset_out: T::AssetId,
			asset_in: T::AssetId,
			amount: Balance,
			max_sell_amount: Balance,
		) -> DispatchResult {
...
```
(https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1129)

Recommendations:
Change the comment into Amount of the asset bought.

### Low-02 Incorrect comment - Should be Swap hub asset for asset  
**instances(1)**
In Omnipool -`buy_asset_for_hub_asset()`, the code comment says `/// Swap asset for Hub Asset`, which is incorrect. it should be `Swap hub asset for asset`.
```rust
//HydraDX-node/pallets/omnipool/src/lib.rs
        //@audit incorrect comment - should be Swap hub asset for asset.
|>	/// Swap asset for Hub Asset
	/// Special handling of buy trade where asset in is Hub Asset.
	fn buy_asset_for_hub_asset(
		origin: T::RuntimeOrigin,
		who: &T::AccountId,
		asset_out: T::AssetId,
		amount: Balance,
		limit: Balance,
	) -> DispatchResult {
...
```
(https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1804)

Recommendations:
Change the comment into Swap hub asset for asset.

### Low-03 `withdraw_protocol_liquidity()` doesn't check whether `safe_withdrawal` is true but only assume it is true
**instances(1)**
In Omnipool -`withdraw_protocol_liquidity()`, it assumes the function is only called when `safe_withdrawal` is true (no trading is allowed), however, it doesn't check `safe_withdrawal` value.
```rust
//HydraDX-node/pallets/omnipool/src/lib.rs
		pub fn withdraw_protocol_liquidity(
			origin: OriginFor<T>,
			asset_id: T::AssetId,
			amount: Balance,
			price: (Balance, Balance),
			dest: T::AccountId,
		) -> DispatchResult {
			T::AuthorityOrigin::ensure_origin(origin.clone())?;
...
			// Callback hook info
//@audit note: AssetInfo is input with hardcoded as `true` which indicates safe_withdrawal is true
|>			let info: AssetInfo<T::AssetId, Balance> =
				AssetInfo::new(asset_id, &asset_state, &new_asset_state, &state_changes.asset, true);
...
```
(https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1511)

Due to this function is access-controlled, this is low severity considering `AutorityOrigin` will not call it when safe_withdrawal is false.

Recommendations:
Add a check to ensure safe_withdrawal is true, instead of hardcoding `true` value.


### Low-04 `pool.is_valid()` will only check the minimum number of assets , not maximum number assets
**instances(1)**
In stableswap - `do_create_pool()`, pool validity is check in `is_valid()`. 

However, `is_valid()` only ensure there are minimum (2) assets in the pool but no maximum number cap is checked. Based on doc and lib.rs, we know the maximum number cap is (5). (`pub const MAX_ASSETS_IN_POOL: u32 = 5;`)
```rust
//HydraDX-node/pallets/stableswap/src/lib.rs
fn do_create_pool(
...
//@audit pool.is_valid() only check wether there're at least two assets in the pool, but it didn't check whether the max is over 5. 
|>		ensure!(pool.is_valid(), Error::<T>::IncorrectAssets);
...
pub(crate) fn is_valid(&self) -> bool {
|> self.assets.len() >= 2 && has_unique_elements(&mut self.assets.iter())
}
```
(https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/types.rs#L54)

Recommendations:
Check both minimum number and maximum number.

### Low-05 Single constant number of MinTradingLimit/ MinPoolLiquidity might not fit assets with various decimal precision.
**instances(1)**
In stableswap pool, `MinTradingLimit`/ `MinPoolLiquidity` are used to check trading and liquidity management. However, both `MinTradingLimit` and `MinPoolLiquidity` are hardcoded `Balance` which might not fit all possible assets decimal precisions stableswap intends to support. It might over constraint or under constraint depending on exact token decimal place, which allow user to deposit or trade dust amount.
```rust
		/// Minimum pool liquidity
		#[pallet::constant]
		type MinPoolLiquidity: Get<Balance>;

		/// Minimum trading amount
		#[pallet::constant]
		type MinTradingLimit: Get<Balance>;
```
(https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L152-L157)

Recommendations:
Consider allowing this to be configurable perhaps by pool or by asset. Or trading limit can be enforced as a ratio as well.

### Low-06 Stableswap pool will always have a portion of funds locked inside due to vulnerable fee implementation.
**instances(1)**
The problem is there is going to be a small amount of assets left in the stableswap pool, because the fee implementation in `remove_liquidity_one_asset()` and `withdraw_asset_amount()`. 
```rust
//HydraDX-node/pallets/stableswap/src/lib.rs
		pub fn remove_liquidity_one_asset(
			origin: OriginFor<T>,
			pool_id: T::AssetId,
			asset_id: T::AssetId,
			share_amount: Balance,
			min_amount_out: Balance,
		) -> DispatchResult {
...
			//Calculate how much asset user will receive. Note that the fee is already subtracted from the amount.
                        //@audit fee amount will be deducted no matter the current reserve state
|>			let (amount, fee) = hydra_dx_math::stableswap::calculate_withdraw_one_asset::<D_ITERATIONS, Y_ITERATIONS>(
				&initial_reserves,
				share_amount,
				asset_idx,
				share_issuance,
				amplification,
				pool.fee,
			)
...
```
(https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L592)

Recommendations:
Allowing the fee feature to be disabled by admin.





