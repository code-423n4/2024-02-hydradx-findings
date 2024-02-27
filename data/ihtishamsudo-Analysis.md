# HydraDX Analysis Report

## Preface 

This report is a summary of the analysis of HydraDX Protocol's Codebase.
- This report is not an extension of the documents/Info provided by HydraDX Protocol.
- This report provides a high-level overview of the codebase and its security implications and some edge cases missed by developers and some suggestions to improve the codebase.
- This report aims to provide value to the sponsors and the developers of HydraDX Protocol.

## Brief Introduction & Overview Of HydraDX Protocol

HydraDX is a cutting-edge DeFi protocol, meticulously engineered to infuse a vast expanse of liquidity into the Polkadot ecosystem. The cornerstone of our approach is the HydraDX Omnipool, a groundbreaking Automated Market Maker (AMM). This innovative tool transcends traditional boundaries by amalgamating all assets into a singular trading pool, thereby unlocking efficiencies hitherto unseen. This unique approach not only streamlines trading operations but also optimizes liquidity provision, setting a new standard in the DeFi landscape.

#### Current Scope of the Analysis

<table><thead><tr><th>Contract</th><th>SLOC</th><th>Purpose</th><th>Libraries used</th></tr></thead><tbody><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/omnipool">Omnipool</a></td><td></td><td>Omnipool pallet</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs">omnipool/src/lib.rs</a></td><td>1367</td><td>Omnipool pallet - main pallet's file</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/types.rs">omnipool/src/types.rs</a></td><td>233</td><td>Omnipool pallet - types</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/traits.rs">omnipool/src/traits.rs</a></td><td>162</td><td>Omnipool pallet - traits</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/omnipool">Omnipool Math</a></td><td></td><td>Omnipool math</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/math.rs">math/src/omnipool/math.rs</a></td><td>409</td><td>Omnipool math - math implementation</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/types.rs">math/src/omnipool/types.rs</a></td><td>226</td><td>Omnipool math - types</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/stableswap">Stableswap</a></td><td></td><td>Stableswap pallet</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/lib.rs">stableswap/src/lib.rs</a></td><td>871</td><td>Stableswap pallet - main pallet's file</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/types.rs">stableswap/src/types.rs</a></td><td>136</td><td>Stableswap pallet - types</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/stableswap">Stableswap Math</a></td><td></td><td>Stableswap Math</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/math.rs">math/src/stableswap/math.rs</a></td><td>670</td><td>Stableswap Math - math implementation</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/types.rs">math/src/stableswap/types.rs</a></td><td>25</td><td>Stableswap Math - types</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/ema-oracle">EMA Oracle</a></td><td></td><td>Ema on-chain oracle</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/lib.rs">ema-oracle/src/lib.rs</a></td><td>395</td><td>Ema oracle pallet - main pallet's file</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/types.rs">ema-oracle/src/types.rs</a></td><td>154</td><td>Ema oracle pallet - types</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/ema">Ema Oracle Math</a></td><td></td><td>Omnipool math</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/ema/math.rs">math/src/ema/math.rs</a></td><td>174</td><td>Omnipool math - math implementation</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/circuit-breaker">Circuit breaker</a></td><td></td><td>Circuit breaker</td><td></td></tr><tr><td><a href="https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/circuit-breaker/src/lib.rs">circuit-breaker/src/lib.rs</a></td><td>451</td><td>Circuit breaker- main pallet's file</td><td></td></tr></tbody></table>


## Codebase Quality 

After a deep walkthrough of this codebase, i observed this codebase to be a little more complex and difficult to digest as it was first rust contest to be grasp on. 

`Omnipools`, `stableswap`, `ema-oracle` and `circuit-breaker's` main pallet files named `lib.rs` are the most complex and long files to understand. I couldn't understand properly not even close to 70% these files and i found functionalities of these main files to be the most complex ones from all over the codebase.

Every other rust code except above files was somehow easy to understand GPT-4 helped a ton understanding how complex `traits` and structures are implemented in hypothetical Omnipool.

Protocol handles the complex mathematical part of the protocol very perfeclty like in `math.rs` it uses `Newtons method` for approximating the roots of a real-valued function, which is used in the calculation of the invariant D and the reserve Y. The number of iterations for these calculations is defined by the constants `MAX_D_ITERATION` and `MAX_Y_ITERATIONS`.

