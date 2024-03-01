
  

# HydraDX Audit Analysis
![download (3)](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/25e60678-e3b3-42cd-a3fc-8feae8a231be)

HydraDX is a next-gen DeFi protocol which is designed to bring an ocean of liquidity to Polkadot. Our tool for the job the HydraDX Omnipool - an innovative Automated Market Maker (AMM) which unlocks unparalleled efficiencies by combining all assets in a single trading pool.

# Overview

Structuring the code is crucial for conducting a thorough analysis of the code base. This enables auditors to predict the level of contract difficulty, identify critical contracts, and determine security-critical features such as payment functions or assembly usage. Additionally, this approach accurately measures the time required to perform a detailed audit and helps determine the cost of a project audit. To achieve efficiency, clarity, and improvements, it is essential to have a comprehensive view of the entire architecture. This enables the identification of the basic structure, relationships between modules, and key design patterns. By understanding the big picture, we can analyze the complexity of the code and reveal its strengths and potential for improvement with confidence.


## Meet Omnipool: Revolutionizing Liquidity on Polkadot

# Introduction

HydraDX introduces the Omnipool, a groundbreaking Automated Market Maker (AMM) designed to revolutionize liquidity provision in the DeFi space, particularly within the Polkadot ecosystem. This audit delves into the core features and advantages of the Omnipool, highlighting its role in enhancing trading efficiency, reducing impermanent loss, and empowering users through innovative functionalities such as single-sided liquidity provisioning.Users can supply liquidity, trade, use DCA (Dollar-Cost Averaging) and Split Trade features, participate in Hydrated Farms, join staking programs, and use the Yield DCA feature. All these interactions show how users interact with the system to benefit from the features offered by HydraDX Omnipool.

![HydraXProtocol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/8e280f8e-c6e7-4acb-84c7-f6dea42750db)


## Efficient Trading: The Need for Unified Liquidity

In traditional DeFi landscapes, liquidity is fragmented across multiple trading pools, leading to inefficiencies such as increased price impact and slippage. HydraDX addresses this challenge with the Omnipool, which combines all liquidity into a single pool, enabling seamless trading with minimal slippage and hops.

## Traditional AMMs vs. Omnipool

Traditional AMMs operate with fragmented liquidity, constrained to pairs of assets, resulting in higher price impact and slippage due to multiple hops. In contrast, the Omnipool consolidates liquidity, offering direct access to the entirety of liquidity for any asset deposited, thereby minimizing price impact and enhancing trading efficiency.

## Single-Sided Liquidity Provisioning: Empowering Users

The Omnipool introduces the concept of single-sided liquidity provisioning, allowing users to provide liquidity for a specific asset without the need for a paired asset. This innovative approach eliminates liquidity fragmentation and provides instant exposure to a diverse range of assets within the pool.

## Hub Token LRNA: Enabling Single-Sided LPing

Single-sided liquidity provisioning is facilitated by the hub token LRNA, which maintains balance within the Omnipool by minting and burning tokens corresponding to added or removed liquidity. LRNA serves as a proxy for the value locked in the pool, ensuring stability and efficiency in liquidity provision.

## Enhancing Trading Strategies: DCA and Split Trade

HydraDX introduces Dollar-Cost Averaging (DCA) and Split Trade features, enabling users to implement sophisticated trading strategies with ease. DCA allows for gradual execution of orders to mitigate price volatility, while Split Trade minimizes slippage by splitting large orders into smaller chunks.

## HydraDX DCA and Split Trade

HydraDX offers intuitive interfaces for DCA and Split Trade, empowering users to customize trading strategies according to their preferences. These features enhance liquidity provision and trading efficiency, catering to both novice and experienced traders.

## Incentivizing Liquidity Provision: Hydrated Farms

Hydrated Farms incentivize liquidity provision for selected assets through time-limited reward programs. These programs offer additional rewards on top of trading fees, encouraging sustained participation and liquidity provision within the Omnipool.

## Farm NFTs and Rewards Stacking

Hydrated Farms utilize NFTs to represent user positions, enabling sustainable reward distribution over time. Rewards from multiple assets can be stacked, providing enhanced incentives for liquidity provision and participation in the HydraDX ecosystem.

## Maximizing Protocol Growth: Bonds and LBP

HydraDX employs Bonds and Liquidity Bootstrapping Pools (LBP) to facilitate protocol growth and diversification of Protocol-owned liquidity (POL). Bonds enable the acquisition of assets through time-bound campaigns, while LBP facilitates fair token distribution and liquidity provision for emerging projects.

## The Role of Bonds and LBP

Bonds campaigns enable the acquisition of assets for POL, contributing to protocol growth and sustainability. LBP fosters fair token distribution and liquidity provision, empowering emerging projects within the Polkadot ecosystem.

## What are the features in HydraDX Omnipool?

- Supplying Liquidity:

Users can provide liquidity for certain assets in Omnipool. This allows users to earn rewards in the form of trading fees and additional incentives.
Making Trades:

- Making Trades:

