# Analysis - HydraDX Contest

![HydraDX-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FZp3kgBd3EtV.0&w=256&q=75)

## Description overview of HydraDX Contest

HydraDX is a cutting-edge DeFi protocol designed to enhance liquidity within the `Polkadot` ecosystem. At its core, HydraDX operates as a cross-chain liquidity protocol built on Substrate, offering an open and permissionless platform for users. The protocol functions as a parachain within the `Polkadot` network, enabling native asset swaps within the ecosystem while also facilitating interoperability with other blockchain networks such as Ethereum.

One of the key innovations of `HydraDX` is its Omnipool, which serves as an AMM. Unlike traditional order book-based exchanges, where buyers and sellers create orders that are matched against each other, an AMM like the Omnipool provides liquidity through a single pool where assets are traded against the protocol itself. Liquidity providers contribute assets to the pool and earn rewards in return, helping to maintain liquidity for various trading pairs.

HydraDX addresses several challenges faced by existing `DEX` protocols, such as slippage, impermanent loss, and front-running. To overcome these challenges, `HydraDX` implements several innovative features. The Omnipool consolidates liquidity for all assets into a single pool, reducing slippage and improving trading efficiency. Additionally, the protocol employs an order matching engine connected to the AMM pool via an oracle, allowing for more efficient trade execution. Transaction fees can be paid in any currency, enhancing flexibility for users, and the protocol supports dollar-cost averaging, automating asset purchases at regular intervals regardless of price fluctuations. Looking ahead, HydraDX is exploring additional features such as zkRollups for transaction scalability, resistance mechanisms against front-running, and the introduction of new financial instruments like lending, derivatives, and synthetics. The protocol also utilizes a LBP model to distribute tokens and bootstrap liquidity, ensuring a fair and sustainable ecosystem for users and the community.


## System Overview

### Scope

#### Omnipool

1. **lib.rs**: This contract implements a DEX using an AMM model on the Substrate framework. It allows users to trade assets without intermediaries by pooling liquidity from providers and using on-chain math functions to calculate state changes during operations like `adding/removing` liquidity or executing swaps. Key functions allow managing the assets, positions represented as NFTs, and trades, while adhering to parameters like price barriers and dynamic fees. Precise updates to the pool reserves and hub asset imbalance are performed through hooks and events provide transparency. The non-custodial and low fee nature of the omnipool model enables greater decentralization and accessibility for traders compared to order-book based `HydraDX` alternatives.

   Here's a breakdown of the key functions:

   - **Protocol Account**:

     - `protocol_account()`: Returns the protocol account address, which is used for managing the omnipool's assets.

   - **Asset Management**:

     - `load_asset_state(asset_id)`: Retrieves the state of an asset, including its reserve and tradability.
     - `set_asset_state(asset_id, new_state)`: Sets the new state of an asset.
     - `add_asset(asset_id, state)`: Adds a new asset to the omnipool.
     - `update_asset_state(asset_id, delta)`: Updates the state of an asset with the given delta changes.
     - `remove_asset(asset_id)`: Removes an asset from the omnipool.
     - `allow_assets(asset_in, asset_out)`: Checks if the given assets can be traded based on their tradability.
     - `sell_hub_asset(origin, who, asset_out, amount, limit)`: Swaps hub asset for asset_out.
     - `buy_asset_for_hub_asset(origin, who, asset_out, amount, limit)`: Swaps asset for hub asset.
     - `buy_hub_asset(who, asset_in, amount, limit)`: Buys hub asset from the pool.
     - `sell_asset_for_hub_asset(who, asset_in, amount, limit)`: Sells asset for hub asset.

   - **Position Management:**

     - `create_and_mint_position_instance(owner)`: Generates an NFT instance ID and mints an NFT for the position.
     - `set_position(position_id, position)`: Inserts or updates a position with the given data.
     - `load_position(position_id, owner)`: Loads a position and checks its owner.

   - **Other Functions:**

     - `get_hub_asset_balance_of_protocol_account()`: Returns the hub asset balance of the protocol account.
     - `is_hub_asset_allowed(operation)`: Checks if the given operation is allowed for the hub asset.
     - `exists(asset)`: Checks if an asset exists in the omnipool.
     - `process_trade_fee(trader, asset, amount)`: Calls the `on_trade_fee` hook and ensures that no more than the fee amount is transferred.
     - `process_hub_amount(amount, dest)`: Processes the given hub amount by transferring it to the specified destination or burning it if the transfer fails.

2. **types.rs**: The codebase defines types and structures used in an Omnipool.

   Here's a breakdown of the key functions:

   - **Types:**

     - **`Balance`**: This type represents the balance of an asset and is implemented as a `u128`.
     - **`Price`**: This type represents the price of an asset and is implemented as a `FixedU128`.
     - **`Tradability`**: This bitflag type indicates whether an asset can be bought, sold, or have liquidity added or removed.
     - **`AssetState`**: This type stores the state of an asset in the Omnipool, including its hub reserve, LP shares, protocol shares, weight cap, and tradability.
     - **`Position`**: This type represents a liquidity position in the Omnipool, including the asset ID, amount added, LP shares owned, and the price at which liquidity was provided.
     - **`SimpleImbalance`**: This type represents an imbalance, which can be positive or negative.
     - **`AssetReserveState`**: This type stores the state of an asset reserve, including its reserve, hub reserve, LP shares, protocol shares, weight cap, and tradability.

   - **Functions:**

     - **`impl From<AssetReserveState<Balance>> for AssetState<Balance>`**: Converts an `AssetReserveState` to an `AssetState`.

     - **`impl From<(MathReserveState<Balance>, Permill, Tradability)> for AssetState<Balance>`**: Converts a tuple of `MathReserveState`, `Permill`, and `Tradability` to an `AssetState`.

     - **`impl From<&Position<Balance, AssetId>> for hydra_dx_math::omnipool::types::Position<Balance>`**: Converts a `Position` to a `Position` in the `hydra_dx_math::omnipool::types` module.

     - **`impl Position<Balance, AssetId>`**: Provides methods to update the position state with delta changes.

     - **`impl Add<Balance> for SimpleImbalance<Balance>`**: Adds a `Balance` to a `SimpleImbalance`.

     - **`impl Sub<Balance> for SimpleImbalance<Balance>`**: Subtracts a `Balance` from a `SimpleImbalance`.

     - **`impl From<&AssetReserveState<Balance>> for MathReserveState<Balance>`**: Converts an `AssetReserveState` to a `MathReserveState`.

     - **`impl From<AssetReserveState<Balance>> for MathReserveState<Balance>`**: Converts an `AssetReserveState` to a `MathReserveState`.

     - **`impl From<(&AssetState<Balance>, Balance)> for AssetReserveState<Balance>`**: Converts a tuple of `AssetState` and `Balance` to an `AssetReserveState`.

     - **`impl From<(AssetState<Balance>, Balance)> for AssetReserveState<Balance>`**: Converts a tuple of `AssetState` and `Balance` to an `AssetReserveState`.

     - **`impl AssetReserveState<Balance>`**: Provides methods to calculate the price and weight cap of an asset, and update the asset state with delta changes.

   - **Key Features:**

   - The `Tradability` bitflag provides a convenient way to represent the tradability of an asset.
   - The `SimpleImbalance` type provides a simple way to represent imbalances, which can be positive or negative.
   - The `AssetReserveState` type provides a comprehensive representation of an asset reserve's state in the Omnipool.
   - The `Position` type provides a complete representation of a liquidity position in the Omnipool.
   - The functions provided allow for the conversion between different representations of assets, liquidity positions, and imbalances.

3. **traits.rs**: This contract defines `traits`, `structs`, and implementations facilitating the management of an Omnipool, including hooks for liquidity changes and trades, external price fetching, and enforcing price constraints.

   Here's a breakdown of the key functions:

   - **Traits:**

     - **`OmnipoolHooks`**: This trait defines hooks that can be implemented to perform custom actions when liquidity is changed or a trade occurs in the Omnipool.
     - **`ExternalPriceProvider`**: This trait defines an interface for an external price provider that can be used to get the price of an asset pair.
     - **`ShouldAllow`**: This trait defines a way to validate whether a price change is allowed.

   - **Types:**

     - **`AssetInfo`**: This type stores information about an asset before and after a liquidity change or trade.
     - **`EnsurePriceWithin`**: This type implements the `ShouldAllow` trait and ensures that the price of an asset is within a certain range of the current spot price and the external oracle price.

   - **Key Features:**

     - The `OmnipoolHooks` trait provides a way to hook into the Omnipool protocol and perform custom actions when liquidity is changed or a trade occurs.
     - The `ExternalPriceProvider` trait provides a way to get the price of an asset pair from an external source.
     - The `ShouldAllow` trait provides a way to validate whether a price change is allowed.
     - The `EnsurePriceWithin` type implements the `ShouldAllow` trait and ensures that the price of an asset is within a certain range of the current spot price and the external oracle price.

#### Omnipool Math

1. **math.rs**: This codebase implements a set of functions for calculating the delta changes in the state of an asset pool when various liquidity-related operations are performed, such as selling, buying, adding liquidity, removing liquidity, and calculating the total value locked (TVL) and cap difference.

   Here's a breakdown of the key functionality of the contract:

   - **Selling an asset:**

     - `calculate_sell_state_changes` calculates the delta changes in the state of the asset pool when an asset is sold. It takes the current state of the asset in and out, the amount being sold, and the asset and protocol fees as input. It calculates the delta changes in the hub and reserve balances of both assets, the delta change in the imbalance, and the fee amounts.
     - `calculate_sell_hub_state_changes` calculates the delta changes in the state of the asset pool when the asset being sold is the Hub asset. It takes the current state of the asset out, the amount of Hub asset being sold, the asset fee, the current imbalance, and the total hub reserve as input. It calculates the delta changes in the reserve and hub reserve balances of the asset out, the delta change in the imbalance, and the fee amount.

   - **Buying an asset:**

     - `calculate_buy_for_hub_asset_state_changes` calculates the delta changes in the state of the asset pool when the asset being bought is the Hub asset. It takes the current state of the asset out, the amount of asset out being bought, the asset fee, the current imbalance, and the total hub reserve as input. It calculates the delta changes in the reserve and hub reserve balances of the asset out, the delta change in the imbalance, and the fee amount.
     - `calculate_buy_state_changes` calculates the delta changes in the state of the asset pool when an asset is bought. It takes the current state of the asset in and out, the amount being bought, the asset and protocol fees, and the current imbalance as input. It calculates the delta changes in the hub and reserve balances of both assets, the delta change in the imbalance, and the fee amounts.

   - **Adding liquidity:**

     - `calculate_add_liquidity_state_changes` calculates the delta changes in the state of the asset pool when liquidity is added. It takes the current state of the asset, the amount being added, the current imbalance, and the total hub reserve as input. It calculates the delta changes in the hub and reserve balances of the asset, the delta change in the shares, and the delta change in the imbalance.

   - **Removing liquidity:**

     - `calculate_remove_liquidity_state_changes` calculates the delta changes in the state of the asset pool when liquidity is removed. It takes the current state of the asset, the shares being removed, the position from which liquidity should be removed, the current imbalance, the total hub reserve, and the withdrawal fee as input. It calculates the delta changes in the hub and reserve balances of the asset, the delta change in the shares, the delta change in the imbalance, the amount of Hub asset transferred to the LP, and the delta changes in the position's reserve and shares.

   - **Calculating TVL and cap difference:**

     - `calculate_tvl` calculates the total value locked (TVL) in the asset pool. It takes the hub reserve and the stable asset reserve as input. It calculates the TVL by multiplying the hub reserve by the stable asset reserve and dividing by the stable asset's hub reserve.
     - `calculate_cap_difference` calculates the difference between the current weight of an asset in the pool and its weight cap. It takes the current state of the asset, the asset cap, and the total hub reserve as input. It calculates the weight cap, the maximum allowed hub reserve, the price of the asset, and the cap difference.
     - `calculate_tvl_cap_difference` calculates the difference between the current TVL of the asset pool and its TVL cap. It takes the current state of the asset, the current state of the stable asset, the TVL cap, and the total hub reserve as input. It calculates the TVL cap, the maximum allowed hub reserve, the price of the asset, and the TVL cap difference.
     - `verify_asset_cap` verifies if the weight of an asset in the pool exceeds its weight cap. It takes the current state of the asset, the asset cap, the hub amount being added, and the total hub reserve as input. It calculates the weight of the asset and compares it to the weight cap.