## Technical & Architectual Overview

As the codebase was complex and difficult to understand, i found the codebase to be well-architectured and well-structured. The codebase is divided into different modules and sub-modules which makes it easy to understand and navigate through the codebase.

Because of this contest in Rust, it would be difficult to read report for wardens that's why, I will try to provide details about how each and every module works along with architectural view so protocol dev and all other wardens get insights and idea about the protocol.

This codebase has four modules Named `Omnipools`, `StableSwap`, `Ema-Oracle` & `Circuit-Breaker`. Each module has its own main pallet file and some other files like `types.rs` and `traits.rs` which contains the types and traits used in the main pallet file.

Details will be in a sequence Module by Module.

Starting from `Omnipool`

1. ### Omnipool

Omnipool is the main pallet of the protocol. It is the Automated Market Maker (AMM) of the protocol. It is the main module that handles the liquidity provision and trading operations of the protocol. It is the most complex module of the protocol.

It contains `lib.rs` which handles the most functionality and also the complex ones.

The other two files are `types.rs` and `traits.rs` which contains the types and traits used in the `lib.rs` file.

- #### Traits.rs

Traits.rs, part of the Substrate framework, manages assets in a DeFi liquidity pool. The `AssetInfo` struct holds asset details, including its ID, state changes, and withdrawal safety. The `OmnipoolHooks` trait handles events in the pool, such as liquidity changes, trades, and fees. The `ExternalPriceProvider` and ShouldAllow traits get asset prices and validate them, respectively. The `EnsurePriceWithin` struct checks if an asset price is within a certain range, using an external oracle. The `impl_trait_for_tuples::impl_for_tuples(5)` macro allows checking prices for up to five assets simultaneously.

<a href="https://imgur.com/9r2gCtq"><img src="https://i.imgur.com/9r2gCtq.png" title="source: imgur.com" /></a>

- #### Types.rs

1.  `AssetInfo` struct stores asset information including ID, state changes, and withdrawal safety.
2.  OmnipoolHooks trait defines methods triggered by liquidity pool events. It's generic over types like Origin, AccountId, AssetId, and Balance. A default implementation is provided for the unit type.
3.  `ExternalPriceProvider` trait fetches an asset's price. It has a `get_price` method that returns a price or an error.
4. `ShouldAllow` trait checks if an action should be allowed based on an asset's price. It has an `ensure_price` method that returns a Result indicating the action's allowance.
5.  `EnsurePriceWithin` struct ensures an asset's price is within certain bounds. It implements the `ShouldAllow` trait and bypasses the check if the account is whitelisted.

<a href="https://imgur.com/SrtFAu5"><img src="https://i.imgur.com/SrtFAu5.png" title="source: imgur.com" /></a>

2. ### OmniPool Math

Omnipool Math is the module that handles the mathematical operations of the Omnipool module. It contains `math.rs` which handles the mathematical operations of the Omnipool module.

- #### Math.rs

`Math.rs` defines several functions that perform calculations related to trading operations, fees, and balances.

`amount_without_fee(amount: Balance, fee: Permill) -> Option<Balance>:` This function calculates the amount after subtracting a fee. The fee is represented as a percentage (Permill is a type representing parts-per-million). The function subtracts the fee from 100% to get the remaining percentage, then multiplies this by the amount. The result is the amount after the fee has been deducted.

`calculate_imbalance_in_hub_swap(total_hub_reserve: Balance, delta_hub_reserve: Balance`, `imbalance: I129<Balance>) -> Option<Balance>:` This function calculates the `imbalance` in a hub swap. It takes the total hub reserve, the change in hub reserve, and the current imbalance as inputs. The function calculates a new imbalance based on these values. It appears to be used in the context of a trading system where balances can be negative, indicating a deficit.

`calculate_fee_amount_for_buy(fee: Permill, amount: Balance) -> Balance:` This function calculates the fee amount for a buy operation. It takes a fee (as a percentage) and an amount as inputs. The function calculates the fee amount based on these values. If the fee is 0%, the function returns 0. If the fee is 100%, the function returns the amount. For other values, the function calculates the fee as a proportion of the amount, rounding up to the nearest integer.

