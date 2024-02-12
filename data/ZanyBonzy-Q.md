***

# 1. Centralization risk from admin points of control
Links to affected code *

https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L337-L342
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L369-L374
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L403-L408
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L447-L454
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1340-L1345
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1384-L1390
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1425-L1426
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1452-L1459
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1552-L1553
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L341-L349
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L384-L386
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L408-L415
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L847

## Impact
The admins control a number of important functionalities in the protocol including setting trading limits, fees, which tokens to add or remove, amplifications and so on. These effect can have serious effect on the protocol. For instance, the admin by calling the `set_asset_tradable_state` function can maliciously freeze asset transactions causing instability and griefing users.

## Recommended Mitigation Steps
Consider implementing a multisig account and documenting the potential risks.
***
***


# 2. Ensure more sanity checks to prevent user griefing
Links to affected code *

https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1415
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L384
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L408
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L408
## Impact
Lack of sanity checks can cause users griefing through unreasonable entries from the admins. For instance, pool fee can be set so high that user swaps become unprofitable, leading to loss of funds for the users.

```
		pub fn update_pool_fee(origin: OriginFor<T>, pool_id: T::AssetId, fee: Permill) -> DispatchResult {
			T::AuthorityOrigin::ensure_origin(origin)?;

			Pools::<T>::try_mutate(pool_id, |maybe_pool| -> DispatchResult {
				let pool = maybe_pool.as_mut().ok_or(Error::<T>::PoolNotFound)?;

				pool.fee = fee;
				Self::deposit_event(Event::FeeUpdated { pool_id, fee });
				Ok(())
			})
		}
```
## Recommended Mitigation Steps
Consider introduction upper and if needed, lower limits to keep the values at a reasonable range.
Consider also including a check for 0 in the `update_amplification` function.
***
***


# 3. `set_asset_tradable_state` should also prevent freezing of `LRNA` token
Links to affected code *

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1337-L1343

## Impact
The `set_asset_tradable_state` allows for setting the tradeability status of the assets including LRNA, the protocol's hub token. For this reason, adding and removing LRNA liquidity is prohibited. However, the token can still be frozen which makes it possible to be removed from the protocol or just destabilize the protocol's swapping functionality. 
```
			if asset_id == T::HubAssetId::get() {
				// Atm omnipool does not allow adding/removing liquidity of hub asset.
				// Although BUY is not supported yet, we can allow the new state to be set to SELL/BUY.
				ensure!(
					!state.contains(Tradability::ADD_LIQUIDITY) && !state.contains(Tradability::REMOVE_LIQUIDITY),
					Error::<T>::InvalidHubAssetTradableState
				);

```
## Recommended Mitigation Steps
Consider checking also, that it cannot be frozen.
***
***

# 4. `sell` function asset state check can be refactored
Links to affected code *

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L957-L971

## Impact

When selling tokens, the `sell` function makes two important checks, if any of the tokens to be sold is a hub token and if the tokens to the sold have the `SELL` tradeability.

Doing this checks first if a token is the hub token, to be bought or sold, after which the the tokens state are finally checked.
```
			if asset_in == T::HubAssetId::get() { // if hub token is to be sold
				return Self::sell_hub_asset(origin, &who, asset_out, amount, min_buy_amount); // function checks for hub token tradeability
			}

			if asset_out == T::HubAssetId::get() { //if hub token is to be bought, Not allowed
				return Self::sell_asset_for_hub_asset(&who, asset_in, amount, min_buy_amount); // function checks for hub token tradeability
			}

			let asset_in_state = Self::load_asset_state(asset_in)?; //gets tokenIn state
			let asset_out_state = Self::load_asset_state(asset_out)?; //gets tokenOut state

			ensure!(
				Self::allow_assets(&asset_in_state, &asset_out_state),
				Error::<T>::NotAllowed
			);
```
The function can be refactored by checking for assets' state before specifically checking for the hub assets. It makes the checks earlier, can revert earlier as the HubAssetId is still a valid asset id and saves gas. 
## Recommended Mitigation Steps

```
		    let asset_in_state = Self::load_asset_state(asset_in)?; 
			let asset_out_state = Self::load_asset_state(asset_out)?; 

			ensure!(
				Self::allow_assets(&asset_in_state, &asset_out_state), //reverts here if hub token is not allowed to be bought/sold
				Error::<T>::NotAllowed
			);

			if asset_in == T::HubAssetId::get() {
				return Self::sell_hub_asset(origin, &who, asset_out, amount, min_buy_amount); 
			}

			if asset_out == T::HubAssetId::get() { //if hub token is to be bought, Not allowed
				return Self::sell_asset_for_hub_asset(&who, asset_in, amount, min_buy_amount); // function checks for hub token tradeability
			}
			
```
***
***

# 5. Consider checking for asset existence before setting limits
Links to affected code *

https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L369
https://github.com/code-423n4/2024-02-hydradx//blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L337
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L403
## Impact
The functions `set_add_liquidity_limit` `set_trade_volume_limit` and `set_remove_liquidity_limit` allow setting limits to asset trading. It however only checks that the asset is not the hub asset. 
## Recommended Mitigation Steps
Consider checking if the assetId is valid to avoid setting limits for non existent assets.

***
***

# 6. Lack of deadline parameter implementation in pools

Links to affected code *
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L577
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L716
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L934
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1140
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L475
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L512
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L551
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L638
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L717
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L787

## Impact

Interactions with AMMs are vulnerable to sandwich attacks, frontrunning and transaction execution at old slippage values. To fix this, AMMs provide users with an option to limit the execution of pending actions. The most common solution is to include a deadline timestamp as a parameter (for example see Uniswap V2 and Uniswap V3). 
The absence of a deadline means that the user's trades can be "saved for later", and held for a long time before execution, causing trade executions at bad or slightly unfavourable values, i.e the transaction can be withheld indefinitely at the advantage of the block producers.
It also facilitates integration with broader DeFi protocol as protcols that make calls to external AMMs tend to include deadline parameters.
## Recommended Mitigation Steps
Consider introducing a deadline parameter and a check for dated transactions. 

***
***

# 7. Vulnerable packages are being used 

-----
name = "shlex"
version = "1.2.0"

https://rustsec.org/advisories/RUSTSEC-2024-0006.html

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/Cargo.lock#L12937


-----

name = "snow"
version = "0.9.4"
https://rustsec.org/advisories/RUSTSEC-2024-0011.html

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/Cargo.lock#L13136

-----
name = "quinn-proto" ??
version = "0.9.6"

https://rustsec.org/advisories/RUSTSEC-2023-0063.html

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/Cargo.lock#L10694

-----

name = "ansi_term"
version = "0.12.1" 

https://rustsec.org/packages/ansi_term.html

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/Cargo.lock#L141

-----
name = "atty"
version = "0.2.14"

https://rustsec.org/packages/atty.html

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/Cargo.lock#L698

-----
name = "h2"
version = "0.3.22"

https://rustsec.org/advisories/RUSTSEC-2024-0003.html

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/Cargo.lock#L4212

## Recommended Mitigation Steps
Consider updating to the latest, safest versions.

***