2. **types.rs**: This contract defines structs and implementations related to asset reserves, liquidity pools, and trading mechanics, including functions for updating asset states, calculating prices, and handling balance updates.

   Here's a breakdown of the key functions:

   - **AssetReserveState**: Represents the current state of an asset in the Omnipool, including its reserve, hub reserve, and LP shares.
   - **BalanceUpdate**: Indicates whether the balance of an asset should be increased or decreased, and by how much.
   - **AssetStateChange**: Tracks delta changes in asset state, such as reserve, hub reserve, and shares.
   - **TradeFee**: Stores information about trade fees, including the asset fee and protocol fee.
   - **TradeStateChange**: Represents the changes in asset states after a trade is executed, including fee information.
   - **LiquidityStateChange**: Tracks delta changes in asset states and other parameters after liquidity is added or removed.
   - **Position**: Represents the amount of asset added to the Omnipool and the corresponding LP shares owned by the LP.

#### Stableswap

1. **lib.rs**: It implements a Curve-style stablecoin AMM with up to 5 assets in a pool. Pools are created by an authority and have a pricing formula based on amplification. Also implements a stableswap pallet for the HydraDX runtime.The stableswap pallet allows for the creation of liquidity pools for stablecoins, which are asset that are pegged to a usd. Stablecoins are designed to be less volatile than other assets, making them more suitable for use in everyday transactions. The stableswap pallet uses a constant product formula to calculate the price of assets in a pool. This formula ensures that the price of an asset in a pool is always proportional to the amount of that asset in the pool.

   Here's a breakdown of the key functionality:

   - **Pool creation:** Pools can be created by any account that has the `AuthorityOrigin` role. When a pool is created, the creator must specify the following information:
     - The pool's share asset: The share asset is a token that represents ownership of a share of the pool.
     - The pool's assets: The pool's assets are the stablecoins that will be traded in the pool.
     - The pool's amplification: The pool's amplification is a parameter that can be used to adjust the shape of the constant product curve.
     - The pool's trade fee: The pool's trade fee is a fee that is charged on all trades executed in the pool.
   - **Liquidity addition:** Liquidity can be added to a pool by any account that has the `LiquidityProviderOrigin` role. When liquidity is added to a pool, the provider must specify the following information:
     - The pool's share asset: The share asset is the token that represents ownership of a share of the pool.
     - The pool's assets: The pool's assets are the stablecoins that will be traded in the pool.
     - The amount of liquidity to add: The amount of liquidity to add is the amount of each asset that the provider is willing to contribute to the pool.
   - **Liquidity removal:** Liquidity can be removed from a pool by any account that has the `LiquidityProviderOrigin` role. When liquidity is removed from a pool, the provider must specify the following information:
     - The pool's share asset: The share asset is the token that represents ownership of a share of the pool.
     - The pool's assets: The pool's assets are the stablecoins that will be traded in the pool.
     - The amount of liquidity to remove: The amount of liquidity to remove is the amount of each asset that the provider wants to withdraw from the pool.
   - **Trading:** Trades can be executed in a pool by any account that has the `TraderOrigin` role. When a trade is executed, the trader must specify the following information:
     - The pool's share asset: The share asset is the token that represents ownership of a share of the pool.
     - The pool's assets: The pool's assets are the stablecoins that will be traded in the pool.
     - The amount of the input asset: The amount of the input asset is the amount of the asset that the trader is willing to trade.
     - The amount of the output asset: The amount of the output asset is the amount of the asset that the trader wants to receive.

2. **types.rs**: This codebase defines data structures and traits related to the management of stable pools . It includes representations of pool properties, asset amounts, tradability flags, and interfaces for interacting with the oracle and calculating weights.

   Here's a breakdown of the key functionality:

   - **PoolInfo**

     - The `PoolInfo` struct defines the properties of a stable pool.
     - It includes the following fields:
       - `assets`: List of asset IDs in the pool.
       - `initial_amplification`: Initial amplification parameter.
       - `final_amplification`: Final amplification parameter.
       - `initial_block`: Block number at which the pool was created.
       - `final_block`: Block number at which the amplification parameter will reach its final value.
       - `fee`: Trade fee to be withdrawn on sell/buy operations.
     - It provides methods to find an asset by ID and check if the pool is valid (has at least two unique assets).

   - **AssetAmount**

     - The `AssetAmount` struct represents the amount of an asset with a specified asset ID.
     - It can be converted to and from a `u128` balance value.

   - **Tradability**

     - The `Tradability` flag indicates whether an asset can be bought, sold, or have liquidity added or removed.
     - It is represented as a bitmask with the following flags:
       - `FROZEN`: Asset is frozen and no operations are allowed.
       - `SELL`: Asset can be sold into the stable pool.
       - `BUY`: Asset can be bought from the stable pool.
       - `ADD_LIQUIDITY`: Liquidity of the asset can be added.
       - `REMOVE_LIQUIDITY`: Liquidity of the asset can be removed.
     - By default, all operations are allowed.

   - **PoolState**

     - The `PoolState` struct tracks the state of a stable pool before and after an operation.
     - It includes the following fields:
       - `assets`: List of asset IDs in the pool.
       - `before`: Balances of assets before the operation.
       - `after`: Balances of assets after the operation.
       - `delta`: Difference in balances between before and after.
       - `issuance_before`: Total issuance before the operation.
       - `issuance_after`: Total issuance after the operation.
       - `share_prices`: Share prices of the assets in the pool.

   - **StableswapHooks**

   - The `StableswapHooks` trait defines an interface for interacting with the oracle and calculating weights.
   - It includes the following methods:
     - `on_liquidity_changed`: Called when liquidity is added or removed from a pool.
     - `on_trade`: Called when a trade occurs in a pool.
     - `on_liquidity_changed_weight`: Calculates the weight for liquidity changed operations.
     - `on_trade_weight`: Calculates the weight for trade operations.
   - The default implementation of this trait does nothing and returns zero weight.

#### Stableswap Math

1. **math.rs**: The contract implementing functions for a stableswap pool, used for AMM with multiple assets, incorporating features such as calculating asset amounts for liquidity provision, trading between assets with fees, and determining the number of shares to be distributed to liquidity providers based on their contribution. The contract implements formulas for calculating the D invariant, representing the product of reserves to maintain stable trading ratios between assets, and the Y reserve value, used in trading calculations within the pool. These calculations are performed using mathematical operations and iterative algorithms to ensure accuracy and stability within the automated market making system.

   Here's a breakdown of the key function:

   - **calculate_d**: calculates the pool's `D` invariant
   - **calculate_y**: calculates new reserve amounts
   - **calculate_shares**: handles share minting
   - **normalize_value**: ensures consistent precision
   - **calculate_out_given_in/in_given_out** handle trades

2. **types.rs**: This contract an implementation of the StableSwap for calculating the amount of tokens to be received or sent to a liquidity pool given the amount of tokens to be sent or received from the pool, respectively. That is use a mathematical formula that ensures that the ratio of the reserves of the different assets in the pool remains constant, even as tokens are added or removed from the pool.

   Here's a breakdown of the key functionality:

   - **calculate_out_given_in**: Calculates the amount of tokens to be received from the pool given the amount of tokens to be sent to the pool.
   - **calculate_in_given_out**: Calculates the amount of tokens to be sent to the pool given the amount of tokens to be received from the pool.
   - **calculate_out_given_in_with_fee**: Calculates the amount of tokens to be received from the pool given the amount of tokens to be sent to the pool, taking into account a fee.
   - **calculate_in_given_out_with_fee**: Calculates the amount of tokens to be sent to the pool given the amount of tokens to be received from the pool, taking into account a fee.
   - **calculate_shares**: Calculates the amount of shares to be given to a liquidity provider after they have provided liquidity to the pool.
   - **calculate_shares_for_amount**: Calculates the amount of shares to be given to a liquidity provider after they have provided a specific amount of a single asset to the pool.
   - **calculate_withdraw_one_asset**: Calculates the amount of a specific asset to be withdrawn from the pool by a liquidity provider, given the amount of shares they have in the pool.
   - **calculate_add_one_asset**: Calculates the amount of a specific asset that needs to be added to the pool by a liquidity provider in order to receive a specific number of shares in the pool.
   - **calculate_d**: Calculates the "D" invariant of the StableSwap algorithm, which is a mathematical value that remains constant as tokens are added or removed from the pool.
   - **calculate_y_given_in**: Calculates the new reserve of an asset in the pool given the amount of that asset to be added to the pool.
   - **calculate_y_given_out**: Calculates the new reserve of an asset in the pool given the amount of that asset to be removed from the pool.

#### EMA Oracle

1. **lib.rs**: The code is implementation of an `EMA` oracle for the protocol. The EMA oracle is used to track the price, volume, and liquidity of assets traded on the `HydraDX` over time. The oracle is implemented as a pallet in the Substrate framework.

   Here's a breakdown of the key functionality:

   - **on_trade**: This function is called when a trade occurs on the `HydraDX`.
     It updates the EMA oracle with the new trade data.
   - **on_liquidity_changed**: This function is called when the liquidity of an asset pair changes on the `HydraDX`.
     It updates the EMA oracle with the new liquidity data.
   - **get_entry**: This function is used to retrieve the current value of the EMA oracle for a given asset pair and period.
   - **get_price**: This function is used to retrieve the current price of an asset pair from the EMA oracle.