`calculate_delta_imbalance(delta_hub_reserve: Balance, imbalance: I129<Balance>`, `hub_reserve: Balance) -> Option<Balance>:` This function calculates the change in imbalance. It takes the change in hub reserve, the current imbalance, and the hub reserve as inputs. The function calculates a new imbalance based on these values. It appears to be used in the context of a trading system where balances can be negative, indicating a deficit. The function only supports negative imbalances.

<a href="https://imgur.com/CAACqNC"><img src="https://i.imgur.com/CAACqNC.png" title="source: imgur.com" /></a>

- #### Types.rs

In OnmipooMath::Types.rs, 
The `checked_add` function is a method of the `BalanceUpdate` enum. This function takes another BalanceUpdate as an argument and returns an `Option<BalanceUpdate>`. The function uses Rust's match expression to handle different combinations of Increase and Decrease variants of BalanceUpdate. The `checked_add` method is used to safely add two balances together. If the addition of the two balances overflows, it will return None. Similarly, the `checked_sub` method is used to safely subtract one balance from another, returning None if the operation would result in a negative number.

The default function is an implementation of the Default trait for the BalanceUpdate enum. The Default trait in Rust is used to create a default value of a data type. In this case, the default value for a BalanceUpdate is an Increase variant with a default Balance. The `Balance::default()` call is a placeholder and would be replaced with the actual default value of the Balance type in the real code.

2. ### StableSwap

StableSwap is the module that handles the stableswap functionality of the protocol. It is the module that handles the stablecoin trading operations of the protocol. It is the second most complex module of the protocol.

It contains `lib.rs` which handles the most functionality and also the complex ones.
And types.rs which contains the types used in the `lib.rs` file.

- #### Lib.rs

The Pallet implementation block contains several callable functions (extrinsics) that allow users to interact with the pallet. These include functions to create a pool, update a pool's fee or amplification, add or remove liquidity, and execute trades. Each function is annotated with a weight, which is used for transaction fee calculation, and is marked as transactional, which means that changes will be rolled back if the function fails.

The `create_pool` function allows an authorized origin to create a new pool with a given list of assets, amplification, and fee. The `update_pool_fee` and `update_amplification` functions allow an authorized origin to update a pool's fee or amplification. The `add_liquidity` and `add_liquidity_shares` functions allow a user to add liquidity to a pool, either by specifying the assets and amounts to add, or by specifying the shares to receive. The `remove_liquidity_one_asset` and `withdraw_asset_amount` functions allow a user to remove liquidity from a pool, either by specifying the shares to burn, or by specifying the asset amount to receive.

<a href="https://imgur.com/VOLWyey"><img src="https://i.imgur.com/VOLWyey.png" title="source: imgur.com" /></a>

- #### Types.rs 

The `has_unique_elements` function  generic function that checks if all elements in an iterator are unique. This function takes a mutable reference to an iterator as an argument. The iterator can be of any type T that implements the Iterator trait, and the items yielded by the iterator must implement the Ord trait, which means they are orderable (i.e., they can be compared for ordering).

Inside the function, a mutable `BTreeSet` named uniq is created. A BTreeSet is a data structure in Rust that is a set backed by a `B-Tree`. It automatically sorts its elements and ensures that there are no duplicates.

The function then calls the all method on the iterator, passing in a closure that attempts to insert each element x into the uniq set. The insert method of BTreeSet returns true if the value was inserted successfully (i.e., the value was not already present in the set) and false otherwise.

The all method checks whether all elements of the iterator satisfy a predicate. In this case, the predicate is the result of the insert operation. If all insert operations return true, then all will return true, indicating that all elements in the iterator are unique. If any insert operation returns false, all will immediately return false, indicating that there is a duplicate element in the iterator.

<a href="https://imgur.com/y3Ea8H9"><img src="https://i.imgur.com/y3Ea8H9.png" title="source: imgur.com" /></a>

3. ### StableSwap Math

- #### Math.rs

It provides functions for calculating the amount of assets to be received or sent to a liquidity pool, the amount of shares to be given to a liquidity provider (LP), and the amount of an asset to be withdrawn or added as liquidity in exchange for a given amount of shares.

The code uses `Newton's method` for approximating the roots of a real-valued function, which is used in the calculation of the invariant D and the reserve Y. The number of iterations for these calculations is defined by the constants `MAX_D_ITERATIONS` and MAX_Y_ITERATIONS.

