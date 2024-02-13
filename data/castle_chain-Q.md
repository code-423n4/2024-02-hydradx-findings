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