2. **types.rs**: The `types.rs` contract using the exponential moving average (EMA). It maintains a set of EMA oracles for each asset pair and period, and updates them whenever a trade or liquidity change occurs on the `HydraDX`.

   Here's a breakdown of the key functionality:

   - **OracleEntry**: This struct represents a single oracle entry, which includes the price, volume, liquidity, and updated timestamp.
   - **calculate_new_by_integrating_incoming**: This function calculates a new oracle entry by integrating the incoming data with the previous oracle entry.
   - **update_to_new_by_integrating_incoming**: This function updates the current oracle entry with the new oracle entry calculated by `calculate_new_by_integrating_incoming`.
   - **calculate_current_from_outdated**: This function calculates the current oracle entry from an outdated oracle entry.
   - **update_outdated_to_current**: This function updates the current oracle entry with the current oracle entry calculated by `calculate_current_from_outdated`.

#### Ema Oracle Math

1. **math.rs**: The contract defines functions for calculating exponential moving averages (EMAs) and performing weighted averages for oracle values in a `HydraDX`.

   - **EMA:** A type of moving average that gives more weight to recent values.
   - **Oracle:** A service that provides external data (e.g., asset prices) to a blockchain.
   - **Weighted average:** A calculation where each value is multiplied by a weight before being averaged.

   Here's a breakdown of the key functiona:

   - **`calculate_new_by_integrating_incoming`:** Calculates new oracle values by integrating incoming values with previous values, using a specified smoothing factor.
   - **`update_outdated_to_current`:** Updates outdated oracle values to current values, using a smoothing factor and the number of iterations since the outdated values were calculated.
   - **`iterated_price_ema`:** Calculates the iterated EMA for prices.
   - **`iterated_balance_ema`:** Calculates the iterated EMA for balances.
   - **`iterated_volume_ema`:** Calculates the iterated EMA for volumes.
   - **`iterated_liquidity_ema`:** Calculates the iterated EMA for liquidity values.
   - **`exp_smoothing`:** Calculates the smoothing factor for a given period.
   - **`smoothing_from_period`:** Calculates the smoothing factor based on a specified period.
   - **`price_weighted_average`:** Calculates a weighted average for prices, giving more weight to the incoming value.
   - **`balance_weighted_average`:** Calculates a weighted average for balances, giving more weight to the incoming value.
   - **`volume_weighted_average`:** Calculates a weighted average for volumes, giving more weight to the incoming value.
   - **`liquidity_weighted_average`:** Calculates a weighted average for liquidity values, giving more weight to the incoming value.

#### Circuit breaker

1. **lib.rs**: This contract is pallet for the Substrate framework. It defines a pallet named `pallet` that manages circuit breakers for trade volume and liquidity limits in a `HydraDX`.

   Here's a breakdown of the key functiona:

   - **Config Trait**: The `Config` trait defines the requirements that the pallet has on the runtime environment. It includes:

     - `RuntimeEvent`: The type of events that the pallet can emit.
     - `AssetId`: The type representing the identifier of an asset.
     - `Balance`: The type representing the balance of an asset.
     - `TechnicalOrigin`: The origin that is allowed to change trade volume limits.
     - `WhitelistedAccounts`: The list of accounts that bypass liquidity limit checks.
     - `DefaultMaxNetTradeVolumeLimitPerBlock`: The default maximum percentage of a pool's liquidity that can be traded in a block.
     - `DefaultMaxAddLiquidityLimitPerBlock`: The default maximum percentage of a pool's liquidity that can be added in a block.
     - `DefaultMaxRemoveLiquidityLimitPerBlock`: The default maximum percentage of a pool's liquidity that can be removed in a block.
     - `OmnipoolHubAsset`: The asset ID of the Omnipool's hub asset.
     - `WeightInfo`: The weight information for the pallet's extrinsics.

   - **Pallet Implementation**: The implementation of the pallet includes various functions and methods:

     - `initialize_trade_limit`: Initializes the trade volume limit for an asset if it doesn't exist.
     - `calculate_and_store_liquidity_limits`: Calculates and stores the liquidity limits for an asset if they don't exist.
     - `ensure_and_update_trade_volume_limit`: Ensures that the trade volume limit for an asset is not exceeded and updates the allowed trade volume limit for the current block.
     - `ensure_and_update_add_liquidity_limit`: Ensures that the add liquidity limit for an asset is not exceeded and updates the allowed add liquidity amount for the current block.
     - `ensure_and_update_remove_liquidity_limit`: Ensures that the remove liquidity limit for an asset is not exceeded and updates the allowed remove liquidity amount for the current block.
     - `validate_limit`: Validates a limit value.
     - `calculate_limit`: Calculates the limit value based on the provided liquidity and limit ratio.
     - `ensure_pool_state_change_limit`: Ensures that the trade volume limit is not exceeded when performing a pool state change.
     - `ensure_add_liquidity_limit`: Ensures that the add liquidity limit is not exceeded.
     - `ensure_remove_liquidity_limit`: Ensures that the remove liquidity limit is not exceeded.
     - `is_origin_whitelisted_or_root`: Checks if the provided origin is whitelisted or is the root account.

### Roles

#### Omnipool

1. **lib.rs**:

2. **types.rs**

   - **Tradability**:

     - Role: Represents the tradability status of an asset within the Omnipool.
     - Responsibilities: Defines whether an asset can be bought or sold into the Omnipool and whether liquidity can be added or removed.
     - Relevant Code: `Tradability` enum and its associated methods.

   - **AssetState**:

     - Role: Represents the state of an asset within the Omnipool.
     - Responsibilities: Stores various parameters related to an asset's state, such as reserves, shares, weight cap, and tradability status.
     - Relevant Code: `AssetState` struct and its associated methods for conversion and updating state.

   - **Position**:

     - Role: Represents a position in the Omnipool, indicating when liquidity was provided for an asset at a particular price.
     - Responsibilities: Stores information about the asset, the amount added to the pool, LP shares owned, and the price at which liquidity was provided.
     - Relevant Code: `Position` struct and its associated methods.

   - **SimpleImbalance**:

     - Role: Represents an imbalance, which can be positive or negative.
     - Responsibilities: Provides a simple way to handle positive or negative imbalances.
     - Relevant Code: `SimpleImbalance` struct and its associated methods for addition and subtraction.

   - **AssetReserveState**:

     - Role: Represents the state of an asset pool reserve within the Omnipool.
     - Responsibilities: Stores information about the asset reserve, hub reserve, shares, weight cap, and tradability status.
     - Relevant Code: `AssetReserveState` struct and its associated methods for conversion, calculating price, and updating state.

3. **traits.rs**

   - **AssetInfo**:

     - Role: Represents information about an asset's state before and after a change.
     - Responsibilities: Stores details such as asset ID, reserve states, delta changes, and a flag indicating safe withdrawal.
     - Relevant Code: `AssetInfo` struct and its associated methods.

   - **OmnipoolHooks**:

     - Role: Defines hooks for handling liquidity changes, trades, and hub asset trades within the Omnipool.
     - Responsibilities: Provides methods to handle various actions related to liquidity changes, trades, and hub asset trades.
     - Relevant Code: `OmnipoolHooks` trait and its associated methods.

   - **ExternalPriceProvider**:

     - Role: Defines an interface for fetching external price information.
     - Responsibilities: Provides a method to get the price of an asset pair from an external oracle.
     - Relevant Code: `ExternalPriceProvider` trait and its associated method.

   - **ShouldAllow**:

     - Role: Defines a trait for enforcing conditions or permissions, particularly related to asset price changes.
     - Responsibilities: Provides a method to ensure that a price change is allowed based on certain conditions.
     - Relevant Code: `ShouldAllow` trait and its associated method.

   - **EnsurePriceWithin**:
     - Role: Implements the `ShouldAllow` trait to ensure that the price change is within certain bounds.
     - Responsibilities: Enforces a constraint on the allowable price change based on the current spot price, external oracle price, and a maximum allowed difference.
     - Relevant Code: `EnsurePriceWithin` struct and its implementation of the `ShouldAllow` trait.

#### Omnipool Math

1. **math.rs**:

   - **Trade Calculations**:

     - Roles: `calculate_sell_state_changes`, `calculate_sell_hub_state_changes`, `calculate_buy_for_hub_asset_state_changes`, `calculate_buy_state_changes`
     - Responsibilities: These functions are responsible for calculating the delta changes in asset reserves and other parameters when trades occur, such as selling or buying assets.

   - **Liquidity Operations**:

     - Roles: `calculate_add_liquidity_state_changes`, `calculate_remove_liquidity_state_changes`
     - Responsibilities: These functions handle the calculations for adding and removing liquidity from the pool, including updating asset reserves, shares, and other relevant parameters.

   - **Fee Calculations**:

     - Roles: `calculate_fee_amount_for_buy`, `calculate_withdrawal_fee`
     - Responsibilities: These functions calculate the fees associated with trades and withdrawals, considering parameters such as asset amounts, fees, and protocol-specific configurations.

   - **Imbalance Handling**:

     - Roles: `calculate_imbalance_in_hub_swap`, `calculate_delta_imbalance`
     - Responsibilities: These functions handle the calculation of imbalances that may occur during trades or liquidity operations, ensuring the proper adjustment of reserves and other parameters to maintain stability.

   - **Price Calculations**:

     - Roles: `calculate_spot_sprice`, `calculate_lrna_spot_sprice`
     - Responsibilities: These functions calculate spot prices for assets based on their reserve states, which are essential for determining trade ratios and other financial metrics.

   - **Capacity and Cap Management**:

     - Roles: `calculate_cap_difference`, `calculate_tvl_cap_difference`, `verify_asset_cap`
     - Responsibilities: These functions manage the capacity and cap constraints of assets within the system, ensuring that they do not exceed predefined limits and handling adjustments based on reserve states and total hub reserves.

2. **types.rs**

   - **AssetReserveState**: Represents the state of an asset within the system. Roles associated with this struct include:

     - Keeper of asset reserves: Tracks the quantity of the asset in the omnipool and hub reserves.
     - Keeper of LP shares: Tracks the quantity of LP shares for the asset and LP shares owned by the protocol.
     - Calculator of asset price: Provides functions to calculate the price of the asset in terms of the hub asset.

   - **AssetStateChange**: Represents the delta changes of asset state. Roles associated with this struct include:

     - Tracker of changes: Tracks changes in reserve, hub reserve, shares, and protocol shares.

   - **TradeFee**: Contains information about trade fee amounts. Roles associated with this struct include:

     - Keeper of fee amounts: Tracks asset fees and protocol fees associated with trades.

   - **TradeStateChange**: Represents the delta changes after a trade is executed. Roles associated with this struct include:

     - Tracker of trade effects: Tracks changes in asset in, asset out, delta imbalance, HDX hub amount, and fees.

   - **HubTradeStateChange**: Represents delta changes after a trade with hub asset is executed. Roles associated with this struct include:

     - Tracker of hub asset trade effects: Tracks changes in asset, delta imbalance, and fees for trades involving the hub asset.

   - **LiquidityStateChange**: Represents delta changes after adding or removing liquidity. Roles associated with this struct include:

     - Tracker of liquidity changes: Tracks changes in asset reserves, delta imbalance, delta position reserves, delta position shares, and LP hub amount.

   - **Position**: Represents a position with regards to liquidity provision. Roles associated with this struct include:

     - Keeper of position information: Tracks the amount of asset added to the omnipool, LP shares owned, and the price at which liquidity was provided.

   - **I129**: Represents a signed integer. Roles associated with this struct include:

     - Keeper of signed integer value: Stores a signed integer value along with a flag indicating its sign.