The `calculate_out_given_in` and `calculate_in_given_out` functions calculate the amount to be received from the pool given the amount to be sent to the pool and vice versa. They use the Newton's method to calculate the new reserve Y and then normalize the output amount.

The `calculate_out_given_in_with_fee` and `calculate_in_given_out_with_fee` functions are similar to the previous ones but they also take into account the fee that is applied when making the swap.

The `calculate_shares` function calculates the amount of shares to be given to a liquidity provider after they provided liquidity of some assets to the pool. It uses the invariant D to calculate the new amount of shares.

The `calculate_shares_for_amount` function calculates the amount of shares to be given to a liquidity provider after they provided liquidity of one asset with a given amount.

The `calculate_withdraw_one_asset` function calculates the amount of a selected asset to be withdrawn given the amount of shares and the asset reserves.

The `calculate_add_one_asset` function calculates the amount of an asset that has to be added as liquidity to the pool in exchange for a given amount of shares.

The `calculate_d` function calculates the invariant D, which is a value that remains constant regardless of price fluctuations in a liquidity pool.

The `calculate_y_given_in` and `calculate_y_given_out` functions calculate the new amount of reserve Y given the amount to be added or withdrawn from the pool.

The `calculate_amplification` function calculates the current amplification value, which is a parameter that affects the shape of the curve in a stableswap or AMM model.

<a href="https://imgur.com/XPAAoYK"><img src="https://i.imgur.com/XPAAoYK.png" title="source: imgur.com" /></a>

4. ### Ema-Oracle

Ema-Oracle is the module that handles the EMA (Exponential Moving Average) Oracle functionality of the protocol. It is the module that handles the on-chain oracle functionality of the protocol.

- #### Lib.rs

It uses a number of Substrate's built-in libraries, such as `frame_support` and `frame_system` as well as some custom traits defined in hydradx_traits.

The module is structured around a pallet which contains the main logic. The pallet has a number of associated types defined in its Config trait, which allow it to be customized for different use cases. These include RuntimeEvent, WeightInfo, BlockNumberProvider, SupportedPeriods, and MaxUniqueEntries.

The pallet also defines a number of storage items. Accumulator is used to store oracle data for the current block, which is then recorded at the end of the block. Oracles is a more complex storage item that stores the data for each oracle, keyed by the data source, the assets involved, and the period length of the oracle.

The pallet provides a number of public functions that allow other parts of the blockchain to interact with it. These include on_entry, on_trade, on_liquidity_changed, `last_block_oracle`, `update_oracles_from_accumulator`, `update_oracle`, and `get_updated_entry`.

The module also defines a number of helper functions and structs, such as OnActivityHandler, determine_normalized_price, determine_normalized_volume, determine_normalized_liquidity, ordered_pair, and OracleError.

<a href="https://imgur.com/ITCPIHA"><img src="https://i.imgur.com/ITCPIHA.png" title="source: imgur.com" /></a>

- #### Types.rs

The `raw_data` method returns the raw data of the entry as a tuple of tuples, excluding the block number. It returns the price, volume, and liquidity of the oracle entry.

The `update_outdated_to_current` method is used to update an outdated oracle entry to the current one. It takes a period and an update_with entry as parameters. The period is used to determine the smoothing factor alpha for an exponential moving average. The update_with entry should be more recent than the current entry. The method uses the difference between updated_at to determine the time (i.e., iterations) to cover. If the calculations are successful, the method updates the current entry and returns a mutable reference to it.

The `into_smoothing` function converts a given period into the smoothing factor used in the weighted average. The smoothing factor is a value between 0 and 1 that determines the weight of the most recent data points in the moving average calculation. The function uses a match expression to map each period to a specific smoothing factor. The smoothing factors are represented as fractions and are created from a bit representation using the `from_bits` method. The specific bit values are not immediately clear from the code, but they are likely chosen to give a specific desired behavior for the moving average calculation.

<a href="https://imgur.com/AzoENJg"><img src="https://i.imgur.com/AzoENJg.png" title="source: imgur.com" /></a>


5. ### Ema-Math

- #### Math.rs

Math.rs defines several types and functions:

`EmaPrice`, `EmaVolume`, and `EmaLiquidity` are type aliases used to represent different types of data that the EMA calculations might be applied to. EmaPrice is a ratio, while EmaVolume and EmaLiquidity are tuples of balances.

