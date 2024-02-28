# Stapleswap
## 1) liquidity providers can not provide amount of assets that will result in shares more than the total issuance by calling `add_liquidity_shares` function , which is considered loss of value for the protocol .
affected code : 
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/math/src/stableswap/math.rs#L317-L337

the function `add_liquidity_shares` uses the function `calculate_add_one_asset` [here](https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/math/src/stableswap/math.rs#L317) which has a constrain that ensures that the shares specified by the user to be minted are not greater than the total issuance , there is nothing should prevent the liquidity providers from providing such amount of asset , and this amount of shares to be minted is not prevented by the function `add_liquidity` so such a constrain should be removed .
```rust 
        if shares > share_asset_issuance {
                return None;
        }
```
### coded poc 
consider add this test to the test file `add_liquidity.rs`
```rust 
#[test]
fn add_liquidity_should_work_correctly_when_fee_is_applied_test() {
	let asset_a: AssetId = 1;
	let asset_b: AssetId = 2;
	let asset_c: AssetId = 3;
	ExtBuilder::default()
		.with_endowed_accounts(vec![
			(BOB, asset_a, 200_000_000_000_000_000_000),
			(ALICE, asset_a, 52425995641788588073263117),
			(ALICE, asset_b, 52033213790329),
			(ALICE, asset_c, 119135337044269),
		])
		.with_registered_asset("one".as_bytes().to_vec(), asset_a, 18)
		.with_registered_asset("two".as_bytes().to_vec(), asset_b, 6)
		.with_registered_asset("three".as_bytes().to_vec(), asset_c, 6)
		.with_pool(
			ALICE,
			PoolInfo::<AssetId, u64> {
				assets: vec![asset_a, asset_b, asset_c].try_into().unwrap(),
				initial_amplification: NonZeroU16::new(2000).unwrap(),
				final_amplification: NonZeroU16::new(2000).unwrap(),
				initial_block: 0,
				final_block: 0,
				fee: Permill::from_float(0.0001),
			},
			InitialLiquidity {
				account: ALICE,
				assets: vec![
					AssetAmount::new(asset_a, 5_641_788_588_073_263_117),
					AssetAmount::new(asset_b, 52033213790329),
					AssetAmount::new(asset_c, 119135337044269),
				],
			},
		)
		.build()
		.execute_with(|| {
			let pool_id = get_pool_id_at(0);
			// let amount = 2_000_000_000_000_000_000;
			let total_shares = Tokens::total_issuance(pool_id); 
			assert_ok!(Stableswap::add_liquidity_shares(
				RuntimeOrigin::signed(BOB),
				pool_id,
				total_shares + 1 , 
				asset_a , 
				200_000_000_000_000_000_000
			));
			let received = Tokens::free_balance(pool_id, &BOB);
			println!("shares received after providing 2_000_000_000_000_000_000 asset and call add_liquidity function : {:?}", received);
			let bob_balance = Tokens::free_balance(asset_a, &BOB);
			let used = 200_000_000_000_000_000_000 - bob_balance ; 
			println!("used : {:?}" , used);
		});
// 108_887_514_683_615_710_558 assets should be taken from the user but the function call will revert due to that the shares requested are more than the total issuance 
}

```

this test should revert due to the shares requested is greater than the total issuance by 1 
to make this test work you can simply remove this one here 
```diff 
+       total_shares ,
-				total_shares + 1 , 
```

### Recommendation 
consider removing this constrain 
```diff
-        if shares > share_asset_issuance {
-                return None;
-        }
```

## 2) huge loss of funds for the users and the protocol , if the pool is created with share_asset that does have a different decimals from 18 decimals 
affected code : 
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L341-L369 . 

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L950-L992

The implementation of stableswap assumes that all the shares assets have 18 decimals only but there is no check for this in the function `create_pool` which can allow set the `share_asset` to has any number of decimals which will affect the whole pool since the normalization function scale all the reserves to 18 decimals in order to calculate D , and Y parameters . 

**the impact is :** 
this will lead to huge loss of funds for the user and the protocol because of the wrong calculation of Y , and D parameters 

### Recommendations 
consider adding a check to ensure that the `share_asset` decimals are equal to 18 decimals .
add this check to the `do_create_pool` function : 
```diff 
+                ensure!(
+                        T::AssetInspection::decimals(share_asset)== 18,
+                        Error::<T>::Invalid_decimals
+                );
```

# Omnipool
## 1) If the initial token price is set too far from the oracle price within the `add_token()` function, it will lead to the freezing of liquidity provision .
affected code : https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L447-L549 . 

This outcome is a consequence of the price constraint embedded in the `add_liquidity` [here](https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L597-L603) function, which ensures that the disparity between the spot and oracle prices remains within acceptable bounds. Consequently, deviating significantly from the oracle price when setting the initial token price can trigger this constraint, resulting in the freezing of liquidity provision.


#### Recommendations 
 call the function `ensure_price()` inside the `add_token` function to ensure that initial price is within the range of oracle price.
```diff 
                pub fn add_token(
                        origin: OriginFor<T>,
                        asset: T::AssetId,
                        initial_price: Price,
                        weight_cap: Permill,
                        position_owner: T::AccountId,
                ) -> DispatchResult {
                        T::AuthorityOrigin::ensure_origin(origin.clone())?;
+                          T::PriceBarrier::ensure_price(
+                               &AccountId,
+                                T::HubAssetId::get(),
+                                asset,
+                                initial_price,
+                        )
+                        .map_err(|_| Error::<T>::PriceDifferenceTooHigh)?;

                        ensure!(!Assets::<T>::contains_key(asset), Error::<T>::AssetAlreadyAdded);


                        ensure!(T::AssetRegistry::exists(asset), Error::<T>::AssetNotRegistered);


                        ensure!(initial_price > FixedU128::zero(), Error::<T>::InvalidInitialAssetPrice);


                        // ensure collection is created, we can simply ignore the error if it was already created.
```
## 2) the `add_token()` function lacks a check to prevent exceeding the maximum weight cap which  will disable liquidity provision . 
affected code :  https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L447-L549 . 

 Each token is assigned a maximum weight cap, specifying the maximum amount of hub asset that can be minted in its corresponding pool. If the quantity of hub asset paired (minted) with the added token surpasses its permitted weight capacity, liquidity provision will be frozen. This is because the `add_liquidity`
[here](https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L636-L639) function incorporates a capacity constraint, and exceeding this limit triggers the freezing of liquidity provision . 
#### Recommendation 
check the cap of the asset in the function `add_token()`
```diff
                pub fn add_token(
                        origin: OriginFor<T>,
                        asset: T::AssetId,
                        initial_price: Price,
                        weight_cap: Permill,
                        position_owner: T::AccountId,
                ) -> DispatchResult {

                        ensure!(
                                hub_reserve_ratio <= new_asset_state.weight_cap(),
                                Error::<T>::AssetWeightCapExceeded
                        );
```  
## 3) Customizing the minimum trading limit on a per-asset basis is essential to facilitate liquidity provision for assets characterized by lower decimal precision and higher market prices. 

affected code : 
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L577-L584 . 

Tokens like [Gemini USD](https://etherscan.io/token/0x056Fd409E1d7A124BD7017459dFEa2F387b6d5Cd?a=0x5f65f7b609678448494De4C87521CdF6cEf1e932) possess just 2 decimals, while others like USDC has 6 decimals. Within the `add_liquidity` function, it is crucial to align the `MinimumPoolLiquidity` with the amount of `hub_asset`. Failure to consider this alignment, particularly when the specified amount surpasses the `maxWeightCap` established for the asset, can result in the disabling of liquidity provision for that specific token.


To illustrate, let's take a low-decimal asset with 2 decimals and set its `MinimumPoolLiquidity` at 10,000, equating to 100 units of that asset. If this quantity of tokens corresponds to an amount of hub assets exceeding the weight cap assigned to the asset , if the max weight cap for this asset is 20% of a total hub reserve is 100,000 ,
then the max amount of hub allowed to be added equal to 25,000 ,and if the corresponding hub asset for the 100 units of asset equal to 30,000,
 liquidity provision for this asset becomes disabled. This underscores the necessity for a nuanced approach in determining the minimum trading limit, ensuring the smooth provision of liquidity for assets with lower decimal precision and elevated market values, whether with 2 or 6 decimals. 

#### Recommendation 
create a map type to store the `minimumPoolLiquidty` of each Pool , or can store this minimum limit in the asset registry . 
this will allow specify a suitable minimum limit to each asset in the omnipool . 

## 4) the getter function can be used instead of repeating code to get the hub_asset_liquidity , to increase code simplicity 
code affected : https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1946-L1948

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L509
   
`get_hub_asset_balance_of_protocol_account()` function can be used instead of getting the balance of the protocol in each function like this , 
```diff 
-                    let current_hub_asset_liquidity = T::Currency::free_balance(T::HubAssetId::get(), &Self::protocol_account());
+                          let hub_asset_liquidity = Self::get_hub_asset_balance_of_protocol_account();
```

# Ema_oracle

 ## 1) prevnet calling the funtcions `on_trade` and `on_liquidity_changed` with asset_in = asset_out , to prevent storing invalid prices . 
code affected : 
1) https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/ema-oracle/src/lib.rs#L386-L417
2) https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/ema-oracle/src/lib.rs#L428-L456