Users can trade crypto assets directly within Omnipool without the need to go through a centralized exchange or another AMM. This provides convenience and efficiency in executing trades.

- Using the DCA (Dollar-Cost Averaging) Feature:

The DCA feature allows users to make periodic purchases of a fixed amount at specific time intervals. This helps reduce the impact of price volatility and allows users to average their purchase price over time.

- Using the Split Trade Feature:

The Split Trade feature breaks down large trade orders into smaller parts to be executed in stages. This helps reduce slippage and ensures that orders can be executed at better prices.

- Participate in Hydrated Farms:

Hydrated Farms is an incentive program that allows users to earn additional rewards by providing liquidity for certain assets within Omnipool. These additional rewards come from trading fees and additional incentive programs.

- Joining the Staking Program:

The Staking Program allows token holders to place their tokens in smart contracts for rewards. These rewards are derived from various protocol revenue sources, such as trading fees and other incentives.

- Using the Yield DCA Feature:

The Yield DCA feature allows users to shift the yield from staking tokens (e.g. DOT) to other assets traded on Omnipool periodically. This allows portfolio diversification and reduces risk.

# Codebase quality

Overall, I consider the quality of the HydraDX codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

## Omnipool

| File            | Issue                                                                                                 | Recommendation                                                                                                                                                      |
|-----------------|-------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `lib.rs`        | Excessive memory allocation within the `update_hdx_subpool_hub_asset` function.                      | Consider optimizing memory usage by using stack allocation where possible, or reusing existing memory buffers instead of creating new ones.                      |
| `lib.rs`        | Potential resource leak due to failure to handle errors in the `buy_hub_asset` function.             | Implement error handling mechanisms such as `Result` or `Option` to properly handle errors and prevent resource leaks.                                            |
| `lib.rs`        | Inefficient algorithm used for calculating state changes in the `sell_asset_for_hub_asset` function. | Explore more efficient algorithms or data structures to improve the performance of the function, such as binary search or hash maps.                            |
| `lib.rs`        | Redundant memory allocations in the `process_trade_fee` function.                                    | Utilize object pooling or recycling mechanisms to reduce the frequency of memory allocations and deallocations, thereby improving performance.                    |
| `lib.rs`        | Excessive branching and conditional logic in the `load_position` function.                            | Simplify the logic by refactoring the function into smaller, more focused units, and reducing the number of conditional branches where possible.                  |
| `lib.rs`        | Potential performance bottleneck in the `set_position` function due to frequent database accesses.    | Consider implementing caching mechanisms to store frequently accessed data in memory, reducing the number of database queries and improving performance.          |

## Omnipool Math

| Issue Description                                                                                                      | Recommendation                                                                                                                                                                                                |
|------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **calculate_sell_state_changes** :                                                                                    |                                                                                                                                                                                                               |
| - Potential performance issue with multiple divisions and multiplications using `checked_mul`, `checked_div` methods.    | Consider refactoring the algorithm to minimize the number of divisions and multiplications, as they can be computationally expensive operations.                                                            |
| - Excessive use of `to_balance!` macro for converting U256 to Balance type.                                              | Evaluate if direct conversion or different type usage could optimize the conversion process.                                                                                                                |
| - Use of `min` function may not be necessary, potentially introducing unnecessary overhead.                              | Evaluate if the usage of `min` function is crucial for the algorithm, and if not, consider alternative approaches to achieve the same outcome without using `min`.                                            |
| **calculate_buy_state_changes** :                                                                                     |                                                                                                                                                                                                               |
| - Similar issues as `calculate_sell_state_changes` function.                                                             | Apply the same recommendations mentioned for the `calculate_sell_state_changes` function.                                                                                                                     |
| **calculate_remove_liquidity_state_changes** :                                                                          |                                                                                                                                                                                                               |
| - Complex logic with multiple calculations involving U256 types.                                                         | Consider simplifying the algorithm and optimizing the calculations to reduce complexity and potential performance overhead.                                                                                |
| - Excessive use of `to_balance!` macro and conversions.                                                                    | Evaluate if conversions can be minimized or optimized to improve performance.                                                                                                                               |
| - Usage of `FixedU128` type for `withdrawal_fee` calculation may need further validation.                                 | Ensure that the `FixedU128` type and associated calculations for `withdrawal_fee` are necessary and produce accurate results.                                                                              |
| **calculate_tvl_cap_difference** function:                                                                                     |                                                                                                                                                                                                               |
| - Potential performance impact due to multiple conversions and calculations.                                              | Review the necessity of conversions and calculations, and optimize the algorithm to reduce unnecessary operations.                                                                                          |
| - Complexity in logic for calculating TVL cap difference.                                                                | Simplify the logic if possible and ensure that the calculations are accurate and efficient.                                                                                                                  |
| **Overall Codebase:**                                                                                                       |                                                                                                                                                                                                               |
| - Lack of comments and documentation may hinder code maintenance and understanding.                                      | Provide adequate comments and documentation to explain the purpose of each function and its parameters, as well as any complex logic or algorithms employed.                                                |
| - Potential for code duplication across functions.                                                                      | Identify common patterns or functionalities and refactor them into reusable components to reduce redundancy and improve maintainability.                                                                      |