#### Stableswap

1. **lib.rs**:

   1. **Authority Origin**: This role is designated to the origin that has the authority to create a new pool. It is specified in the `Config` trait as `type AuthorityOrigin`.

   2. **Liquidity Provider (LP)**: Liquidity providers are users who add liquidity to the pool by providing assets. They are the origin of functions like `add_liquidity` and `remove_liquidity`.

   3. **Pool Manager**: The pool manager is responsible for managing and updating pool parameters such as fees and amplification. The pool manager's role is typically associated with the `Authority Origin`.

   4. **System Account**: The system account, represented by `frame_system::Config::AccountId`, may have implicit roles in the contract, such as executing transactions and maintaining system-level operations.

   5. **Share Token Holder**: Share token holders are users who receive shares in exchange for providing liquidity to the pool. These shares represent the LP's ownership stake in the pool.

2. **types.rs**

   1. **PoolInfo**: Represents the properties of a liquidity pool. Roles associated with this struct include:

      - Configuration keeper: Stores parameters related to the liquidity pool such as assets, amplification, fee, and block numbers.
      - Validator: Validates the validity of the pool configuration.

   2. **AssetAmount**: Represents the amount of an asset. Roles associated with this struct include:

      - Data holder: Stores information about the asset ID and its amount.

   3. **Tradability**: Represents the tradability flags for assets. Roles associated with this struct include:

      - Permission manager: Controls the tradability permissions for different operations such as buying, selling, adding liquidity, and removing liquidity.

   4. **BenchmarkHelper**: Trait for benchmarking purposes. Roles associated with this trait include:

      - Benchmark utility: Provides functions for benchmarking asset registration.

   5. **PoolState**: Represents the state of a liquidity pool. Roles associated with this struct include:

      - State keeper: Stores information about the assets in the pool, their balances before and after a transaction, issuance amounts, and share prices.

   6. **StableswapHooks**: Trait for interacting with the stableswap module. Roles associated with this trait include:

      - Oracle updater: Defines functions for updating the oracle with liquidity changes and trades.
      - Weight calculator: Calculates the weight associated with liquidity changes and trades for benchmarking.

   7. **Default implementations** (`impl` blocks): Provide default behavior for certain structs and traits. Roles associated with these blocks include:
      - Default behavior provider: Defines default implementations for certain functionalities.

#### Stableswap Math

1. **math.rs**:

The roles represented in the provided Rust code are primarily related to managing and interacting with a stableswap pool, which is a type of automated market maker (AMM) commonly used in decentralized finance (DeFi) platforms. Here's a breakdown of the roles associated with the functions in the code:

1. **Liquidity Providers (LPs)**:

   - LPs provide liquidity to the stableswap pool by depositing assets.
   - They receive shares representing their ownership in the pool.
   - `calculate_shares` and `calculate_shares_for_amount` functions calculate the amount of shares to be given to LPs based on the assets they provide and the fees involved.

2. **Traders**:

   - Traders interact with the stableswap pool to swap assets.
   - They may also provide liquidity to the pool and receive shares in return.
   - Functions like `calculate_out_given_in`, `calculate_in_given_out`, `calculate_out_given_in_with_fee`, and `calculate_in_given_out_with_fee` are used by traders to determine the amounts they will receive or need to send for a given trade, taking into account fees.

3. **Stableswap Pool**:

   - The stableswap pool facilitates asset swaps and provides liquidity to traders.
   - It maintains reserves of various assets and calculates prices and fees for trades.
   - Functions such as `calculate_d`, `calculate_ann`, `calculate_amplification`, `normalize_reserves`, and `normalize_value` are involved in managing the stableswap pool's internal state and performing calculations related to trading and liquidity provision.

4. **Governance**:

   - Governance may set parameters such as the amplification factor and fee percentage for the stableswap pool.
   - `calculate_amplification` function calculates the current amplification value based on specified parameters and the current block number, which could be set by governance.

5. **Developers**:
   - Developers maintain and improve the stableswap pool's functionality.
   - They implement and optimize algorithms for calculating swap amounts, share prices, fees, and other parameters.

Overall, these roles work together to ensure the efficient operation of the stableswap pool, providing liquidity to traders while incentivizing LPs with fees and share ownership in the pool. 2. **types.rs**:

The roles represented in the provided Rust code are related to managing asset reserves within a system. Here's an overview:

1. **Asset Reserves Manager**:

   - This role manages the reserves of various assets within the system.
   - The `AssetReserve` struct represents each asset reserve, containing information about the reserve amount and the number of decimals.
   - The manager initializes and updates the reserves as needed.
   - It ensures that the reserves are correctly represented and can be queried when necessary.

2. **Asset Reserves Users**:

   - Users of the system interact with asset reserves for various purposes such as trading, providing liquidity, or obtaining information.
   - They may query the state of asset reserves to understand the available liquidity or perform calculations based on reserve amounts.
   - The `AssetReserve` struct provides methods like `new` and `is_zero` for users to create new asset reserves and check if a reserve is empty.

3. **Integration Developers**:
   - Developers integrating this code into larger systems or applications need to understand and utilize the functionality provided by the `AssetReserve` struct.
   - They may use these reserves in conjunction with other components of their system, such as liquidity pools or trading algorithms.
   - Integration developers ensure that asset reserves are correctly managed and utilized within the broader context of their application.

Overall, the `AssetReserve` struct and associated functionality serve to manage asset reserves efficiently within a system, providing a foundational component for various financial operations and calculations.

#### EMA Oracle

1. **lib.rs**:

   - **Source:** The source of the data. E.g. xyk pallet.
   - **Asset Pair:** The pair of assets for which the oracle is being calculated. E.g. HDX/DOT.
   - **Period:** The period over which the oracle is averaged. E.g. 10 minutes.
   - **Oracle:** The oracle entry for the given source, asset pair, and period. Contains the price, volume, liquidity, and updated_at timestamp.
   - **Accumulator:** A temporary storage for oracle data that is aggregated during the block and updated at the end of the block.
   - **OnActivityHandler:** A callback handler for trading and liquidity activity that schedules oracle updates.

2. **types.rs**:

- **Oracle:** The oracle is responsible for providing the price data for the asset.
- **Consumer:** The consumer is responsible for using the price data provided by the oracle.

#### Ema Oracle Math

1. **math.rs**:

   1. **EMA Calculation Functions**:

      - These functions are responsible for calculating exponential moving averages (EMAs) of prices, volumes, and liquidity.
      - Functions like `iterated_price_ema`, `iterated_balance_ema`, `iterated_volume_ema`, and `iterated_liquidity_ema` calculate EMAs based on previous values, incoming values, and smoothing factors.

   2. **Weighted Average Calculation Functions**:

      - Functions like `price_weighted_average`, `balance_weighted_average`, `volume_weighted_average`, and `liquidity_weighted_average` compute weighted averages for prices, balances, volumes, and liquidity.
      - These functions are used in EMA calculations to determine the influence of incoming data on the overall average.

   3. **Utility Functions**:

      - Functions like `saturating_sub`, `multiply`, `round`, `round_to_rational`, `rounding_add`, and `rounding_sub` are utility functions used in precision handling, arithmetic operations, and rounding during EMA calculations.

   4. **Constants and Types**:
      - Definitions for types like `EmaPrice`, `EmaVolume`, and `EmaLiquidity` are provided.
      - Constants like `Fraction::ONE` and `Fraction::ZERO` are used in calculations.

#### Circuit breaker

1. **lib.rs**:

   1. **Technical Origin**: This role has permission to change the trade volume limit of an asset. It is specified in the `Config` trait as `type TechnicalOrigin: EnsureOrigin<Self::RuntimeOrigin>;`. Functions that require this role include:

      - `set_trade_volume_limit`

   2. **Whitelisted Accounts**: These accounts bypass checks for adding/removing liquidity. The root account is always whitelisted. Functions that check for whitelisted accounts include:

   - `ensure_add_liquidity_limit`
   - `ensure_remove_liquidity_limit`

   3. **Root**: The root account always has special privileges and is considered whitelisted by default.

### Invariants Generated

#### Function Level