the function `on_trade` and `on_liquidity_changed` do not check if the `asset_in` equal `asset_out` or not , so this will allow storing invalid prices in the oracle . 

since the function `buy` on stableswap pallet does not prevent that `asset_in` to be equal to `asset_out` as shown here : 
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L787-L807

```rust 
                pub fn buy(
                        origin: OriginFor<T>,
                        pool_id: T::AssetId,
                        asset_out: T::AssetId,
                        asset_in: T::AssetId,
                        amount_out: Balance,
                        max_sell_amount: Balance,
                ) -> DispatchResult {
                        let who = ensure_signed(origin)?;


                        ensure!(
                                Self::is_asset_allowed(pool_id, asset_in, Tradability::SELL)
                                        && Self::is_asset_allowed(pool_id, asset_out, Tradability::BUY),
                                Error::<T>::NotAllowed
                        );


                        ensure!(
                                amount_out >= T::MinTradingLimit::get(),
                                Error::<T>::InsufficientTradingAmount
                        );


```
### Recommendation 
the function should prevent set `asset_in` to be equal to `asset_out`


## 2) prevent store prices from `on_trade` function with `amount_a` and `amount_b` equal to zero . 
code affected:
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/ema-oracle/src/lib.rs#L386-L404

`on_trade` function should be called by the `adapter` in order to storing the price after a trade has been executed , in order to consider the price comes from a trade is valid the amount traded should be greater than zero . 