## Stableswap

| Potential Performance Issue  | Code Analysis                                                                                              | Optimization Suggestions                                                                                                                                                                   |
|-------------------------------|------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Inefficient Algorithms       | Some parts of the code use lengthy or nested iterations, such as in the `calculate_shares` and `get_pool_state` functions.          | Consider optimizing algorithms by reducing time complexity or considering more efficient approaches, such as using more suitable data structures.                                             |
| Excessive Memory Usage       | There are several cases where vectors and maps are used without sufficient memory consideration, such as in the `do_add_liquidity` and `get_pool_state` functions.            | Evaluate the memory usage in the usage of vectors and maps. Consider using references or lighter data structures where possible.                                                            |
| Resource Leaks               | Memory or assets are not properly deallocated after use, such as improper memory allocation in the `do_create_pool` function.       | Ensure proper deallocation of allocated memory or resources after use. Use safe conventions and ensure no resource leaks occur.                                                  |

## Stableswap Math

| Identification | Description | Optimization Suggestion |
|--------------|-----------|--------------------|
| Excessive Memory Usage | In several functions, such as `calculate_y_given_in` and `calculate_y_given_out`, there is excessive use of vectors to store data that can be simplified. | Replace the use of vectors with direct references to the original data. This will reduce memory usage and improve performance. |
| Resource Leakage | In the `calculate_shares` function, there is potential resource leakage if `share_issuance` is zero. | Add an initial check to prevent resource leakage if `share_issuance` is zero. |
| Other Code Patterns | In several functions, there is the use of `try_from` and `checked_mul` that can cause compilation or runtime failures. | Replace the use of `try_from` and `checked_mul` with safer and easier-to-read approaches, such as using `unwrap` or `expect`. This will make debugging easier and improve performance. |

## EMA Oracle
1.Excessive Memory Usage
Optimization Suggestions:
- Consider using more efficient data structures such as `BTreeMap` or `HashMap` where applicable.
- Ensure memory usage can be optimized by reducing unnecessary allocations.

## Circuit breaker
| Potential Performance Issues | Description                                                                                                                                               | Optimization Recommendations                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Excessive Memory Usage       | Some functions, such as `update_amounts`, `check_outflow_limit`, and `check_influx_limit`, involve significant memory usage due to the data structures used. | - Ensure to minimize memory usage by considering the selection of more efficient data types if possible.                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Resource Leakage             | There are several arithmetic operations that do not consider resource leakage, such as in the `calculate_limit` function.                                | - Check each arithmetic operation and make sure to handle resource leakage cases, such as overflow and division by zero.                                                                                                                                                                                                                                                                                                                                                                                                                                                          |


# Approach we followed when reviewing the code

`Omnipool` serves as a protocol that facilitates the exchange of assets, specifically assets known as "Hub Assets". i.e. it has several key functions that include adding, deleting, and swapping assets in the pool, as well as processing trading fees and asset deletions.

`Omnipool Math` exchange of assets for liquidity management includes calculation of changes in the state of assets during the process and addition or removal of liquidity from the pool, calculation of transaction costs, and evaluation of assets.

`Stableswap` It is a user implementation to perform such as adding liquidity to the pool, performing asset swaps, and managing the pool and also each operation is performed by fulfilling various requirements and validations. In addition, it uses hooks to execute additional functions after certain operations have been successfully performed.

`Stableswap Math` It uses a constant product formula (x*y=k) for stablecoin exchange, and also includes amplification (similar to Uniswap v3) to improve price precision. Various rounding and normalization functions are used to handle different token decimals and precision requirements and calculate amplification factors based on the initial and final amplification values and block numbers. This allows dynamic adjustment of the amplification factor over time.

`EMA Oracle` part of Oracle's ema-palette that provides exponential moving average (EMA) oracles with multiple periods for price, volume, and liquidity for data-driven combinations of asset sources and pairs

# Analysis of the code base and diagram architecture

## Omnipool 

![omnipool](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/a942d7af-5732-43e0-9b89-0f3bf030d724)

## Omnipool Math
![omnipoolmath](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/95f1a2d1-1ef6-4a7a-97ee-afbb624d4a65)

## Stableswap
![Stableswap](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/fbef9418-e613-4d7b-badd-60898848e4ed)

## Stableswap Math
![testedddz](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/86ff79a8-de35-4c33-9efc-510ce8d7ae9f)

## EMA Oracle
![adwadwa](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/4cfb96db-4afb-422b-8b73-2f5fc2679a80)

## Circuit Breaker
![Circuit breaker](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/48bf1173-df03-4aa6-a37c-cb5b4cd12b07)

# Conclusion

The HydraDX Omnipool represents a paradigm shift in DeFi liquidity provision, offering efficiency, flexibility, and innovation to users within the Polkadot ecosystem. By consolidating liquidity, empowering single-sided liquidity provisioning, and incentivizing participation, HydraDX is poised to reshape the future of decentralized finance on Polkadot.


### Time spent:
32 hours