1.  **Omnipool**

    - **Notation**

      | Variable   | Description                          |
      | ---------- | ------------------------------------ |
      | $R_i$      | Omnipool reserves of asset $i$       |
      | $Q_i$      | LRNA in subpool for asset $i$        |
      | $S_i$      | Shares in asset $i$ subpool          |
      | $\omega_i$ | Weight cap for asset $i$ in Omnipool |

      For a given state variable $X$, we will generally denote the value of $X$ _after_ the operation by $X^+$.

      [Omnipool Specification](https://github.com/galacticcouncil/HydraDX-simulations/blob/main/hydradx/spec/OmnipoolSpec.ipynb)

    - **Function-level Invariants**

      - **Swap**

      - For all assets $i$ in Omnipool, the invariant $R_i Q_i$ should not decrease due to a swap. This means that after a swap for all assets $i$ in Omnipool:

      $$
      R_i^+ Q_i^+ \geq R_i Q_i
      $$

      - $R_iQ_i$ should be invariant, but one is calculated from the other. If e.g. $R_i^+$ is calculated it may have error up to $1$, in which case the product $R_i^+Q_i^+$ may have error up to $Q_i^+$. If $Q_i^+$ is calculated, then the product has error up to $R_i^+$. Thus we should always be able to bound the error by $max(R_i^+,Q_i^+)$, giving us

      $$
      R_i Q_i + max(R_i^+, Q_i^+) \geq R_i^+ Q_i^+
      $$

      - **Add liquidity**

      - Add liquidity should respect price $\frac{Q_i}{R_i}$. This means $\frac{Q_i}{R_i} = \frac{Q_i^+}{R_i^+}$, or $Q_i R_i^+ = Q_i^+ R_i$. What is most important here is not which direction we round but the accuracy. So we must test that

      $$
      (Q_i^+ - 1)R_i \leq Q_i R_i^+ \leq (Q_i^+ + 1)R_i
      $$

      - Adding liquidity in asset $i$ should keep the ratio of assets per shares constant. We round so as to not decrease the assets per share of asset $i$, $\frac{R_i}{S_i}$; that is, we favor the other LPs over the LP currently adding liquidity, to avoid any potential exploit. This means, $\frac{R_i^+}{S_i^+}\geq \frac{R_i}{S_i}$, so

      $$
      R_i (S_i^+ + 1) \geq R_i^+ S_i \geq R_i S_i^+
      $$

      - Adding liquidity needs to respect weight caps. That is,

      $$
      \omega_iQ^+ \geq Q_i^+
      $$

      - **Withdraw liquidity**

      - Withdraw liquidity should respect price $\frac{Q_i}{R_i}$. This means $\frac{Q_i}{R_i} = \frac{Q_i^+}{R_i^+}$, or $Q_i R_i^+ = Q_i^+ R_i$. Allowing for rounding error, we must check

      $$
      (Q_i^+ - 1)R_i \leq Q_i R_i^+ \leq (Q_i^+ + 1)R_i
      $$

      - Withdraw liquidity in asset $i$ should keep the ratio of assets per shares constant. We round so as to not decrease the assets per share of asset $i$, $\frac{R_i}{S_i}$; that is, we favor the other LPs over the LP currently withdrawing liquidity, to avoid any potential exploit. This means $\frac{R_i^+}{S_i^+}\geq \frac{R_i}{S_i}$, so

      $$
      R_i (S_i^+ + 1) \geq R_i^+ S_i \geq R_i S_i^+
      $$

2.  **Stableswap**

    - **Notation**

    | Variable | Description                     |
    | -------- | ------------------------------- |
    | $R_i$    | Stableswap reserve of asset $i$ |
    | $S$      | Shares in stableswap pool       |
    | $D$      | Stableswap invariant            |

    For a given state variable $X$, we will generally denote the value of $X$ _after_ the operation by $X^+$.

    - **Operation-specific**

      - **Swap**

      - After a swap, we should have

      $$
      D + 5000 \geq D^+ \geq D
      $$

      - **Add Liquidity (arbitrary assets)**

      - After adding liquidity, we should have

      $$
      D^+ \geq D
      $$

      $$
      D^+ S \geq D S^+ \geq (D^+ - 1)S
      $$

      - **Withdraw Liquidity (one asset)**

      - After removing liquidity, we should have

      $$
      D^+\leq D
      $$

      $$
      D(S^+ + 1) \geq D^+ S \geq D S^+
      $$

#### System level

1. **Omnipool**

- **Notation**

| Variable     | Description                             |
| ------------ | --------------------------------------- |
| $Q$          | Total LRNA in Omnipool                  |
| $Q_i$        | LRNA in subpool for asset $i$           |
| $S_i$        | Shares in asset $i$ subpool             |
| $B_i$        | Protocol shares in asset $i$            |
| $s_i^\alpha$ | Shares in asset $i$ held by LP $\alpha$ |

For a given state variable $X$, we will generally denote the value of $X$ _after_ the operation by $X^+$.

[Omnipool Specification](https://github.com/galacticcouncil/HydraDX-simulations/blob/main/hydradx/spec/OmnipoolSpec.ipynb)

- Total shares issued should equal the sum of shares held in all LP positions and protocol.

$$
S_i = \sum_{\alpha}s_i^\alpha + B_i
$$

- The total LRNA in Omnipool should equal the sum of LRNA held by pools.

$$
Q = \sum_i Q_i
$$

## Approach Taken-in Evaluating HydraDX Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the `HydraDX`.

    I start with the following contracts, which play crucial roles in the `HydraDX`:

    **Main Contracts I Looked At**

    I start with the following contracts, which play crucial roles in the HydraDX:

                omnipool/src/lib.rs
                omnipool/src/types.rs
                omnipool/src/traits.rs
                src/omnipool/math.rs
                src/omnipool/types.rs
                stableswap/src/lib.rs
                stableswap/src/types.rs
                src/stableswap/math.rs
                src/stableswap/types.rs
                ema-oracle/src/lib.rs
                ema-oracle/src/types.rs
                src/ema/math.rs
                circuit-breaker/src/lib.rs

    I started my analysis by examining the intricate structure and functionalities of the hydrax protocol, particularly focusing on its various modules such as Omnipool, Omnipool Math, and Stableswap. These modules offer a comprehensive suite of features for managing liquidity, trading assets, and maintaining stablecoin pools within the ecosystem.In Omnipool, the protocol enables decentralized trading through an Automated Market Maker (AMM) model, facilitating asset swaps without intermediaries. Key functions such as asset management, position management, and trade processing are provided to ensure efficient operation of the liquidity pool. Additionally, the protocol defines types and traits to facilitate the management and interaction with the Omnipool. The Omnipool Math module offers essential mathematical functions for calculating changes in asset states during various liquidity-related operations. Functions for selling, buying, adding liquidity, removing liquidity, as well as calculating total value locked (TVL) and cap differences, are meticulously implemented to ensure accurate state transitions within the pool.On the other hand, the Stableswap module introduces a Curve-style stablecoin AMM, allowing the creation and management of liquidity pools for stablecoins. With features like pool creation, liquidity addition, removal, and trading, the Stableswap module provides a robust framework for maintaining stablecoin liquidity pools with adjustable parameters such as amplification and trade fees.

2.  **Documentation Review**:

    Then went to Review [this docs](https://docs.hydradx.io/) for a more detailed and technical explanation of the HydraDX project.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Codebase Quality

Overall, I consider the quality of the HydraDX protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Architecture & Design**                | The protocol features a modular design, segregating functionality into distinct contracts (e.g., Omnipool, Stableswap, Oracle) for clarity and ease of maintenance. The use of libraries like Stableswap Math for mathematical operations also indicates thoughtful design choices aimed at optimizing contract performance and gas efficiency.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **Upgradeability & Flexibility**         | The project does not explicitly implement upgradeability patterns (e.g., proxy contracts), which might impact long-term maintainability. Considering an upgrade path or versioning strategy could enhance the project's flexibility in addressing future requirements..                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **Community Governance & Participation** | The protocol incorporates mechanisms for community governance, enabling token holders to influence decisions. This fosters a decentralized and participatory ecosystem, aligning with the broader ethos of blockchain development.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| **Error Handling & Input Validation**    | Functions check for conditions and validate inputs to prevent invalid operations, though the depth of validation (e.g., for edge cases transactions) would benefit from closer examination.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Code Maintainability and Reliability** | The provided contracts are well-structured, exhibiting a solid foundation for maintainability and reliability. Each contract serves a specific purpose within the ecosystem, following established patterns and standards. This adherence to best practices and standards ensures that the code is not only secure but also future-proof. The usage of contracts for implementing token and security features like access control further underscores the commitment to code quality and reliability. However, the centralized control present in the form of admin and owner privileges could pose risks to decentralization and trust in the long term. Implementing decentralized governance or considering upgradeability through proxy contracts could mitigate these risks and enhance overall reliability. |
| **Code Comments**                        | The contracts are accompanied by comprehensive comments, facilitating an understanding of the functional logic and critical operations within the code. Functions are described purposefully, and complex sections are elucidated with comments to guide readers through the logic. Despite this, certain areas, particularly those involving intricate mechanics or tokenomics, could benefit from even more detailed commentary to ensure clarity and ease of understanding for developers new to the project or those auditing the code.                                                                                                                                                                                                                                                                       |
| **Testing**                              | The contracts exhibit a commendable level of test coverage, approaching nearly 100%, which is indicative of a robust testing regime. This coverage ensures that a wide array of functionalities and edge cases are tested, contributing to the reliability and security of the code. However, to further enhance the testing framework, the incorporation of fuzz testing and invariant testing is recommended. These testing methodologies can uncover deeper, systemic issues by simulating extreme conditions and verifying the invariants of the contract logic, thereby fortifying the codebase against unforeseen vulnerabilities.                                                                                                                                                                          |
| **Code Structure and Formatting**        | The codebase benefits from a consistent structure and formatting, adhering to the stylistic conventions and best practices of Solidity programming. Logical grouping of functions and adherence to naming conventions contribute significantly to the readability and navigability of the code. While the current structure supports clarity, further modularization and separation of concerns could be achieved by breaking down complex contracts into smaller, more focused components. This approach would not only simplify individual contract logic but also facilitate easier updates and maintenance.                                                                                                                                                                                                   |
| **Strengths**                            | Among the notable strengths of the codebase are its adherence to innovative integration of blockchain technology with dex's and stableswaps. The utilization of stableswap libraries for security and standard compliance emphasizes a commitment to code safety and interoperability. The creative use of circuit-breaker/src/lib.rs and src/ema/math.rs in the Dex's mechanics demonstrates.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **Documentation**                        | The contracts themselves contain comments and some descriptions of functionality, which aids in understanding the immediate logic. It was learned that the project also provides external documentation. However, it has been mentioned that this documentation is somewhat outdated. For a project of this complexity and scope, keeping the documentation up-to-date is crucial for developer onboarding, security audits, and community engagement. Addressing the discrepancies between the current codebase and the documentation will be essential for ensuring that all stakeholders have a clear and accurate understanding of the system's architecture and functionalities.                                                                                                                             |

## Architecture

### **System Workflow**

**Omnipool**

1. **Asset Management**:

   - Assets are added to the Omnipool, each with its own state (tradability, reserve, etc.).
   - Users can buy, sell, add liquidity, or remove liquidity for any supported asset.

2. **Position Management**:

   - Liquidity providers create positions by adding liquidity to the Omnipool.
   - Positions represent the amount of liquidity provided and the LP shares owned.

3. **Trade Execution**:
   - Trades are executed between assets in the Omnipool based on the constant product formula.
   - Trades incur fees, which are distributed to the protocol and liquidity providers.

**Stableswap**

1. **Pool Creation**:

   - Pools are created with a set of stablecoins and an amplification parameter.
   - The amplification parameter determines the shape of the constant product curve for the pool.

2. **Liquidity Management**:

   - Liquidity providers can add or remove liquidity from pools.
   - Liquidity changes are calculated using mathematical formulas to maintain the pool's stability.

3. **Trading**:
   - Traders can buy or sell stablecoins within a pool.
   - Trades are executed based on the constant product formula, ensuring that the price of each stablecoin remains relatively stable.

**EMA Oracle**

1. **Price Tracking**:

   - The EMA oracle tracks the price, volume, and liquidity of assets traded on the HydraDX.
   - This data is used to provide accurate and up-to-date information to traders and other users.

2. **Exponential Moving Average (EMA)**:
   - The EMA oracle uses an EMA to smooth out price fluctuations and provide a more stable representation of asset values.
   - The EMA is calculated based on historical data and a smoothing factor.

**Circuit Breaker**

1. **Trade Volume and Liquidity Limits**:

   - Circuit breakers are implemented to prevent excessive trading volume or liquidity changes in a short period.
   - These limits help maintain the stability and liquidity of the HydraDX markets.

2. **Limit Enforcement**:
   - The circuit breaker pallet ensures that trade volume and liquidity limits are not exceeded.
   - If a limit is reached, trading or liquidity changes may be restricted until the limit resets.

**Overall Workflow**

The HydraDX protocol combines these components to provide a comprehensive and flexible trading platform for digital assets. Users can trade assets, provide liquidity, and access accurate price information through the EMA oracle. The circuit breaker mechanism helps ensure the stability and liquidity of the markets, while the Omnipool and Stableswap modules provide efficient and scalable trading mechanisms for both volatile and stable assets.

| File Name                  | Core Functionality                                                                                                                                                                                                                                                                                                           | Technical Characteristics                                                                                                                                                                                                                                                                                                                 | Importance and Management                                                                                                                                                                                                                                                                                              |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| omnipool/src/lib.rs        | The core functionality of this contract is to provide a decentralized exchange (DEX) using an Automated Market Maker (AMM) model, allowing users to trade assets without intermediaries by pooling liquidity and utilizing on-chain math functions for state calculations.                                                   | Technical characteristics include asset management functionalities, position management represented as NFTs, and trade execution with parameters like price barriers and dynamic fees, all implemented through precise updates to pool reserves and asset imbalances using hooks and events.                                              | Importance and management in this contract involve enabling non-custodial trading with low fees, thereby promoting greater decentralization and accessibility compared to order-book based alternatives like HydraDX.                                                                                                  |
| omnipool/src/types.rs      | The core functionality of this contract is to define types and structures essential for managing an Omnipool, including representations of asset balances, prices, tradability, positions, imbalances, and asset reserve states.                                                                                             | Technical characteristics encompass features like using bitflags for asset tradability, providing types for representing imbalances and asset reserve states, and offering functions for converting between different representations of assets and positions.                                                                            | Importance and management in this contract involve facilitating precise tracking and management of asset states and positions within the Omnipool, thereby enabling efficient liquidity provision and trading operations while ensuring consistency and accuracy in state transitions.                                 |
| omnipool/src/traits.rs     | The core functionality of this contract is to define traits, structs, and implementations facilitating the management of an Omnipool, including hooks for liquidity changes and trades, external price fetching, and enforcing price constraints.                                                                            | Technical characteristics include the definition of traits such as OmnipoolHooks, ExternalPriceProvider, and ShouldAllow, along with types like AssetInfo and EnsurePriceWithin, which collectively enable extensibility, external integration, and validation mechanisms within the Omnipool ecosystem.                                  | Importance and management in this contract involve providing a flexible framework for customizing Omnipool behavior, integrating external price data, and enforcing price constraints, thus ensuring the integrity and efficiency of liquidity management and trading operations within the ecosystem.                 |
| src/omnipool/math.rs       | The core functionality of this contract is to implement mathematical functions for calculating delta changes in the state of an asset pool during liquidity-related operations like selling, buying, adding liquidity, removing liquidity, and determining metrics such as total value locked (TVL) and cap differences.     | Technical characteristics include the provision of precise mathematical calculations for various liquidity operations, including selling, buying, adding, and removing liquidity, along with functionalities for determining TVL and cap differences, ensuring accuracy and efficiency in managing asset pools.                           | Importance and management in this contract involve enabling accurate and efficient management of liquidity operations within the asset pool, facilitating informed decision-making regarding TVL and cap differences, thereby enhancing the stability and functionality of the liquidity system.                       |
| src/omnipool/types.rs      | The core functionality of this contract involves defining structures and implementations for managing asset reserves, liquidity pools, and trading mechanisms, facilitating operations such as updating asset states, calculating prices, and handling balance adjustments.                                                  | Technical characteristics include the provision of structured data types and functions to efficiently represent and manipulate asset states, balance updates, trade fees, and liquidity changes within the Omnipool, ensuring accuracy and reliability in managing liquidity operations.                                                  | Importance and management in this contract entail enabling precise tracking and management of asset reserves, liquidity states, and trading activities, thereby enhancing the stability, efficiency, and functionality of the Omnipool ecosystem.                                                                      |
| stableswap/src/lib.rs      | The core functionality of this contract is to enable the creation and management of Curve-style stablecoin automated market maker (AMM) pools with up to 5 assets, featuring a pricing formula based on amplification and facilitating liquidity provision, removal, and asset trading.                                      | Technical characteristics include the implementation of a stableswap pallet within the HydraDX runtime, utilizing a constant product formula for price calculation and providing roles such as AuthorityOrigin, LiquidityProviderOrigin, and TraderOrigin for managing pool creation, liquidity addition/removal, and trading operations. | Importance and management in this contract involve facilitating stablecoin trading with minimized volatility, enabling efficient liquidity provision and removal, and ensuring fair and transparent trading mechanisms, ultimately enhancing the stability and usability of the ecosystem for everyday transactions.   |
| stableswap/src/types.rs    | The core functionality of this contract is to define data structures and traits for managing stable pools, including pool properties, asset amounts, tradability flags, and interfaces for oracle interaction and weight calculation.                                                                                        | Technical characteristics include the representation of pool information through the PoolInfo struct, asset amounts with the AssetAmount struct, tradability flags with the Tradability bitmask, and pool state tracking with the PoolState struct, alongside the StableswapHooks trait for oracle interaction and weight calculation.    | Importance and management in this contract revolve around enabling the creation and management of stable pools, ensuring efficient tracking of pool state and asset tradability, and providing extensible interfaces for oracle integration and weight calculation to support stable trading operations effectively.   |
| src/stableswap/math.rs     | The core functionality of this contract involves implementing mathematical functions for a stableswap pool, facilitating automated market making with multiple assets, including liquidity provision, asset trading with fees, and share distribution to liquidity providers, ensuring stable trading ratios between assets. | Technical characteristics include formulas for calculating the D invariant and reserve values (Y), alongside functions for handling share minting, precision normalization, and trade execution using mathematical operations and iterative algorithms.                                                                                   | Importance and management in this contract revolve around maintaining accurate and stable trading within the automated market making system, enabling efficient liquidity provision, asset trading, and fair share distribution among liquidity providers, crucial for the effective operation of the stableswap pool. |
| src/stableswap/types.rs    | The core functionality of this contract involves implementing the StableSwap algorithm for calculating token amounts in liquidity pools, maintaining constant reserve ratios, and facilitating token swaps.                                                                                                                  | Technical characteristics include mathematical functions like calculate_out_given_in, calculate_in_given_out, calculate_shares, and others, ensuring accurate calculation of token amounts, shares, and reserves in liquidity pools.                                                                                                      | Importance and management in this contract lie in providing efficient and stable token swaps within liquidity pools, crucial for decentralized exchanges and other DeFi applications, enabling effective liquidity provision, trading, and asset management.                                                           |
| ema-oracle/src/lib.rs      | The core functionality of this contract involves implementing an Exponential Moving Average (EMA) oracle for tracking asset price, volume, and liquidity over time in the HydraDX protocol.                                                                                                                                  | Technical characteristics include functions such as on_trade and on_liquidity_changed for updating the oracle with trade and liquidity data, as well as methods like get_entry and get_price for retrieving EMA values and asset prices.                                                                                                  | Importance and management in this contract revolve around providing accurate and up-to-date data on asset metrics, crucial for efficient price discovery, liquidity provision, and trading strategies within the HydraDX protocol, managed by the oracle's functionalities.                                            |
| ema-oracle/src/types.rs    | The core functionality of this contract involves managing Exponential Moving Average (EMA) oracles for each asset pair and period in the HydraDX protocol, updating them with trade and liquidity changes.                                                                                                                   | Technical characteristics include OracleEntry struct for storing oracle data, and functions like calculate_new_by_integrating_incoming and update_to_new_by_integrating_incoming for calculating and updating oracle entries.                                                                                                             | Importance and management lie in providing accurate EMA data for asset pairs and periods, crucial for price tracking, liquidity monitoring, and informed decision-making within the HydraDX protocol, managed through oracle updates and calculations.                                                                 |
| src/ema/math.rs            | The core functionality of this contract lies in providing functions for calculating exponential moving averages (EMAs) and performing weighted averages for oracle values in the HydraDX protocol.                                                                                                                           | Technical characteristics include the calculation of EMAs and weighted averages using specified smoothing factors, as well as functions for updating outdated values and determining smoothing factors based on periods.                                                                                                                  | Importance and management revolve around accurately tracking oracle values, particularly prices, balances, volumes, and liquidity, which are crucial for informed decision-making within the HydraDX protocol, managed through the calculation and updating of EMAs and weighted averages.                             |
| circuit-breaker/src/lib.rs | The core functionality of this contract lies in managing circuit breakers for trade volume and liquidity limits in the HydraDX protocol.                                                                                                                                                                                     | Technical characteristics include the definition of a Config trait specifying runtime requirements and the implementation of functions to initialize and enforce trade and liquidity limits, as well as account whitelisting.                                                                                                             | Importance and management revolve around maintaining the stability and security of the HydraDX protocol by preventing excessive trading and liquidity operations through circuit breakers, which are configurable and enforceable via the pallet's functions and methods.                                              |

## Systemic Risks, Centralization Risks, Technical Risks & Integration Risks

Here's an analysis of potential systemic, centralization, Technical and Integration risks in the contracts:

#### Omnipool

1. **lib.rs**:

2. **types.rs**

   - **Systemic Risks:**

     1. **Asset Reserve State Mutation**: Changes to the asset reserve state may impact the stability and functioning of the Omnipool, potentially leading to systemic risks if not managed properly.
     2. **Market Liquidity Fluctuations**: Fluctuations in market liquidity can affect the overall performance and stability of the Omnipool system, posing systemic risks to users and investors.
     3. **Operational risk**: The contract could be subject to downtime or other operational issues that could prevent traders from accessing their funds or executing trades.

   - **Centralization Risks:**

     1. **Protocol-Owned Asset Shares**: The presence of protocol-owned asset shares may introduce centralization risks, potentially giving the protocol undue influence over market dynamics and user interactions.
     2. **Hub Asset Control**: Centralized control over the hub asset reserves may lead to centralization risks, affecting the autonomy and decentralization of the Omnipool ecosystem.

   - **Technical Risks:**

     1. **Code complexity**: The contract code is complex and could be difficult to understand and maintain.
     2. **Mathematical Calculations**: The reliance on complex mathematical calculations for asset pricing and state updates introduces technical risks related to computational accuracy and efficiency.
     3. **Data Integrity**: Risks associated with data integrity and accuracy in asset state representation, which may impact the reliability and trustworthiness of the Omnipool system.

   - **Integration Risks:**

     1. **Runtime Environment Dependencies**: Dependencies on specific runtime environments and configurations may pose integration risks, potentially leading to compatibility issues with different blockchain frameworks or versions.
     2. **Third-Party Module Integration**: Risks associated with integrating third-party modules or dependencies into the Omnipool system, including version compatibility, security vulnerabilities, and maintenance challenges.
     3. **Exchange integration**: The contract may not be listed on all exchanges, which could limit the liquidity available to traders.

3. **traits.rs**

   - **Systemic Risks**:

     1. **External Price Provider Failure**: Reliance on external price providers for asset prices may introduce systemic risks if these providers experience downtime or provide inaccurate data, leading to potential disruptions in pricing mechanisms and trade execution.
     2. **Price Discrepancies**: Discrepancies between the spot price and prices provided by external oracles may result in systemic risks, impacting the fairness and efficiency of trading within the Omnipool system.

   - **Centralization Risks**:

     1. **Whitelisted Accounts Influence**: The presence of whitelisted accounts that bypass price checks may introduce centralization risks, potentially allowing certain entities to manipulate asset prices and trading activities within the Omnipool.
     2. **Dependency on External Oracles**: Reliance on external oracle data for price comparisons introduces centralization risks, as the accuracy and reliability of asset prices are contingent on the performance and integrity of these oracles.
     3. **Price manipulation**: The contract could be used to manipulate the price of assets by creating artificial demand or supply.

   - **Technical Risks**:

     1. **Data Integrity**: Risks associated with data integrity and accuracy in asset price comparisons, including potential vulnerabilities to data manipulation or tampering that may compromise the integrity of trading operations within the Omnipool.
     2. **Numerical Stability**: Risks related to numerical stability and precision in price calculations, particularly when performing arithmetic operations on fixed-point numbers, which may result in computational errors or inaccuracies.

   - **Integration Risks**:

     1. **External Oracle Integration**: Risks associated with integrating external price providers into the Omnipool system, including challenges related to compatibility, reliability, and maintenance of these external services, which may impact the overall performance and functionality of the system.
     2. **Whitelisted Account Management**: Risks associated with managing whitelisted accounts within the Omnipool system, including potential complexities in account verification, authorization, and access control mechanisms, which may introduce vulnerabilities or operational inefficiencies.

#### Omnipool Math

1. **math.rs**:

   - **Systemic Risks**:

     1. **Asset Price Calculation**: Incorrect calculation of asset prices may lead to systemic risks, affecting the accuracy of trade executions and liquidity provision within the system.
     2. **Withdrawal Fee Calculation**: Inaccurate calculation of withdrawal fees based on spot prices and oracle prices may introduce systemic risks, impacting the fairness and efficiency of liquidity withdrawals.

   - **Centralization Risks**:

     1. **Reliance on External Data**: Dependency on external data sources, such as spot prices and oracle prices, for fee calculations and liquidity adjustments introduces centralization risks, as the integrity and reliability of these sources can affect the overall performance of the system.
     2. **Imbalance Calculation**: Centralization risks arise from the calculation of imbalances in liquidity provision, as discrepancies in determining the appropriate imbalance may impact the stability and fairness of the system.

   - **Technical Risks**:

     1. **Numerical Precision**: Risks associated with numerical precision in arithmetic operations, particularly when handling fixed-point numbers and calculating fees, may lead to technical challenges such as overflow or underflow errors, potentially compromising the accuracy of financial calculations.
     2. **Data Integrity**: Risks related to data integrity and accuracy in asset reserve states and position calculations, including potential vulnerabilities to data manipulation or tampering that may undermine the reliability of liquidity adjustments and fee calculations.

   - **Integration Risks**:

     1. **External Price Integration**: Risks associated with integrating external price data providers for asset price calculations and fee determinations, including challenges related to compatibility, reliability, and security of data transmission, may impact the overall functionality and performance of the system.
     2. **Liquidity Adjustment Integration**: Risks arise from integrating liquidity adjustment mechanisms, such as imbalance calculations and position adjustments, into the system, including complexities in implementation, maintenance, and validation that may affect the stability and robustness of liquidity management.

2. **types.rs**:

   - **Systemic Risks**:

     1. **Unchecked Overflow**: There is a risk of arithmetic overflow in operations involving balance updates (`BalanceUpdate`) if the balances exceed their maximum representable value. This can lead to unexpected behavior or loss of funds if not handled properly.

   - **Centralization Risks**:

     1. **Protocol Controlled Shares**: The presence of `protocol_shares` in `AssetReserveState` indicates a degree of control exerted by the protocol over the LP shares for an asset. Depending on how these shares are managed and utilized, there could be centralization risks if the protocol wields disproportionate power.

   - **Technical Risks**:

     1. **Unchecked Arithmetic Operations**: The use of unchecked arithmetic operations (`CheckedAdd`, `CheckedSub`) in the implementation of balance updates (`BalanceUpdate`) introduces technical risks. While these operations aim to prevent arithmetic overflow or underflow, there is still a risk of unexpected behavior if the checks fail or if the checks are not comprehensive.

   - **Integration Risks**:

     1. **Compatibility Issues**: The integration of this contract with other systems or modules may pose risks related to compatibility, especially if different parts of the system handle balances differently or rely on different balance representations. Ensuring seamless integration and interoperability with other components of the system is crucial to mitigate these risks.

#### Stableswap

1. **lib.rs**:

   - **Systemic Risks**:

     1. **Maximum Assets in Pool**: The contract has a maximum limit for the number of assets allowed in a pool (`MAX_ASSETS_IN_POOL`). Exceeding this limit could potentially cause systemic issues or unexpected behavior in the pool management logic.

   - **Centralization Risks**:

     1. **AuthorityOrigin**: The `create_pool` function requires an origin that must be `T::AuthorityOrigin`, indicating a centralized authority responsible for creating pools. This centralization could lead to dependency risks if the authority is compromised or misuses its power.

   - **Technical Risks**:

     1. **Amplification Range**: The contract specifies an amplification range (`AmplificationRange`), and the `update_amplification` function allows modifying the pool's amplification within this range. However, incorrect manipulation of amplification could introduce technical risks such as impermanent loss or instability in the pool's behavior.

   - **Integration Risks**:

     1. **Asset Registry**: The contract relies on an asset registry (`T::AssetInspection`) to check if assets are correctly registered and retrieve asset decimals. Integration with this external registry introduces the risk of data inconsistency or reliance on external systems, which could impact the contract's functionality if the registry is unavailable or inaccurate.

2. **types.rs**:

   - **Systemic Risks**:

     1. **Maximum Assets in Pool**: The contract specifies a maximum number of assets allowed in a pool (`MAX_ASSETS_IN_POOL`). Exceeding this limit could lead to systemic issues or unexpected behavior in the pool management logic.

   - **Centralization Risks**:

     1. **Authority Over Pool Creation**: The contract does not include explicit mechanisms for decentralized pool creation. The authority to create pools is not distributed among users or governed by a decentralized mechanism, potentially leading to centralization risks if the central authority misuses its power or becomes compromised.

   - **Technical Risks**:

     1. **Asset Uniqueness Check**: The contract includes a function `has_unique_elements` to check for unique elements in a collection of assets. However, this function relies on the correct implementation of iterators and could introduce technical risks if not implemented correctly, potentially leading to incorrect pool configurations or unexpected behavior.

   - **Integration Risks**:

     1. **Asset Decimals Retrieval**: The contract interacts with an external asset registry (`Pallet::<T>::retrieve_decimals`) to retrieve asset decimals. Integration with this external registry introduces the risk of data inconsistency or reliance on external systems, which could impact the contract's functionality if the registry is unavailable or inaccurate.

#### Stableswap Math

1. **math.rs**:

   - **Systemic Risks**:

     1. **Convergence Issues**: The contract relies on iterative methods such as Newton's formula for convergence, which might not converge properly if the number of iterations (`D` and `Y`) is insufficient. This lack of convergence can lead to incorrect calculations and potential loss of funds.

   - **Centralization Risks**:

     1. **Amplification Parameter Adjustment**: The `calculate_amplification` function adjusts the amplification parameter based on block numbers. Depending on how this adjustment is governed, it could introduce centralization risks if controlled by a small number of entities or subject to manipulation.

   - **Technical Risks**:

     1. **Numerical Precision**: The contract involves numerous calculations with fixed-point arithmetic and conversions between different numeric representations. Any miscalculations or inaccuracies in these operations could result in incorrect financial outcomes or vulnerabilities to attacks.
     2. **Iteration Limits**: The contract imposes limits on the number of iterations for certain iterative calculations (`MAX_Y_ITERATIONS` and `MAX_D_ITERATIONS`). If these limits are set too low, it may lead to premature termination of calculations, potentially resulting in inaccurate results or failed transactions.
     3. **Overflow/Underflow**: There are several arithmetic operations throughout the contract (`checked_add`, `checked_sub`, etc.) aimed at preventing overflow or underflow. However, if these checks are inadequate or incorrectly implemented, they could introduce vulnerabilities to arithmetic errors.
     4. **Input Validation**: The contract assumes valid input parameters in functions such as `calculate_out_given_in`, `calculate_in_given_out`, etc. Insufficient input validation could lead to unexpected behavior or vulnerabilities such as denial-of-service attacks or manipulation of calculations.

   - **Integration Risks**:

     1. **External Dependencies**: The contract relies on external crates and libraries (`num_traits`, `primitive_types`, `sp_arithmetic`, `sp_std`, etc.). Any vulnerabilities or changes in these dependencies could impact the security and functionality of the contract.
     2. **Interoperability**: If this contract interacts with other contracts or systems, there could be integration risks associated with data consistency, protocol compatibility, and security vulnerabilities in the interaction mechanisms.

2. **types.rs**:

   - **Systemic Risks**:

     1. **Data Loss Risk**: While the contract defines a `is_zero` function to check if the reserve amount is zero, there are no measures in place to prevent or handle data loss or corruption. If reserve amounts are inadvertently set to zero or corrupted, it could lead to unexpected behavior or loss of funds.

   - **Centralization Risks**:

     1. **Control over Reserves**: Depending on how the reserve amounts are managed and updated, there could be centralization risks if a small number of entities or controllers have the authority to modify these reserves. Centralized control over reserves could lead to manipulation or misuse, impacting the fairness and integrity of the system.

   - **Technical Risks**:

     1. **Data Integrity**: There are no explicit checks or validations to ensure the integrity of reserve amounts or decimals. If reserve amounts are manipulated or set incorrectly, it could lead to incorrect calculations or financial losses.
     2. **Data Conversion**: The contract provides conversion functions (`From`) to convert `AssetReserve` instances into `u128` values. However, there are no checks or safeguards to ensure the validity of these conversions, potentially leading to unexpected behavior or errors if used improperly.
     3. **Zero Balance Handling**: While the `is_zero` function checks if the reserve amount is zero, there are no explicit error-handling mechanisms if zero balance assets are encountered in calculations. This lack of error handling could lead to unexpected behavior or vulnerabilities in downstream processes or calculations.

   - **Integration Risks**:

     1. **Dependency Risks**: The contract relies on external dependencies such as `num_traits`. Any vulnerabilities or changes in these dependencies could impact the security and functionality of the contract.
     2. **Interoperability**: If this contract is part of a larger system or interacts with other contracts or systems, there could be integration risks associated with data consistency, protocol compatibility, and security vulnerabilities in the interaction mechanisms.

#### EMA Oracle

1. **lib.rs**:

   - **Systemic Risks**:

     1. **Data Loss Risk**: There's a potential risk of data loss if the accumulator storage encounters issues during write operations. If data is not properly stored or overwritten, it could lead to inaccuracies or loss of historical information required for calculating oracle values accurately.

   - **Centralization Risks**:

     1. **Control over Oracles**: The pallet seems to centralize the aggregation of oracle data, as it accumulates data from various sources into a single accumulator. Depending on how this data aggregation is managed, there could be centralization risks if a small number of entities or controllers have the authority to influence oracle values, potentially leading to manipulation or inaccuracies.

   - **Technical Risks**:

     1. **Data Integrity**: The pallet relies on accurate data aggregation and calculation of oracle values. Any bugs or vulnerabilities in the data aggregation logic could lead to incorrect oracle values being reported, impacting the reliability and trustworthiness of the system.
     2. **Error Handling**: While error handling is implemented for certain scenarios, such as when liquidity amounts are zero, there might be other potential error scenarios that are not adequately handled, leading to unexpected behavior or vulnerabilities.
     3. **Dependency Risks**: The pallet relies on external dependencies such as `frame_support` and `sp_runtime`. Any vulnerabilities or changes in these dependencies could impact the security and functionality of the pallet.

   - **Integration Risks**:

     1. **Dependence on Other Pallets**: The pallet relies on data ingestion from other pallets, such as the xyk pallet, through provided callback handlers. Any changes or issues in the integration with these pallets could affect the functionality and accuracy of oracle values reported by this pallet.
     2. **Protocol Compatibility**: As the pallet interacts with other modules or systems, there's a risk of compatibility issues or protocol mismatches, especially if the integration requirements or protocols change over time.

2. **types.rs**:

   - **Systemic Risks**:

     1. **Dependency on External Systems**: The contract relies on external systems like `hydra_dx_math` and `hydradx_traits` for mathematical operations and trait implementations. Any failure or vulnerability in these dependencies could impact the contract's functionality.
     2. **Block Number Dependency**: The contract relies on block numbers for certain operations. Any disruption or inconsistency in block generation could affect the accuracy of data or trigger unexpected behavior.

   - **Centralization Risks**:

     1. **Single Source of Truth**: The contract appears to centralize price, volume, and liquidity data. Depending on a single oracle or source for this critical information could lead to manipulation or inaccuracies if the oracle or source is compromised.
     2. **Vendor Lock-in**: The contract's dependency on specific libraries (`hydra_dx_math` and `hydradx_traits`) could create a centralization risk if these libraries are controlled by a single entity or if there are limited alternatives available.

   - **Technical Risks**:

     1. **Algorithm Complexity**: The contract utilizes complex mathematical algorithms for calculating exponential moving averages and updating oracle entries. Complex algorithms increase the risk of implementation errors, which could lead to incorrect results or vulnerabilities.
     2. **Data Type Safety**: The contract uses custom data types (`Price`, `Volume`, `Liquidity`) that may require careful handling to ensure type safety and prevent overflow or underflow vulnerabilities.
     3. **External Call Dependence**: The contract may rely on external calls to retrieve data or perform calculations. Dependency on external calls introduces risks such as network congestion, oracle failures, or malicious data feeds.

   - **Integration Risks**:

     1. **Compatibility Issues**: Integrating this contract with other systems or smart contracts may pose compatibility challenges due to its reliance on specific libraries and data structures.
     2. **Versioning Concerns**: Changes to external dependencies or upgrades to the contract itself may introduce versioning conflicts or compatibility issues when integrating with existing systems.
     3. **Oracle Integration**: The contract's functionality heavily depends on oracle entries. Integrating with different oracles or upgrading the oracle system may require careful consideration and testing to ensure seamless integration and data consistency.

#### Ema Oracle Math

1. **math.rs**:
   Here's an analysis of the provided contract for systemic risks, centralization risks, technical risks, and integration risks:

   - **Systemic Risks**:

     1. **Precision Loss**: The contract performs arithmetic operations on rational numbers (`EmaPrice`, `EmaVolume`, `EmaLiquidity`) with limited precision. Precision loss during calculations could lead to inaccuracies in the oracle values, especially over multiple iterations or when dealing with large numbers.
     2. **Iteration Dependency**: Certain functions in the contract, such as `iterated_price_ema` and `iterated_balance_ema`, rely on the number of iterations (`u32`) to calculate exponential moving averages. Dependency on iteration count introduces risks if iterations are miscounted or inconsistent, leading to incorrect results.

   - **Centralization Risks**:

     1. **Dependency on External Libraries**: The contract depends on external libraries (`crate::fraction`, `crate::support`, `crate::transcendental`) for arithmetic operations and utility functions. Centralization risks arise if these libraries are controlled by a single entity or if there are limited alternatives available.
     2. **Single Source of Truth**: The contract centralizes oracle calculations for price, volume, and liquidity. Depending on a single source for critical calculations could lead to manipulation or inaccuracies if the source is compromised.

   - **Integration Risks**:

     1. **Arithmetic Overflows**: The contract performs arithmetic operations on large numbers (`U512`) and may be susceptible to arithmetic overflow or underflow vulnerabilities if not handled properly. Saturating operations are used to mitigate this risk, but thorough testing is required to ensure correctness.
     2. **Precision Handling**: The contract uses shifting and rounding techniques to handle precision reduction. Mishandling precision reduction could lead to incorrect results or unexpected behavior, especially in edge cases or under extreme conditions.
     3. **Complex Arithmetic Logic**: The contract contains complex arithmetic logic for weighted averaging and exponential moving average calculations. Complex logic increases the risk of implementation errors, making the contract harder to maintain and debug.

   - **Integration Risks**:

     1. **Compatibility Challenges**: Integrating this contract with other systems or smart contracts may pose compatibility challenges due to its reliance on specific external libraries and data structures. Ensuring compatibility and consistency across different environments may require additional effort and testing.
     2. **External Dependency Management**: The contract relies on external dependencies for arithmetic operations and utility functions. Managing these dependencies, including versioning and updates, could introduce integration risks if not handled properly.
     3. **Interoperability Concerns**: Interacting with this contract from other smart contracts or systems may require careful consideration of data types and precision handling to ensure seamless integration and data consistency.

#### Circuit breaker

1. **lib.rs**:

   - **Systemic Risks**:

     1. **Arithmetic Errors**: The contract performs arithmetic operations like addition, subtraction, multiplication, and division on balance types (`T::Balance`). Errors like overflow, underflow, and division by zero can lead to systemic risks if not handled properly, potentially disrupting the functioning of the entire system.

   - **Centralization Risks**:

     1. **Whitelisted Accounts**: Certain accounts specified in `type WhitelistedAccounts` bypass checks for adding/removing liquidity. Centralizing control over these accounts poses centralization risks as they can influence liquidity operations without standard checks, potentially favoring specific entities and impacting the fairness of the system.

   - **Integration Risks**:

     1. **Origin Permissions**: The contract relies on the `TechnicalOrigin` to ensure the origin of certain calls. Technical permissions are crucial for maintaining the integrity of the system, but incorrect or insufficient origin checks can lead to technical risks such as unauthorized access or manipulation of critical parameters.

   - **Integration Risks**:

     1. **External Contract Integration**: If this contract interacts with external contracts or systems, integration risks may arise. These risks include vulnerabilities in the external interfaces, dependencies on external systems' availability and correctness, and the potential for unforeseen interactions impacting the contract's behavior and security.

## Suggestions

### What could they have done better?

- 1.  If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;

[![test-cases.jpg](https://i.postimg.cc/1zgD5wCt/test-cases.jpg)](https://postimg.cc/v1s40gdF)

### What ideas can be incorporated?

1. **Integration of Stableswap and Omnipool**:

   - Explore opportunities to integrate the Stableswap AMM model with the Omnipool contract to provide users with additional trading functionalities, particularly for stablecoin trading pairs.
   - Implement cross-contract calls between Stableswap and Omnipool contracts to enable seamless trading experiences for users looking to swap between stablecoins and other assets.

2. **Dynamic Fee Adjustment Mechanism**:

   - Develop a dynamic fee adjustment mechanism that adjusts trading fees based on factors such as liquidity utilization, trading volume, and network congestion.
   - Implement governance controls to allow token holders to vote on proposed fee adjustments, promoting community engagement and decentralization.

3. **Advanced Risk Management Strategies**:

   - Introduce advanced risk management strategies such as impermanent loss protection mechanisms or dynamic capital allocation strategies to mitigate risks for liquidity providers.
   - Explore options for integrating with decentralized insurance protocols to provide additional risk coverage for liquidity providers.

4. **Enhanced Oracle Integration**:

   - Enhance the oracle integration to support multiple oracle providers and decentralized oracle networks, improving price accuracy and resilience against oracle failures or manipulation.
   - Implement price aggregation mechanisms to obtain reliable and accurate price feeds from multiple independent oracles.

5. **Security Audits and Formal Verification**:
   - Conduct comprehensive security audits and formal verification processes to identify and mitigate potential security vulnerabilities and smart contract bugs.
   - Engage with reputable auditing firms and security experts to perform code reviews and penetration testing, ensuring the contracts' robustness and resilience against attacks.

## Issues surfaced from Attack Ideas in [README](https://github.com/galacticcouncil/HydraDX-security/blob/main/threat_modelling.md)

### General

- Circumvent any of the mitigation mechanisms (eg through a price manipulation)

### Omnipool

- Sandwich attack on add/remove liquidity, since we do not have slippage limits on these transactions
- Attack exploiting assets with different decimal counts
- Price manipulation attacks
  - Price manipulation sandwiching add or remove liquidity
- Find the edges / limitations of our current attack prevention mechanisms (caps, withdrawal fees, trading fees)
- Large Omnipool LPs - extract value from other LPs by manipulating prices and withdrawing liquidity
- Attacks via XCM (cross-chain messaging) - eg fake minting on another parachain
- DDOS via fees

### Stableswap

- Attack on stableswap as A (amplification) changes
- Implications of having stablepool shares in the Omnipool - rounding, conversions, add/withdraw liquidity, IL from fees?
- Stableswap - manipulation via `withdraw_asset_amount` (buy / add liquidity), missing in Curve implementation
- Stableswap - manipulation via `add_liquidity_shares` (buy / add liquidity), missing in Curve implementation

### Oracles

- Correct oracle price and liquidity update via Omnipool and Stableswap hooks.
- Oracle price manipulation
  - What damage can be done? (withdrawal limits, DCA)

### Circuit breaker

- manipulating blocking add/remove liquidity
- manipulate trade volume limits

### LBP

- Attack on LBP taking advantage of exponent implementation




### Time spent:
90 hours