### Recommendation 
the function should check if the `amount_a` and `amount_b` are greater than zero , if not the function should return error .
consider add this code to `on_trade` function . 
```rust

                // We assume that zero liquidity values are not valid and can be ignored.
                if amount_a.is_zero() && amount_b.is_zero() {
                        log::warn!(
                                target: LOG_TARGET,
                                "trade amounts should not be zero. Source: {source:?}, amounts: ({amount_a},{amount_b})"
                        );
                        return Err((Self::on_trade_weight(), Error::<T>::OnTradeValueZero.into()));
                }


```

# circuit breaker 

## should check that the amounts to be added or subtracted are greater than zero before executing the rest of the function and update the values . 
code affected : 
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L485-L516

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L518-L545


the functions such as `ensure_and_update_trade_volume_limit` , and `ensure_and_update_add_liquidity_limit` , should make sure that the `amount_out` and `amount_in` , and `liquidity_added` are greater than zero before keeping execution of the code.


```rust 
        fn ensure_and_update_trade_volume_limit(
                asset_in: T::AssetId,
                amount_in: T::Balance,
                asset_out: T::AssetId,
                amount_out: T::Balance,
        ) -> DispatchResult {
                // liquidity in
                // ignore Omnipool's hub asset
                if asset_in != T::OmnipoolHubAsset::get() {
                        let mut allowed_liquidity_range = Pallet::<T>::allowed_trade_volume_limit_per_asset(asset_in)
                                .ok_or(Error::<T>::LiquidityLimitNotStoredForAsset)?;


                        allowed_liquidity_range.update_amounts(amount_in, Zero::zero())?;
                        allowed_liquidity_range.check_limits()?;


                        <AllowedTradeVolumeLimitPerAsset<T>>::insert(asset_in, allowed_liquidity_range);
                }


                // liquidity out
                // ignore Omnipool's hub asset
                if asset_out != T::OmnipoolHubAsset::get() {
                        let mut allowed_liquidity_range = Pallet::<T>::allowed_trade_volume_limit_per_asset(asset_out)
                                .ok_or(Error::<T>::LiquidityLimitNotStoredForAsset)?;


                        allowed_liquidity_range.update_amounts(Zero::zero(), amount_out)?;
                        allowed_liquidity_range.check_limits()?;


                        <AllowedTradeVolumeLimitPerAsset<T>>::insert(asset_out, allowed_liquidity_range);
                }


                Ok(())
        }
```
## Recommendation 
check that those amounts are not equal to zero .
```rust 
ensure!(
			!amount_out.is_zero() && !amount_in.is_zero(),
			Error::<T>::invalidValues
		);
```

## Potential Liquidity addition Freeze in Omnipool Due to Limited Add Functionality by the circuit breaker . 

Omnipool enforces a minimum limit of 1,000,000 for both adding and removing liquidity, regardless of the specific asset.

due to that there is no Ranges for the limits in the circuit breaker , and considering that the default_max_add_liquidity_limit is equal to 5% , the liquidity addition can be freezed by depositing the initial deposit equals to the `MinimumPoolLiquidity` , so if the initial liquidity is 1_000_000 the max amount to be added in a single block allowed by the circuit breaker is 5000 which is much less than the minimum limit of liquidity set by the omnipool , so the liquidity addition will be freezed . 

### Recommendations
set the max limit of adding liquidity of the asset in the circuit breaker to be equal to the minimum liquidity limit of the omnipool if the calculated max limit is below it .

```diff
        fn calculate_and_store_liquidity_limits(asset_id: T::AssetId, initial_liquidity: T::Balance) -> DispatchResult {
                // we don't track liquidity limits for the Omnipool Hub asset
                if asset_id == T::OmnipoolHubAsset::get() {
                        return Ok(());
                }


                // add liquidity
                if let Some(limit) = Pallet::<T>::add_liquidity_limit_per_asset(asset_id) {
                        if !<AllowedAddLiquidityAmountPerAsset<T>>::contains_key(asset_id) {
                                let max_limit = Self::calculate_limit(initial_liquidity, limit)?;
+                 if (max_limit < 1_000_000) {
+                  max_limit = 1_000_000 ; 
+                 }
                                <AllowedAddLiquidityAmountPerAsset<T>>::insert(
                                        asset_id,
                                        LiquidityLimit::<T> {
                                                limit: max_limit,
                                                liquidity: Zero::zero(),
                                        },
                                );
                        }
                }


```