`exp_smoothing` is a function that calculates the exponential smoothing factor for a given original smoothing factor and number of iterations. It does this by exponentiating the complement of the smoothing factor by the number of iterations.

`price_weighted_average`, `balance_weighted_average`, `volume_weighted_average`, and liquidity_weighted_average are functions that calculate the weighted average of previous and incoming values for price, balance, volume, and liquidity respectively. The weight given to the incoming value is specified by the weight parameter.

`saturating_sub` is a utility function that subtracts one EmaPrice from another, returning a tuple of U256 for full precision. If the second EmaPrice is greater than or equal to the first, the function saturates, meaning it returns the maximum possible value.

multiply is another utility function that multiplies a Fraction with a rational number of U256s, returning a tuple of U512s for full precision.

round and `round_to_rational` are functions that reduce the precision of a 512-bit rational number to 383 bits and 128 bits respectively. They do this by shifting the bits, which implicitly rounds down both the numerator and denominator.

rounding_add and rounding_sub are functions that add or subtract two EmaPrices and round the result to a 128-bit rational number. The precision of the second EmaPrice is reduced to 383 bits before the operation to prevent saturation.

<a href="https://imgur.com/ePPBpWx"><img src="https://i.imgur.com/ePPBpWx.png" title="source: imgur.com" /></a>

6. ### Circuit-Breaker

- #### Lib.rs

The Config trait is used to define the configuration of the pallet. It includes types for events, asset identifiers, balances, and other parameters. It also includes constants for default maximum trade volume and liquidity limits.

The Pallet struct is the main struct for this module, and it is declared with the `#[pallet::pallet]` attribute. The Pallet struct doesn't contain any data, but it serves as a namespace for the callable functions that are defined in the impl block.

The Event enum defines the types of events that can be emitted by the pallet. These events include changes to trade volume limits and liquidity limits.

The Error enum defines the types of errors that can be returned by the pallet's functions. These errors include invalid limit values, liquidity limits not being stored for an asset, and limits being reached.

The impl block for `Pallet<T>` contains the callable functions that can be invoked by transactions. These functions include set_trade_volume_limit, set_add_liquidity_limit, and `set_remove_liquidity_limit`. These functions are used to set the trade volume and liquidity limits for a specific asset.

The `TradeVolumeLimit` and `LiquidityLimit` structs are used to represent the trade volume and liquidity limits for an asset. They include methods for updating amounts and checking limits.

The `AllowedTradeVolumeLimitPerAsset`, `AllowedAddLiquidityAmountPerAsset`, and `AllowedRemoveLiquidityAmountPerAsset` storage items are used to store the allowed trade volume and `liquidity` amounts for each asset.

The Hooks implementation block contains functions that are called at the beginning and end of each block. These functions are used to clear the allowed trade volume and liquidity amounts at the end of each block.

<a href="https://imgur.com/TXaOzTB"><img src="https://i.imgur.com/TXaOzTB.png" title="source: imgur.com" /></a>

## Security Consideration & Recommendations

### OmniPool
#### trait.rs
- Considerations:

The `ExternalPriceProvider` trait relies on an external oracle for price information. This could be a potential point of failure if the oracle is compromised or provides inaccurate data.
The ShouldAllow trait and EnsurePriceWithin struct implement checks for price validity. If these checks are not robust, it could lead to incorrect trades or other operations.
- Recommendations:

Ensure the oracle used for ExternalPriceProvider is reliable and secure.
Implement thorough testing and possibly formal verification of the ShouldAllow trait and EnsurePriceWithin struct to ensure their correctness.
#### types.rs
- Considerations:

The AssetInfo struct holds sensitive information about assets. If not handled properly, it could lead to information leakage or incorrect operations.
The OmnipoolHooks trait defines methods that are called on certain events. If these methods are not implemented correctly, it could lead to incorrect state updates or other issues.
- Recommendations:

Ensure proper `access control` and data handling for the AssetInfo struct.
Implement thorough testing and possibly formal verification of the OmnipoolHooks trait and its implementations.
### OmniPool Math
#### math.rs
- Considerations:

The mathematical functions defined in this file are critical for the correct operation of the OmniPool. Errors in these functions could lead to incorrect trades, incorrect fee calculations, or other issues.
- Recommendations:

Implement thorough testing and possibly formal verification of these mathematical functions to ensure their correctness.
#### types.rs
- Considerations:

