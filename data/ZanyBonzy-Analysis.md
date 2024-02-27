#  **Advanced Analysis Report for <img width= auto src="https://hydradx.io/assets/logo-v2.svg" alt="logo">**

[Audit approach](#1-audit-approach)

[Brief Overview](#2-brief-overview)

[Scope and Architecture Overview](#3-scope-and-architecture-overview)

[Codebase Overview](#4-codebase-overview)

[Centralization Risks](#5-centralization-risks)

[Systemic Risks](#6-systemic-risks)

[Recommendations](#7-recommendations)

[Conclusions](#8-conclusions)

[Resources](#9-resources)


## **1. Audit approach**

- **Ecosystem analysis**: A study of the PolkaDot's ecosystem and its parachains. A brief overview of the rust programming language syntax to provide a base for understanding of the protocol's implementation.
 
- **Documentation Dive**: A comprehensive analysis of provided documentation, threat modelling and invariants was conducted to understand protocol functionality, key points were noted, ambiguities were discussed with the dev, and possible risk areas were mapped.

- **Code Inspection**: Manual review of each contract within defined sections was conducted, testing function behavior against expectations, and working out potential attack vectors. Vulnerabilities related to dependencies and  inheritances, were assessed. 

- **Report Compilation**: Identified issues were generated into a comprehensive audit report.
***

## **2. Brief Overview**

- HydraDX Omnipool, an innovative polkadot AMM that combines all assets in a single pool to solve the problem fragmented liquidity in existing DeFi protocols makes trading inefficient.
- It does this by providing unparalleled efficiency through less hopping between pools, lower slippage, and improved capital gains.
- It eases participation by providing liquidity for any single asset, even in early stages, hence incentivizing growth to bootstrap liquidity.
***
## **3. Scope and Architecture Overview**

For the audit scope, the contracts are sectioned into three different sections.

### **3.1. Omnipool [pallet](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/omnipool) & [math](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/omnipool)**

- Omnipool is a type of Automated Market Maker (AMM) where all assets are combined into a single pool. This differs from traditional AMMs which require liquidity providers (LPs) to pair tokens. In Omnipool, LPs deposit any asset and receive pool shares. Traders exchange any token through the pool using the "hub" token LRNA, leading to fragmented liquidity and efficient trades.

- Key features include use of a single pool where all assets are pooled together, simplifying trades and increasing capital efficiency, provision of a token (LRNA) to facilitate trades and absorbs imbalances to stabilize pool value and generation of LP incentives by granting pool shares to liquidity providers representing their contribution and gain rewards.

**[Pallet Lib](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs)**

- **Pool creation and closure** - A pool can be created by adding a token into the omnipool. Before this can be accomplished, the token must have been registered in the asset registry and initial liquidity must first be transferred to the pool account upon which the `AuthorityOrigin` admin calls the `add_token` function. A collection of NFTs are then created to be minted for future liquidity providers.

- To close the pool, the token is first frozen and the remaining shares must belong to the protocol. The `AuthorityOrigin` then calls the `remove_token` function which transfers the existing protocol shares to the beneficiary and the token is removed.

- **Increasing and decreasing pool liquidity** - Users call the `add_liquidity` and `remove_liquidity` to increase/decrease their positions in the pools. Adding liquidity mints a position NFT to the user. Important to note that positions cannot be increased per se. A new nft position is minted to the user everytime liquidity is provided. Removing liquidity allows full/partial removals upon which the user's position is decreased, or when 0, the position NFT, burned. The assets to be traded must have the `ADD_LIQUIDITY` and `REMOVE_LIQUIDITY` states enabled.

- **Asset trades/swaps** - The `sell` and `buy` functions allow users to trade an asset for another. The assets to be traded must have the `SELL` and `BUY` states enabled. 

- On a side note, the protocol's hub token LRNA, only has the `SELL` state enabled. Therefore, it cannot be bought, liquidity cannot be added or removed.

- **Position sacrifice and protocol liquidity withdrawal** - Users feeling charitable can sacrifice their positions by calling the `sacrifice_position` function. Doing this is akin to removing liquidity, but transferring it to the protocol instead of themselves. Upon position sacrifice, the NFT positon is burned and the `AuthorityOrigin` can call the `withdraw_protocol_liquidity` function to withdraw the protocol's own shares. 

To ensure full omnipool functionality, the `lib` depends on the pallet [types](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/types.rs) and [traits](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/traits.rs) contracts to provide the needed data types, perform basic calculations, ensure various prices and holds various omnipool hooks.

**[Math](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/math.rs)**

- **State changes calculations** - When various transactions are performed in the pool, hooks are made from the Omnipool lib contract to the Math contract to rebalance the pool by calculating the delta changes of adding/removing liquidity to the pool. Upon selling assets, the `calculate_sell_state_changes` or the `calculate_sell_hub_state_changes` hooks are called, depending on if the assets being sold is the hub token or other assets. The `calculate_buy_for_hub_asset_state_changes` and `calculate_buy_state_changes` functions handle rebalancing when the hub token and other assets are called. The `calculate_add_liquidity_state_changes` and `calculate_remove_liquidity_state_changes` functions are called when assets are added or removed from the pool.

- **Fee calculations** - These functions handle fee calculation upon adding or removing assets from the pools. When assets are being added or removed, the functions call the state function hooks which calculates the fees to be charged to the caller. The `calculate_fee_amount_for_buy` and `calculate_withdrawal_fee` functions are responsible for this.

- Other important calculations made here are the `calculate_tvl`, `calculate_cap_difference` and `calculate_spot_sprice` functions which check the pool tvls, assets cap difference and asset spot prices.

As a support system, for the Math contract, the [types](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/types.rs) contract is used to perform basic arithmetic operations, hold various data types and performing various delta updates.

```
Omnipool Pallet Lib
|
|--- Assets<T: Config>
|       |
|       |--- StorageMap<_, Blake2_128Concat, T::AssetId, AssetState<Balance>>
|
|--- HubAssetImbalance<T: Config>
|       |
|       |--- StorageValue<_, SimpleImbalance<Balance>, ValueQuery>
|
|--- HubAssetTradability<T: Config>
|       |
|       |--- StorageValue<_, Tradability, ValueQuery, DefaultHubAssetTradability>
|
|--- Positions<T: Config>
|       |
|       |--- StorageMap<_, Blake2_128Concat, T::PositionItemId, Position<Balance, T::AssetId>>
|
|--- NextPositionId<T: Config>
|       |
|       |--- StorageValue<_, T::PositionItemId, ValueQuery>
|
|--- Events<T: Config>
|       |
|       |--- Enum: TokenAdded, TokenRemoved, LiquidityAdded, LiquidityRemoved, ProtocolLiquidityRemoved, SellExecuted, BuyExecuted, PositionCreated, PositionDestroyed, PositionUpdated, TradableStateUpdated, AssetRefunded, AssetWeightCapUpdated
|
|--- Errors<T>
|       |
|       |--- Enum: InsufficientBalance, AssetAlreadyAdded, AssetNotFound, MissingBalance, InvalidInitialAssetPrice, BuyLimitNotReached, SellLimitExceeded, PositionNotFound, InsufficientShares, NotAllowed, Forbidden, AssetWeightCapExceeded, AssetNotRegistered, InsufficientLiquidity, InsufficientTradingAmount, SameAssetTradeNotAllowed, HubAssetUpdateError, PositiveImbalance, InvalidSharesAmount, InvalidHubAssetTradableState, AssetRefundNotAllowed, MaxOutRatioExceeded, MaxInRatioExceeded, PriceDifferenceTooHigh, InvalidOraclePrice, InvalidWithdrawalFee, FeeOverdraft, SharesRemaining, AssetNotFrozen, ZeroAmountOut
|
|--- Calls<T: Config>
|       |
|       |--- add_token
|       |--- add_liquidity
|       |--- remove_liquidity
|       |--- sacrifice_position
|       |--- sell
|       |--- buy
|       |--- set_asset_tradable_state
|       |--- refund_refused_asset
|       |--- set_asset_weight_cap
|       |--- withdraw_protocol_liquidity
|       |--- remove_token
|
|--- Implementation<T: Config>
     |
     |--- protocol_account
     |--- load_asset_state
     |--- set_asset_state
     |--- create_and_mint_position_instance
     |--- update_hdx_subpool_hub_asset
     |--- update_hub_asset_liquidity
     |--- update_imbalance
     |--- allow_assets
     |--- sell_hub_asset
     |--- buy_asset_for_hub_asset
     |--- buy_hub_asset
     |--- sell_asset_for_hub_asset
     |--- get_hub_asset_balance_of_protocol_account
     |--- remove_asset
     |--- set_position
     |--- add_asset
     |--- update_asset_state
     |--- load_position
     |--- is_hub_asset_allowed
     |--- exists
     |--- process_trade_fee
     |--- process_hub_amount

```
***
### **3.2. StableSwap [pallet](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/stableswap) & [math](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/stableswap)**

- The StableSwap automates trades between stablecoins (like USDC and DAI) similar to Curve Finance, slippage for users. It uses a special algorithm designed for stablecoins, similar to the one used by Curve Finance. Liquidity providers can add and remove assets from pools, represented by share tokens. The system is secure as it limits pool creation and asset types by allowing up to 5 different stablecoins, and only authorized entities can create them. Liquidity providers initially need to add all the pool's stablecoins and get share tokens in return. Later, they can withdraw specific stablecoins from the pool.

**[Pallet Lib](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/lib.rs)**

- **Pool creation** - The AuthorityOrigin has the power to create a new stable asset pool with a given list of assets not more than 5. He does this by calling the `create_pool` function. The assets must have been initially registered in the asset registry, so as to ensure proper tracking. The admin doesn't setup the pool's initial liquid through this function, liquidity has to be added seperately.

- **Liquidity addition and removal** - Users call the `add_liquidity` function to add liquidity to pools of their choice. There is a minimum amount of tokens that can be deposited to prevent the first depositor attack. Users can also make calculations on the desired amount of shares they would like, by calling the `add_liquidity_shares`, which holds the `max_asset_amout` parameter to function as slippage. To remove liquidity from the pool, there are 2 main methods. The `remove_liquidity_one_asset` which allows users to specify the amount of shares they're willing to burn, and the `withdraw_asset_amount` function which allows users to specify the amount of assets they're willing to receive. The assets being added or removed must have had their tradeability status set to the addliquidity and removeliquidity statuses.

- **Asset swaps** - Assets with the sell and buy tradeability status turned on can perform swaps by calling the `sell` and `buy` functions. By calling the `sell`  or `buy` functions, the `asset_in` is sold to the pool to receive `asset out`, and to protect from slippage, there are minimum buy amout and maximum sell amount parameters for the user to use.

To handle the various hooks in the StableSwap pool, the lib contract uses the [types](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/types.rs) contract which holds various function types, hook functions, and so on, which can be accessed upon pool activity.

**[Math](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/math.rs)**

- **Amount conversions calculations** - To calculate the amount to be received from the pool when given a certain amount to be sent and vice versa, the `calculate_out_given_in` and `calculate_in_given_out` functions can be called. To consider fees in the calculations instead, the `calculate_out_given_in_with_fee` and `calculate_in_given_out_with_fee` functions are called instead which convert the needed amounts and call the `calculate_fee_amount` hook. 

- **Shares calculations** - When liquidity is provided to a pool, the `calculate_shares` and `calculate_shares_for_amount` are used as hooks to provide the total amount of shares a user is entitled to upon adding liquidity into the pools. 

- **Price calculations** - The `calculate_share_prices`, `calculate_share_price` and `calculate_spot_price` functions are important hooks called to calculate current prices based on the pool balances.

The [types](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/types.rs) contract is used as a dependent library which the math contract uses to hold the asset reserve data types, and perform various checks.

```
     +----------------------------------------------------------------------------------------------------------------+
     |                                  StableSwap Pallet lib                                                         |
     +----------------------------------------------------------------------------------------------------------------+
     | Storage                           |                                                      		      |
     | - Pools                           | StorageMap<Blake2_128Concat, T::AssetId, PoolInfo>    		      |
     | - AssetTradability                | StorageDoubleMap<Blake2_128Concat, T::AssetId, Tradability, ValueQuery>    |
     +-----------------------------------+----------------------------------------------------------------------------+
     | Events                            |                                                       		      |
     | - PoolCreated                     |                                   		                              |
     | - FeeUpdated                      | 		      		      		                              |
     | - LiquidityAdded                  |                                                      		      |
     | - LiquidityRemoved                |                                                       		      |
     | - SellExecuted                    |                                                       		      |
     | - BuyExecuted                     |                                                       		      |
     | - TradableStateUpdated            |                                                   		              |
     | - AmplificationChanging           |                                                 		              |
     +-----------------------------------+----------------------------------------------------------------------------+
     | Errors                            |                                                   	                      |
     | - IncorrectAssets                 |                                                  	                      |
     | - MaxAssetsExceeded               |                                                      	              |
     | - PoolNotFound                    |                                                      	              |
     | - PoolExists                      |                                                       	              |
     | - AssetNotInPool                  |                                                       	              |
     | - ShareAssetNotRegistered         |                                               	                      |
     | - ShareAssetInPoolAssets          |                                                	                      |
     | - AssetNotRegistered              |                                                   	                      |
     | - InvalidAssetAmount              |                                            	                              |
     | - InsufficientBalance             |                                                   	                      |
     | - InsufficientShares              |                                                   	                      |
     | - InsufficientLiquidity           |                                                  	              	      |
     | - InsufficientLiquidityRemaining  |                                          	              	              |
     | - InsufficientTradingAmount       |                                          	              	              |
     | - BuyLimitNotReached              |                                            	                              |
     | - SellLimitExceeded               |                                              	                      |
     | - InvalidInitialLiquidity         |                                                	         	      |
     | - InvalidAmplification            |                                                	                      |
     | - InsufficientShareBalance        |                                              	                      |
     | - NotAllowed                      |                                                      	              |
     | - PastBlock                       |                                                      	              |
     | - SameAmplification               |                                                    	                      |
     | - SlippageLimit                   |                                                       	              |
     | - UnknownDecimals 		 |                                                       	              |
     +-----------------------------------+----------------------------------------------------------------------------+
     | Calls             		 |                                                      	              |
     | - create_pool     		 | origin, share_asset, assets, amplification, fee      	              |
     | - update_pool_fee  		 | origin, pool_id, fee                                 	              |
     | - update_amplification 	         | origin, pool_id, final_amplification, start_block, end_block               |
     | - add_liquidity    		 | origin, pool_id, assets                              	              |
     | - add_liquidity_shares 	         | origin, pool_id, shares, asset_id, max_asset_amount 	                      |
     | - remove_liquidity_one_asset      | origin, pool_id, asset_id, share_amount, min_amount_out 	              |
     | - withdraw_asset_amount 	         | origin, pool_id, asset_id, amount, max_share_amount 	                      |
     | - sell            		 | origin, pool_id, asset_in, asset_out, amount_in, min_buy_amount            |
     | - buy             		 | origin, pool_id, asset_out, asset_in, amount_out, max_sell_amount          |
     | - set_asset_tradable_state        | origin, pool_id, asset_id, state               	                      |
     +-----------------------------------+----------------------------------------------------------------------------+
     | Hooks          		         | Implemented for BlockNumberFor<T>            	                      |
     +-----------------------------------+----------------------------------------------------------------------------+
     | Helper Functions  		 |                                                       	              |
     | - calculate_out_amount 	         | pool_id, asset_in, asset_out, amount_in          	                      |
     | - calculate_in_amount 	         | pool_id, asset_in, asset_out, amount_out       	                      |
     | - do_create_pool   		 | share_asset, assets, amplification, fee               	              |
     | - do_add_liquidity 		 | who, pool_id, assets                                  	              |
     | - do_add_liquidity_shares         | who, pool_id, shares, asset_id, max_asset_amount 	                      |
     | - is_asset_allowed 		 | pool_id, asset_id, operation                          	              |
     | - pool_account     		 | pool_id                                               	              |
     | - get_amplification 		 | pool                                                 	              |
     | - retrieve_decimals 		 | asset_id                                             	              |
     +-----------------------------------+----------------------------------------------------------------------------+
     | Additional Helpers                |                                                       	              |
     | - calculate_shares                | pool_id, assets                                      	              |
     | - call_on_liquidity_change_hook   | pool_id, initial_reserves, initial_issuance 	                              |
     | - call_on_trade_hook              | pool_id, asset_in, asset_out, initial_reserves    	                      |
     | - get_pool_state                  | pool_id, initial_reserves, initial_issuance          	              |
     +-----------------------------------+----------------------------------------------------------------------------+

```
***
### **3.3. EMA Oracle [pallet](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/ema-oracle) & [math](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/ema)**

- This pallet offers oracles (data feeds) for price, volume, and liquidity of different asset pairs. These oracles use exponential moving averages (EMA) to smooth out fluctuations and provide stable values over various timeframes. It does this by connecting data feeds from other pallets through callbacks, adding new data points to temporary storage for each block (time period) upon which, the temporary data is merged into permanent storage using the calculations. Oracles are accessed on-demand, and values are "fast-forwarded" to the last block for freshness.


```
                                                     EMA Oracle Pallet
                                                     |
                                                     |-- Config
                                                     |   |-- RuntimeEvent
                                                     |   |-- WeightInfo
                                                     |   |-- BlockNumberProvider
                                                     |   |-- SupportedPeriods
                                                     |   |-- MaxUniqueEntries
                                                     |
                                                     |-- Error
                                                     |   |-- TooManyUniqueEntries
                                                     |   |-- OnTradeValueZero
                                                     |
                                                     |-- Event
                                                     |
                                                     |-- Storage
                                                     |   |-- Accumulator
                                                     |   |-- Oracles
                                                     |
                                                     |-- GenesisConfig
                                                     |   |-- initial_data
                                                     |   |-- _marker
                                                     |
                                                     |-- Hooks
                                                     |   |-- on_initialize
                                                     |
                                                     |-- Call
                                                     |
                                                     |-- Impl
                                                     |   |-- on_entry
                                                     |   |-- on_trade
                                                     |   |-- on_liquidity_changed
                                                     |   |-- last_block_oracle
                                                     |   |-- update_oracles_from_accumulator
                                                     |   |-- update_oracle
                                                     |   |-- get_updated_entry
                                                     |
                                                     |-- OnCreatePoolHandler
                                                     |   |-- on_create_pool
                                                     |
                                                     |-- OnTradeHandler
                                                     |   |-- on_trade
                                                     |   |-- on_trade_weight
                                                     |
                                                     |-- OnLiquidityChangedHandler
                                                     |   |-- on_liquidity_changed
                                                     |   |-- on_liquidity_changed_weight
                                                     |
                                                     |-- Utility Functions
                                                     |   |-- determine_normalized_price
                                                |   |-- determine_normalized_volume
                                                |   |-- determine_normalized_liquidity
                                                |   |-- ordered_pair
                                                |
                                                |-- OracleError
                                                |   |-- NotPresent
                                                |   |-- SameAsset
                                                |
                                                |-- AggregatedOracle
                                                |   |-- get_entry
                                                |   |-- get_entry_weight
                                                |
                                                |-- AggregatedPriceOracle
                                                    |-- get_price
                                                    |-- get_price_weight

```
***
### **3.4. CircuitBreaker [pallet](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/circuit-breaker)**

- This pallet manages and restricts how much of an asset's liquidity can be traded, added, or removed within a single block. It tracks the liquidity trading, adding, and removing limits and resets the these limits to zero after each block.


**[Pallet lib](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/circuit-breaker/src/lib.rs)**

**Setting limits** - The `TechnicalOrigin` admin can set the trade volume, liquidity that can be added and liquidity that can be removed limits in the protocol. There are sanity checks in place to prevent extreme limits being imposed on the users. These are done by the `set_trade_volume_limit`, `set_add_liquidity_limit`, `set_remove_liquidity_limit` functions. 

```

+-----------------------------------------------------------------------------------------------------------------------------------------------------+
|                                                          CircuitBreaker.rs lib                                                                      |
+-----------------------------------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                                                     |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |                                                       TradeVolumeLimit                                                                         | |
|  |  					+-----------------------+  +------------------------+  +-------------------+                                | |
|  |  					| volume_in: T::Balance |  | volume_out: T::Balance |  | limit: T::Balance |                                | |
|  | 					+-----------------------+  +------------------------+  +-------------------+                                | |
|  |                                                                                                                                                | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |  |  update_amounts(&mut self, amount_in: T::Balance, amount_out: T::Balance) -> DispatchResult                                                 | |
|  |  |  check_outflow_limit(&self) -> DispatchResult                                                                                               | |
|  |  |  check_influx_limit(&self) -> DispatchResult                                                                                                | |
|  |  |  check_limits(&self) -> DispatchResult                                                                                                      | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|                                                                                                                                                     |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |                                                       LiquidityLimit                                                                           | |
|  |  					+-----------------------+  +-------------------+                                                            | |
|  | 					| liquidity: T::Balance |  | limit: T::Balance |                                                            | |
|  |  					+-----------------------+  +-------------------+                                                            | |
|  |                                                                                                                                                | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |  |  update_amount(&mut self, liquidity_in: T::Balance) -> DispatchResult                                                                       | |
|  |  |  check_limit(&self) -> DispatchResult                                                                                                       | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|                                                                                                                                                     |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |                                                       Pallet                                                                                   | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |  |  set_trade_volume_limit(origin: OriginFor<T>, asset_id: T::AssetId, trade_volume_limit: (u32, u32)) -> DispatchResult                       | |
|  |  |  set_add_liquidity_limit(origin: OriginFor<T>, asset_id: T::AssetId, liquidity_limit: Option<(u32, u32)>) -> DispatchResult                 | |
|  |  |  set_remove_liquidity_limit(origin: OriginFor<T>, asset_id: T::AssetId, liquidity_limit: Option<(u32, u32)>) -> DispatchResult              | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|                                                                                                                                                     |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |                                                       Events                                                                                   | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |  |  TradeVolumeLimitChanged { asset_id: T::AssetId, trade_volume_limit: (u32, u32) }                                                           | |
|  |  |  AddLiquidityLimitChanged { asset_id: T::AssetId, liquidity_limit: Option<(u32, u32)> }                                                     | |
|  |  |  RemoveLiquidityLimitChanged { asset_id: T::AssetId, liquidity_limit: Option<(u32, u32)> }                                                  | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|                                                                                                                                                     |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |                                                       Errors                                                                                   | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  |  |  InvalidLimitValue                                                                                                                          | |
|  |  |  LiquidityLimitNotStoredForAsset                                                                                                            | |
|  |  |  TokenOutflowLimitReached                                                                                                                   | |
|  |  |  TokenInfluxLimitReached                                                                                                                    | |
|  |  |  MaxLiquidityLimitPerBlockReached                                                                                                           | |
|  |  |  NotAllowed                                                                                                                                 | |
|  |  +---------------------------------------------------------------------------------------------------------------------------------------------+ |
|  +------------------------------------------------------------------------------------------------------------------------------------------------+ |
|                                                                                                                                                     |
+-----------------------------------------------------------------------------------------------------------------------------------------------------+


```
***
## **3. Codebase Overview**

- **Audit Information** - For the purpose of the security review, HydraDx comprises thirteen smart contracts totaling over 5273 RLoC. Its core design principle is composition, enabling efficient and flexible integration. The EMA oracle is used to generate token prices.

- **Documentation** - The codebase is divided into 4 major sections - Omnipool, StableSwap, EMA oracle, Circuit breaker. Each section has a dedicated README which discusses the major important parts of the contracts. The provided documentation provides brief overview of the omnipool as well as discusses the HydraDX protocol. The contracts are well commented, and explanations where given as to expected functionalities of each function.

- **Protocol Ecosystem** - The protocol's ecosystem  relies on the polkadot and its parachains, for deployments. The contracts are written in rust, and are to be compiled with cargo. 
 
- **Token Support** - The protocol mainly works with ERC20 tokens, including the LRNA and HDX protocol's hub and governance tokens. An ERC721 nft token is also in use which denotes a user's liquidity position in the omnipool.

- **Attack Vectors** - Various points of attack exists for a protocol of this size. Pricing manipulations, incorrect calculations in the math contracts, first depositor issues, general issues with AMMs and liquidity pools, etc.    

***

## **4. Centralization Risks** 
The protocol employs the actions of two main admins in the scope contracts, the `AuthorityOrigin` and the `TechnicalOrigin` While this admin is essentially trusted and assumed to be correctly configured, having such a centralized power puts the protocol at risks, some of which are: 
- Setting various protocol parameters extremely high or low to grief users.
- Maliciously removing assets and switching assets' tradeablity to grief users.

***
## **5. Systemic Risks** 

- Low engagement, can cause low liquidity in pools and consequently, affect protocol.
- External dependencies from compilers, polkachains and so on.
- Smart contract bugs which at best might be contained to erring contract and at worst could affect entire protocol.
- Non standard erc20 tokens as assets which can be a source of attack. 
- Slippage and deadline issues due to interactions with pools.

***
## **6. Recommendations**

- Setter functions should have two step updates should be implemented including zero address checks. A timelock can also be considered for these functions to give users time to react to protocol changes. Adding these fixes can help protect from admin mistakes and unexepected behaviour.
- A proper pause function can be implemented to protect users in cases of turbulent market situations and black swan events. It also helps prevent malicious activities or bugs from causing significant damage while the team investigates the potential issues without affecting users' funds.
- A deadline parameter can also be included to prevent transactions from being executed at an unfavorable time due to network congestion or other factors that could affect the outcome of the trade. By setting a deadline, users specify until what time they are willing to have their transaction included in the blockchain.
- NFT positon collections are not destroyed upon removal of a token in the omnipool contract, consider checking if this is desired, thus leaving as is or destroying the token collection after removing the token.
- Multiple fallback oracles should also be implemented and aggregated, with comparisons made with between the various returned prices. This helps ensure more accurate token pricing and protects from bad trades in case the main oracle becomes faulty.
- Consider upgrading the various cargo packages in use to stay up to date with latest security fixes.
***
## **7. Conclusions**

-  As an endnote, the codebase was pretty well designed, albeit hard to crack, due its fairly unconventional nautre. All the same, a number of risks were identified and they need to be fixed. Recommended measures should be implemented to protect the protocol from potential attacks. Timely audits and codebase cleanup should be conducted to keep the codebase fresh and up to date with evolving security times.
***
## **8. Resources**

- [C4 ReadMe](https://code4rena.com/audits/2024-02-hydradx#top)
- [Protocol overview documentation](https://docs.hydradx.io/)
- [HydraDX security documentation](https://github.com/galacticcouncil/HydraDX-security)
- [Contract visualizer](https://apidocs.bsx.fi/HydraDX)
- [Polkadot documentation](https://polkadot.network/development/docs/)
- [Rust Book](https://doc.rust-lang.org/book/)

### Time spent:
070 hours