The `BalanceUpdate enum` and its methods are used for updating balances. If these are not implemented correctly, it could lead to incorrect balance updates.
- Recommendations:

Implement thorough testing and possibly formal verification of the BalanceUpdate enum and its methods to ensure their correctness.
### StableSwap
#### types.rs
- Considerations:

The `has_unique_elements` function is used to check if all elements in an iterator are unique. If this function is not implemented correctly, it could lead to incorrect results.
- Recommendations:

Implement thorough testing and possibly formal verification of the has_unique_elements function to ensure its correctness.
### StableSwap Math
#### math.rs
- Considerations:

The mathematical functions defined in this file are critical for the correct operation of the StableSwap. Errors in these functions could lead to incorrect trades, incorrect share calculations, or other issues.
- Recommendations:

Implement thorough testing and possibly formal verification of these mathematical functions to ensure their correctness.
#### types.rs
- Considerations:

The `AssetReserve` struct and its methods are used for managing asset reserves. If these are not implemented correctly, it could lead to incorrect reserve updates.
- Recommendations:

Implement thorough testing and possibly formal verification of the AssetReserve struct and its methods to ensure their correctness.
### Ema Oracle
#### lib.rs
- Considerations:

The `update_oracles_from_accumulator` and `update_oracle` functions update oracle data. If these functions are not implemented correctly, it could lead to incorrect oracle updates.
The `on_entry`, `on_trade`, and `on_liquidity_changed` functions are called on certain events. If these functions are not implemented correctly, it could lead to incorrect state updates or other issues.
- Recommendations:

Implement thorough testing and possibly formal verification of the `update_oracles_from_accumulator`, `update_oracle`, `on_entry`, `on_trade`, and `on_liquidity_changed` functions to ensure their correctness.
#### types.rs
- Considerations:

The `update_outdated_to_current` method updates an outdated oracle entry. If this method is not implemented correctly, it could lead to incorrect oracle updates.
- Recommendations:

Implement thorough testing and possibly formal verification of the `update_outdated_to_current` method to ensure its correctness.
### EmaOracleMath
#### math.rs
- Considerations:

The mathematical functions defined in this file are critical for the correct operation of the Ema Oracle. Errors in these functions could lead to incorrect EMA calculations or other issues.
- Recommendations:

Implement thorough testing and possibly formal verification of these mathematical functions to ensure their correctness.
### Circuit-breaker
#### Math.rs
- Considerations:

The `set_trade_volume_limit`, `set_add_liquidity_limit`, and `set_remove_liquidity_limit` functions update limits. If these functions are not implemented correctly, it could lead to incorrect limit updates.
The TradeVolumeLimit and LiquidityLimit structs and their methods are used for managing limits. If these are not implemented correctly, it could lead to incorrect limit updates.
- Recommendations:

Implement thorough testing and possibly formal verification of the `set_trade_volume_limit`, `set_add_liquidity_limit`, `set_remove_liquidity_limit functions`, and the TradeVolumeLimit and LiquidityLimit structs and their methods to ensure their correctness.

## What Protocol Did Unique

I want to mention few concepts that protocol adapts and implements in a unique way.

1. ### Flexible Price Validation: 
The protocol uses the ShouldAllow trait and EnsurePriceWithin struct to validate prices. This allows for flexible price validation strategies, including the use of an external oracle and a whitelist of accounts that bypass price checks.

2. ### Comprehensive Event Hooks: 
The OmnipoolHooks trait provides hooks for various events such as changes in liquidity, trades, and fees. This allows for custom logic to be executed when these events occur.

3. ### Advanced Mathematical Calculations: 
The protocol uses advanced mathematical calculations for managing trades and balances of assets. This includes the use of Newton's method for approximating the roots of a real-valued function, which is used in the calculation of the invariant D and the reserve Y.

4. ### Unique Element Checking: 
The has_unique_elements function checks if all elements in an iterator are unique. This can be useful in various scenarios, such as validating input data.

5. ### Exponential Moving Average (EMA) Oracle: 
The protocol includes an EMA oracle, which calculates the exponential moving average of price, volume, and liquidity data. This can provide more accurate and responsive data for decision-making processes in the protocol.

6. ### Circuit Breaker Mechanism: 
The protocol includes a circuit breaker mechanism that allows setting trade volume and liquidity limits for specific assets. This can help manage risk and prevent market manipulation.

7. ### Asset Reserve Management: 
The AssetReserve struct and its methods are used for managing asset reserves. This allows for detailed tracking and manipulation of asset reserves in the liquidity pool.

8. ### Substrate Framework: 
The protocol is built using the Substrate framework, which provides a robust and flexible foundation for building blockchains. This allows the protocol to leverage Substrate's advanced features such as modular architecture, customizable consensus mechanisms, and interoperability with other blockchains.


## Centralization & Systematic Risks 


1. ### Reliance on External Oracles: 
The ExternalPriceProvider trait relies on an external oracle for price information. If this oracle is controlled by a centralized entity, it could manipulate the price data to their advantage, affecting the operations of the protocol.

2. ### Whitelisting of Accounts: 
The ShouldAllow trait and EnsurePriceWithin struct allow for whitelisting of accounts that bypass price checks. If the process of whitelisting is controlled by a centralized entity, it could lead to preferential treatment of certain accounts, introducing centralization.

3. ### Control Over Trade Volume and Liquidity Limits: 
In the Circuit-breaker module, functions like set_trade_volume_limit, set_add_liquidity_limit, and set_remove_liquidity_limit are used to set trade volume and liquidity limits for specific assets. If these functions are controlled by a centralized entity, it could manipulate these limits to their advantage.

4. ### Control Over Asset Information: 
The AssetInfo struct holds sensitive information about assets. If the process of updating and managing this information is controlled by a centralized entity, it could lead to manipulation of asset information.

5. ### Implementation of Event Hooks: 
The OmnipoolHooks trait defines methods that are called on certain events. If the implementation of these methods is controlled by a centralized entity, it could lead to manipulation of the protocol's behavior.

## Approach Taken in Evalutaing Codebase 

For auditing this codebase, I've adopted a mixture of `Blackboxing` & `SpeedRun` approach that is a combination well structured on focuing on just the main features of the protocol. i.e. Focusing on `ASSETs` Main `Actors` (Power Of Role & Normal Users), and `Action` (When and what can be done on what for whom) & Understand the protocol/expected scenarios through tests, and play around with them to break the developer assumptions and spot a bug as well which was submitted and It's affecting the Protocol Actors and Funds as well.

1. ##### Overview Page On C4: 

Started Auditing HydraDx Protocol By Opening the [Audit Page](https://code4rena.com/audits/2024-02-hydradx) on C4 and read the overview of the protocol and the scope of the audit.

Contest page On C4 Gave ideas and introduction about the protcols, Known issues and Attack vectors to look for in the codebase which helped me to focus on the main functionalities and features of the protocol.

2. ##### Codebase Walkthrough:

Cloned repo from Github and understood the basic structure of the codebase and codebase architecture was clean and well structured.

3. ##### Setting Up & Testing Codebase 

As it was my first contest testing rust code, for the proper setup of the codebase, rust component `cargo` for setting up rust environment,  installing libraries and running tests was used.

As, I'm on linux OS, I installed the cargo by following command listed up in [installation](https://doc.rust-lang.org/cargo/getting-started/installation.html) section of `The Cargo Book`

```bash
curl https://sh.rustup.rs -sSf | sh
```

Rust is completely set up and now it was turn to run all the important tests of the HydraDX protocol.

Ran and tested all the tests listed on the overview page of the protocol on C4. 

- Ran Parallel Tests

- Ran Math Tests

- Ran Integration Test

All tests were executed successfully and little warning were generated which i mentioned above in security consideration and recommendations.

4. ##### Static Analysis 

- ##### Using Clippy For Linting

Then i set up clippy a rust based Linter to find common mistaked, code errors in Rust, it was successfull in finding deprecated keywords and some code functions that were deprecated and errored but fortunately it was in out of scope files so the scoped code was safer according to clippy. 

One thing was odd that at many instance clippy was disables because it might generated any warning there.

So, it should be better to fix the warning generate by clippy instead of disabling clippy.

## Time Spend 

- Joined Contest 4 Days before End of the contest 

- Spend 8-10 on Codebase for understanding and testing

- Focused only on the main components that were crucial to the protocol

- Found some weak spots and submitted One Medium Severity Bug

- Make visual diagrams for understanding protocol

- Wrote Analysis report for the understanding of Wardens or whoever want to deep dive in this protocol.

20 Hours 

### Time spent:
20 hours