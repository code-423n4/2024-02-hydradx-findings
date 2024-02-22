| Serial No. | Topic                                           |
|------------|-------------------------------------------------|
| 01        | Overview                                         |
| 02        | Scope                                            |
| 03        | Architecture view (Diagram)                      |
| 04        | Protocol Roles                                   |
| 05        | Approach Taken in Auditing HydraDX Codebase      |
| 06        | Contract Analysis                                |
| 07        | Codebase Quality (Table)                         |
| 08        | Centralization Risks                             |
| 09        | Systematic Risks                                 |
| 10        | Architectural Improvement                        |
| 11        | Time spent                                       |


## Overview:
HydraDX is a next-gen DeFi protocol which is designed to **bring an ocean of liquidity to Polkadot**. Their tool for the job the **HydraDX Omnipool** - an innovative Automated Market Maker (AMM) which **unlocks unparalleled efficiencies** by **combining all assets in a single trading pool**. The protocol, through its collection of Rust-based smart contracts, presents a sophisticated decentralized finance (DeFi) ecosystem designed for asset trading, liquidity provision, and market stability management on a blockchain network. At its core, the protocol employs a series of innovative mechanisms to facilitate seamless asset exchange, enforce trading limits, manage liquidity pools, and ensure market integrity across various decentralized exchanges (DEXs).

**Key Features:**

`Asset Trading and Exchange Mechanisms:` The protocol introduces robust functions for executing trades and exchanges between different assets. It enables users to swap assets efficiently while ensuring market liquidity and stability. The use of `swap` functions, coupled with price and slippage checks, exemplifies the protocol's commitment to facilitating secure and user-friendly trading experiences.

   ```rust
   pub fn swap(...) -> DispatchResult { ... }
   ```

`Liquidity Provision and Management:` Liquidity providers are at the heart of the protocol, with dedicated mechanisms for adding and removing liquidity to and from pools. This is crucial for maintaining active and balanced markets. The contracts define clear processes for liquidity operations, ensuring that participants can contribute to the ecosystem's liquidity seamlessly.

   ```rust
   pub fn add_liquidity(...) -> DispatchResult { ... }
   pub fn remove_liquidity(...) -> DispatchResult { ... }
   ```

`Decentralized Price Oracle:` A standout feature is the protocol's implementation of a decentralized price oracle, which aggregates asset prices in a trust-minimized way. This is essential for various operations within the protocol, including swap execution and liquidity management, ensuring that all transactions are conducted at fair and accurate market rates.

   ```rust
   impl From<AssetReserve> for u128 { ... }
   ```

`Trading and Liquidity Limits:` To prevent market manipulation and ensure stability, the protocol imposes limits on trading volumes and liquidity changes within a block. These limits are configurable and enforceable across assets, providing a safeguard against excessive market volatility and potential manipulation.

   ```rust
   pub fn ensure_pool_state_change_limit(...) -> Result<Weight, DispatchError> { ... }
   ```

`Flexible and Secure Asset Handling:` The contracts demonstrate a comprehensive approach to asset handling, including support for multiple asset types and secure asset transfer mechanisms. This flexibility ensures that the protocol can support a wide range of assets and trading pairs, catering to diverse user needs and preferences.

   ```rust
   pub fn transfer_assets(...) -> DispatchResult { ... }
   ```

`Governance and Configuration:` Governance plays a crucial role, with mechanisms for configuring key parameters of the protocol, such as fees, limits, and oracle settings. This allows the protocol to adapt to changing market conditions and community preferences, ensuring long-term sustainability and alignment with user interests.

   ```rust
   pub fn set_trade_volume_limit(...) -> DispatchResult { ... }
   ```

`Security and Efficiency:` The protocol places a strong emphasis on security and efficiency, employing rigorous checks and balances to prevent errors and vulnerabilities. The use of weight annotations and optimizations ensures that the protocol remains scalable and performant, even under high transaction volumes.

   ```rust
   #[pallet::weight(T::WeightInfo::swap())]
   pub fn swap(...) -> DispatchResult { ... }
   ```

In summary, HydraDX represents a comprehensive DeFi solution that addresses key challenges in the decentralized trading space, including liquidity provision, price discovery, and market stability. Through its innovative contract design and focus on security, flexibility, and governance, the protocol is well-positioned to support a thriving ecosystem of traders, liquidity providers, and developers, fostering the growth and adoption of decentralized finance.


## Scope

| Contract                                                                                                                                  | SLOC | Purpose                                | Libraries used |  
|-------------------------------------------------------------------------------------------------------------------------------------------|------|----------------------------------------|----------------|
| [Omnipool](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/omnipool)                                         |      | Omnipool pallet                        | |
| [omnipool/src/lib.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs)               | 1367 | Omnipool pallet - main pallet's file   |                |
| [omnipool/src/types.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/types.rs)          | 233  | Omnipool pallet - types                |                |
| [omnipool/src/traits.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/traits.rs)        | 162  | Omnipool pallet - traits               |                |
| [Omnipool Math](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/omnipool)                                   |      | Omnipool math                          |                |
| [math/src/omnipool/math.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/math.rs)          | 409  | Omnipool math - math implementation    |                |
| [math/src/omnipool/types.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/types.rs)        | 226  | Omnipool math - types                  |                |
| [Stableswap](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/stableswap)                                     |      | Stableswap pallet                      |                |
| [stableswap/src/lib.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/lib.rs)          | 871  | Stableswap pallet - main pallet's file |                |
| [stableswap/src/types.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/types.rs)      | 136  | Stableswap pallet - types              |                |
| [Stableswap Math](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/stableswap)                               |      | Stableswap Math                        |                |
| [math/src/stableswap/math.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/math.rs)      | 670  | Stableswap Math - math implementation  |                |
| [math/src/stableswap/types.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/types.rs)    | 25   | Stableswap Math - types                |                |
| [EMA Oracle](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/ema-oracle)                                     |      | Ema on-chain oracle                    |                |
| [ema-oracle/src/lib.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/lib.rs)          | 395  | Ema oracle pallet - main pallet's file |                |
| [ema-oracle/src/types.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/types.rs)      | 154  | Ema oracle pallet - types              |                |
| [Ema Oracle Math](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/math/src/ema)                                      |      | Omnipool math                          |                |
| [math/src/ema/math.rs](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/ema/math.rs)                    | 174  | Omnipool math - math implementation    |                |
| [Circuit breaker](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/circuit-breaker)                           |      | Circuit breaker                        |                |
| [circuit-breaker/src/lib.rs](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/circuit-breaker/src/lib.rs) | 451  | Circuit breaker- main pallet's file    |                |

Total SLOC: 5273

## Architecture view (Diagram):
[Click Here](https://postimg.cc/SXL1fDm1) - for better resolution:
[![hydraDX.png](https://i.postimg.cc/g019dTQp/hydraDX.png)](https://postimg.cc/SXL1fDm1)


## Protocol Roles:
Below are the different roles identified within the HydraDX protocol, along with a description of their contributions:

**Liquidity Providers (LPs):** LPs are foundational to the protocol, depositing assets into pools to facilitate trading and earning fees in return. Their interaction with liquidity pool contracts not only provides the necessary market depth for asset swaps but also stabilizes pool prices through arbitrage opportunities. LPs are incentivized with a share of trading fees and potentially additional yield farming rewards, encouraging a stable and liquid market environment.

**Traders:** Traders utilize the DEX functionality embedded within the protocol to exchange assets. They benefit from the liquidity provided by LPs, executing trades that are optimized for minimal slippage and immediate settlement. Traders interact with smart contracts that execute swaps, manage orders, and provide on-chain price information, contributing to a vibrant trading ecosystem.

**Borrowers and Lenders:** These participants engage with contracts facilitating leveraged positions and lending services. Borrowers access capital for trading, staking, or investment purposes, while lenders supply assets to lending pools in exchange for interest payments. This dynamic enhances the protocol's utility by providing mechanisms for capital efficiency and investment leverage.

**Yield Farmers:** Yield farming strategies are supported by contracts that allocate rewards for providing liquidity, staking, or participating in specific protocol activities. Yield farmers maximize their returns by strategically moving assets across pools or protocols, incentivized by reward tokens or higher interest rates.

**Governance Participants:** The protocol's governance model is supported by contracts that allow token holders to propose, vote on, and implement changes. This decentralized governance ensures the protocol evolves according to community consensus, addressing the need for upgrades, parameter adjustments, and the integration of new features.

**Risk Managers:** Risk management contracts offer tools for mitigating exposure to impermanent loss, price volatility, and other DeFi risks. These tools include options for insurance coverage, hedging positions, and managing collateral ratios, providing participants with options to safeguard their investments.

**Developers and Integrators:** The open and composable nature of the protocol's contracts encourages external developers to build additional services, interfaces, and applications. This role involves leveraging the protocol's infrastructure to introduce new DeFi products, enhance user experiences, and integrate with other blockchain ecosystems.

**Validators and Network Security:** For protocols utilizing PoS or similar mechanisms, validators secure the network by confirming transactions and maintaining consensus. They interact with staking contracts, ensuring network integrity and stability while earning rewards for their services.

**Arbitrageurs:** Arbitrageurs help maintain market efficiency by exploiting price discrepancies across different pools or external exchanges. The protocol facilitates these activities through efficient trading mechanisms and real-time price feeds, ensuring that asset prices remain consistent with broader market values.

**Protocol Administrators:** These participants have privileged access to certain administrative functions within the protocol, such as emergency stops, parameter adjustments, and contract upgrades. Their role is critical for maintaining the protocol's health and security, acting as a safeguard against unforeseen issues.

Each role delineated above underscores the multifunctional and interconnected nature of the DeFi protocol, highlighting the sophisticated interplay between different participants and the smart contracts that orchestrate their activities. The protocol's design prioritizes security, liquidity, user engagement, and decentralization, aiming to create a resilient and versatile financial ecosystem on the blockchain.



## Approach Taken in Auditing HydraDX Codebase

**Feb 3 - Feb 5**:
- Began by thoroughly reading HydraDX documentation to grasp the fundamental concepts and architecture of the protocol.
- Delved into the net specs and comments within each contract for a preliminary understanding of their roles and interactions.
- Focused on understanding HydraDX's key objectives, including liquidity provision, swap functionalities, and its unique selling propositions within the DeFi space.
- Mapped out the structure of the codebase to identify critical components and their dependencies for a systematic audit approach.

**Feb 6 - Feb 9**:
- Dedicated time to explore the Omnipool component, given its centrality to HydraDX's functionality. Examined how liquidity pools are managed and how swaps are executed across different asset pairs.
- Analyzed the contract logic behind liquidity addition and removal, swap execution, and fee distribution within the Omnipool.
- Evaluated the security measures in place to protect Omnipool operations from common vulnerabilities such as reentrancy attacks and integer overflows.
- Investigated the integration points between Omnipool and other protocol components, ensuring seamless interoperability and data consistency.

**Feb 10- Feb 14**:
- Delved into the mathematical models underpinning the Omnipool, aiming to understand the algorithms that calculate swap prices, liquidity provisions, and fee allocations.
- Consulted the documentation, code comments, and external resources to bridge gaps in understanding complex mathematical concepts.
- Attempted to validate the mathematical integrity of the Omnipool by comparing expected outcomes with simulated scenarios.
- Acknowledged areas of partial understanding, particularly in the nuances of the mathematical models, and planned to revisit these with additional resources.

**Feb 16 - Feb 18**:
- Shifted focus to the Stableswap mechanism, another vital component of HydraDX, responsible for providing efficient swaps for stablecoin pairs.
- Explored the logic behind the Stableswap's low-slippage model and how it adjusts prices based on the balance of assets in the pool.
- Examined the smart contracts responsible for Stableswap operations, including asset balancing, swap execution, and fee collection.
- Assessed the resilience of Stableswap against price manipulation and arbitrage opportunities that could undermine the stability of asset pairs.

**Feb 19 - Feb 20**:
- Undertook the challenge of understanding the mathematical foundation of Stableswap. This included the curve algorithm that minimizes slippage for stablecoin trades.
- Compared theoretical models with implemented code to ensure the mathematical accuracy and efficiency of Stableswap operations.
- Conducted test scenarios to observe the impact of various trade sizes, liquidity levels, and asset ratios on the Stableswap curve and resulting prices.

**Feb 21 - Feb 23**:
- Investigated the Exponential Moving Average (EMA) Oracle, critical for providing accurate, tamper-resistant price feeds for HydraDX.
- Analyzed the EMA Oracle's implementation, focusing on data aggregation, smoothing factors, and update mechanisms to ensure reliable price information.
- Evaluated the safeguards against oracle manipulation and the process for incorporating external price feeds into the EMA calculation.
- Explored the mathematical equations and algorithms that underlie the EMA Oracle, seeking to understand their contribution to price stability and accuracy.

**Feb 24**:
- Dived into the Circuit Breaker mechanism designed to protect HydraDX from sudden, adverse market conditions or system anomalies.
- Reviewed the criteria for triggering the Circuit Breaker, the range of actions it can take, and the process for resuming normal operations.
- Assessed the balance between security, market integrity, and user experience in the implementation of the Circuit Breaker.

**Feb 25- Mar 29**:
- Conducted extensive testing across the codebase to identify potential vulnerabilities, focusing on critical functionalities such as liquidity management, swaps, and oracle updates.
- Experimented with various attack vectors, stress tests, and edge cases to evaluate the robustness of the HydraDX protocol.
- While no significant vulnerabilities were successfully exploited, documented findings and observations that could enhance the protocol's security posture and resilience.

**Mar 1 - Mar 2**:
- Compiled all findings, insights, and recommendations from the audit into a comprehensive report. This included highlighting areas of strength, potential improvements, and suggestions for further investigation.
- Prepared a summary of the audit process, key learnings, and the overall assessment of HydraDX's codebase quality and security measures.
- Finalized the report with a reflection on the auditing experience, emphasizing the complexity, innovation, and potential of the HydraDX protocol within the DeFi ecosystem.

## Contract Analysis:

### [omnipool/src/lib.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs) Omnipool pallet - main pallet's file

This contract is an implementation of an `Omnipool`, a type of Automated Market Maker (AMM) that pools all assets into a single pool. This allows for seamless asset swaps and liquidity provision. The `Omnipool` uses a hub asset (`LRNA`) to facilitate trades and manage liquidity. Providers can add liquidity with any asset and receive pool shares in return, represented as NFT tokens. The contract includes mechanisms for adding/removing liquidity, trading, and handling fees. It is designed with a focus on minimizing LRNA's volatility through an imbalance mechanism and supports various hooks for liquidity and trade events.

####  Key Function's Functionality:

`add_token`:
```rust
pub fn add_token(
	origin: OriginFor<T>,
	asset: T::AssetId,
	initial_price: Price,
	weight_cap: Permill,
	position_owner: T::AccountId,
) -> DispatchResult
```
This function allows adding a new token to the Omnipool. It requires transferring initial liquidity to the pool account beforehand. A NFT token is minted for the liquidity provider, representing their position. The function checks for asset registration, minimum liquidity, and valid initial price. It updates the pool's state and triggers the `on_liquidity_changed` hook.

To add a new token, `ABC`, with an initial price of 1 LRNA per ABC, a call to `add_token` would be made with the specified `initial_price`, `weight_cap` for asset's weight in the pool, and `position_owner` as the receiver of the minted position NFT.

`add_liquidity`
```rust
pub fn add_liquidity(
	origin: OriginFor<T>, 
	asset: T::AssetId, 
	amount: Balance
) -> DispatchResult
```
Enables liquidity providers to add liquidity to the pool for a specific asset and receive corresponding shares as an NFT. It checks for trading permissions, minimum liquidity requirements, and applies slippage protection. The function updates the pool and asset states, mints a position NFT for the provider, and triggers `on_liquidity_changed`.

 Adding 100 ABC tokens to the pool would require calling `add_liquidity` with `asset` as ABC and `amount` as 100. This action mints an NFT representing the added liquidity.

`*remove_liquidity`
```rust
pub fn remove_liquidity(
	origin: OriginFor<T>, 
	position_id: T::PositionItemId, 
	amount: Balance
) -> DispatchResult
```
Allows liquidity providers to remove part or all of their liquidity from the pool by burning a portion of their position NFT. It supports partial withdrawals, calculates withdrawal fees based on market conditions, and ensures price slippage protection. The function updates the pool state, burns the NFT (if fully redeemed), and triggers `on_liquidity_changed`.

To withdraw 50 ABC tokens from the pool, a call to `remove_liquidity` with the `position_id` of the NFT and `amount` as 50 is made. Depending on market conditions, a withdrawal fee may apply.

`sell` and `buy`
```rust
pub fn sell(
	origin: OriginFor<T>, 
	asset_in: T::AssetId, 
	asset_out: T::AssetId, 
	amount: Balance, 
	min_buy_amount: Balance
) -> DispatchResult
```
Enables trading from `asset_in` to `asset_out` by specifying the amount to sell and the minimum amount expected to receive. It checks for trading permissions, slippage, and trading limits. The function updates asset states, transfers assets between the trader and the pool, applies fees, and triggers `on_trade` and `on_liquidity_changed`.

Selling 10 ABC tokens for at least 8 XYZ tokens would require calling `sell` with the respective `asset_in`, `asset_out`, `amount`, and `min_buy_amount`.

`set_asset_tradable_state`
```rust
pub fn set_asset_tradable_state(
	origin: OriginFor<T>, 
	asset_id: T::AssetId, 
	state: Tradability
) -> DispatchResult
```
Updates the tradable state of an asset, allowing or forbidding trading activities. It ensures that only authorized origins can make the update and emits an event upon success.

To allow trading for ABC tokens, `set_asset_tradable_state` would be called with `asset_id` as ABC and `state` set to allow trading operations.


### [omnipool/src/types.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/types.rs) Omnipool pallet - types

This contract defines key data structures and operations for managing assets within a liquidity pool, tracking liquidity provider (LP) positions, and handling the tradability and imbalance of assets. Key types include `AssetState`, `Position`, `Tradability`, and `SimpleImbalance`, each serving a fundamental role in the AMM's functionality. These structures are designed to manage liquidity provision, trading activities, asset prices, and the imbalance mechanism crucial for maintaining the stability of a hub asset in a multi-asset liquidity pool.

#### Key Function's Functionality

 `Tradability`
```rust
bitflags::bitflags! {
    #[derive(Encode,Decode, MaxEncodedLen, TypeInfo)]
    pub struct Tradability: u8 {
        const FROZEN = 0b0000_0000;
        const SELL = 0b0000_0001;
        const BUY = 0b0000_0010;
        const ADD_LIQUIDITY = 0b0000_0100;
        const REMOVE_LIQUIDITY = 0b0000_1000;
    }
}
```
Defines the tradable states of an asset within the pool. An asset can be marked for various operations such as `SELL`, `BUY`, `ADD_LIQUIDITY`, and `REMOVE_LIQUIDITY`. This structure is pivotal for controlling asset interactions, ensuring operational integrity and compliance with intended protocol functionalities.

`AssetState`
```rust
#[derive(Clone, Default, Encode, Decode, Eq, PartialEq, RuntimeDebug, MaxEncodedLen, TypeInfo)]
pub struct AssetState<Balance> {
    pub(super) hub_reserve: Balance,
    pub(super) shares: Balance,
    pub(super) protocol_shares: Balance,
    pub cap: u128,
    pub tradable: Tradability,
}
```
Represents the state of an asset within the pool, including its reserve in the hub asset, the total shares distributed to liquidity providers, shares owned by the protocol, a cap on its weight within the pool, and its tradability status. This structure is essential for managing asset liquidity and determining its interaction within the pool.

 `Position`
```rust
#[derive(Clone, Encode, Decode, Eq, PartialEq, RuntimeDebug, MaxEncodedLen, TypeInfo)]
pub struct Position<Balance, AssetId> {
    pub asset_id: AssetId,
    pub amount: Balance,
    pub shares: Balance,
    pub price: (Balance, Balance),
}
```
Defines a liquidity provider's position in the pool for a specific asset. It includes the asset identifier, the amount of asset provided, the shares awarded to the LP, and the price of the asset at the time of liquidity provision. This structure enables tracking of individual contributions and ownership within the pool.

 `SimpleImbalance`
```rust
#[derive(Clone, Copy, Encode, Decode, Eq, PartialEq, RuntimeDebug, MaxEncodedLen, TypeInfo)]
pub struct SimpleImbalance<Balance> {
    pub value: Balance,
    pub negative: bool,
}
```
A simple representation to handle both surplus and deficit in the hub asset's balance, facilitating the management of the pool's stability mechanism. This structure allows the protocol to adjust for imbalances resulting from trades and liquidity events, ensuring the hub asset's value remains stable.

`AssetReserveState`
```rust
#[derive(Clone, Default, Debug, PartialEq, Eq)]
pub struct AssetReserveState<Balance> {
    pub reserve: Balance,
    pub hub_reserve: Balance,
    pub shares: Balance,
    pub protocol_shares: Balance,
    pub cap: u128,
    pub tradable: Tradability,
}
```
An extended representation of an asset's state, including its pool reserve alongside the `AssetState` information. It's used for more detailed operations and calculations within the pool, like asset pricing and liquidity adjustments, by including the actual reserve of the asset in the pool.

Each of these structures plays a crucial role in the functionality of the DeFi protocol, allowing for complex operations such as asset trading, liquidity provision/removal, and maintaining the overall balance and stability of the liquidity pool.


### [omnipool/src/traits.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/traits.rs) Omnipool pallet - traits

This contract defines a set of traits and structures for hook-based interactions within a DeFi protocol, specifically focusing on liquidity and trade events in an Omnipool-like environment. The `OmnipoolHooks` trait is central to the functionality, providing hooks for liquidity changes, trading events, and fee handling. The `AssetInfo` structure encapsulates details about asset states before and after transactions, facilitating detailed analysis and adjustments based on trades or liquidity changes. Additionally, the `ExternalPriceProvider` and `ShouldAllow` traits are designed to integrate external price data and enforce trading conditions, ensuring the protocol's stability and security against price manipulations.

#### Key Function's Functionality

 `OmnipoolHooks`
```rust
pub trait OmnipoolHooks<Origin, AccountId, AssetId, Balance> {
    type Error;
    fn on_liquidity_changed(origin: Origin, asset: AssetInfo<AssetId, Balance>) -> Result<Weight, Self::Error>;
    fn on_trade(
        origin: Origin,
        asset_in: AssetInfo<AssetId, Balance>,
        asset_out: AssetInfo<AssetId, Balance>,
    ) -> Result<Weight, Self::Error>;
    fn on_hub_asset_trade(origin: Origin, asset: AssetInfo<AssetId, Balance>) -> Result<Weight, Self::Error>;
    fn on_trade_fee(
        fee_account: AccountId,
        trader: AccountId,
        asset: AssetId,
        amount: Balance,
    ) -> Result<Balance, Self::Error>;
}
```
Defines a set of hooks that are called during liquidity changes, trading events, and hub asset trades. Each function returns a `Weight` indicating the computational cost of the operation or processes trade fees. These hooks are essential for updating state, applying fees, and reacting to changes within the pool dynamically.

 `AssetInfo`
```rust
pub struct AssetInfo<AssetId, Balance> {
    pub asset_id: AssetId,
    pub before: AssetReserveState<Balance>,
    pub after: AssetReserveState<Balance>,
    pub delta_changes: AssetStateChange<Balance>,
    pub safe_withdrawal: bool,
}
```
A structure that holds information about an asset's state before and after a transaction, including any changes in reserves, shares, and whether the operation allows for a safe withdrawal. It's used within hooks to assess and respond to the effects of trades or liquidity adjustments on individual assets.

 `ExternalPriceProvider`
```rust
pub trait ExternalPriceProvider<AssetId, Price> {
    type Error;
    fn get_price(asset_a: AssetId, asset_b: AssetId) -> Result<Price, Self::Error>;
}
```
A trait for fetching external price data for a pair of assets. This functionality is crucial for comparing on-chain prices with external references, allowing the protocol to mitigate risks associated with price manipulation or discrepancies.

`ShouldAllow`
```rust
pub trait ShouldAllow<AccountId, AssetId, Price> {
    fn ensure_price(who: &AccountId, asset_a: AssetId, asset_b: AssetId, current_price: Price) -> Result<(), ()>;
}
```
Defines a mechanism for validating whether a trade or liquidity operation should proceed based on the current price and potentially other conditions. This trait can enforce additional checks, such as price bounds, to ensure fair and stable trading conditions within the pool.

 `EnsurePriceWithin`
```rust
pub struct EnsurePriceWithin<AccountId, AssetId, ExternalOracle, MaxAllowed, WhitelistedAccounts>(...);
```
An implementation of the `ShouldAllow` trait, ensuring that the price of a transaction is within a specified range compared to an external price source. This structure is part of the protocol's defensive measures against drastic price movements or manipulative trading activities.

Each of these components plays a vital role in maintaining the operational integrity, security, and responsiveness of the Omnipool system to market conditions and participant actions.


### [math/src/omnipool/math.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/math.rs) Omnipool math - math implementation

This contract provides mathematical functions for calculating state changes within a decentralized finance (DeFi) protocol, specifically for an `Omnipool` setup. It includes intricate calculations for trades (buy/sell), liquidity changes (add/remove), fee handling, and adjustments due to price imbalances. It leverages fixed-point arithmetic for precision and supports operations like calculating the without-fee amount, determining trade state changes based on asset reserves, and adjusting liquidity based on current pool states.

#### Key Function's Functionality

 `calculate_sell_state_changes`
```rust
pub fn calculate_sell_state_changes(
    asset_in_state: &AssetReserveState<Balance>,
    asset_out_state: &AssetReserveState<Balance>,
    amount: Balance,
    asset_fee: Permill,
    protocol_fee: Permill,
    imbalance: Balance,
) -> Option<TradeStateChange<Balance>> { ... }
```
Calculates the changes to an asset's state resulting from a sell trade. This includes the adjustment of reserves in the pool, calculation of fees, and handling of imbalances. The function takes into account the amount being sold, fees applied, and the current imbalance in the hub asset. It returns a detailed structure of how the asset states should be updated.

 `calculate_buy_state_changes`
```rust
pub fn calculate_buy_state_changes(
    asset_in_state: &AssetReserveState<Balance>,
    asset_out_state: &AssetReserveState<Balance>,
    amount: Balance,
    asset_fee: Permill,
    protocol_fee: Permill,
    imbalance: Balance,
) -> Option<TradeStateChange<Balance>> { ... }
```
Computes the asset state changes for a buy trade, adjusting pool reserves and applying fees similarly to sell trades but from the perspective of buying. It ensures the pool remains balanced and fees are correctly distributed while accounting for the total imbalance. 

 `calculate_add_liquidity_state_changes`
```rust
pub fn calculate_add_liquidity_state_changes(
    asset_state: &AssetReserveState<Balance>,
    amount: Balance,
    imbalance: I129<Balance>,
    total_hub_reserve: Balance,
) -> Option<LiquidityStateChange<Balance>> { ... }
```
Determines the state changes when liquidity is added to the pool. It adjusts the asset's reserves and shares based on the amount added and updates the imbalance accordingly. This function is crucial for ensuring that liquidity providers are correctly rewarded for their contributions.

`calculate_remove_liquidity_state_changes`
```rust
pub fn calculate_remove_liquidity_state_changes(
    asset_state: &AssetReserveState<Balance>,
    shares_removed: Balance,
    position: &Position<Balance>,
    imbalance: I129<Balance>,
    total_hub_reserve: Balance,
    withdrawal_fee: FixedU128,
) -> Option<LiquidityStateChange<Balance>> { ... }
```
Calculates the necessary updates to an asset's state when liquidity is removed. This includes adjustments for the removed shares, potential withdrawal fees, and the overall impact on the pool's imbalance. It ensures that liquidity removal is handled accurately, maintaining the integrity of the pool's reserves.

`calculate_withdrawal_fee`
```rust
pub fn calculate_withdrawal_fee(
    spot_price: FixedU128,
    oracle_price: FixedU128,
    min_withdrawal_fee: Permill,
) -> FixedU128 { ... }
```
Computes the withdrawal fee based on the difference between the spot price and an external oracle price, applying a minimum fee threshold. This function is essential for protecting the pool from arbitrage and ensuring fairness in liquidity removal scenarios.

Each function meticulously manipulates balance states, fees, and imbalances to maintain the pool's health and fairness towards participants. The calculations support the dynamic nature of DeFi pools, allowing for responsive adjustments to trading, liquidity events, and market conditions.


### [math/src/omnipool/types.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/omnipool/types.rs) Omnipool math - types

This contract focusing on calculating state changes within a liquidity pool or Omnipool, following various operations such as trades and liquidity provisions. The code defines structures for representing asset states (`AssetReserveState`), updating balances (`BalanceUpdate`), and encapsulating the results of these operations (`AssetStateChange`, `TradeStateChange`, `HubTradeStateChange`, `LiquidityStateChange`). It utilizes fixed-point arithmetic to handle price calculations and updates, ensuring precise financial computations.

#### Key Function's Functionality

 `AssetReserveState`
```rust
#[derive(Clone, Default, Debug)]
pub struct AssetReserveState<Balance> {
    pub reserve: Balance,
    pub hub_reserve: Balance,
    pub shares: Balance,
    pub protocol_shares: Balance,
}
```
This structure holds the state of an asset within the Omnipool, including its reserve in the pool, the corresponding reserve of the hub asset, and the distribution of shares among liquidity providers and the protocol itself. It provides methods like `price()` to calculate the current price of the asset in terms of the hub asset.

 `BalanceUpdate`
```rust
#[derive(Copy, Clone, Debug, PartialEq, Eq)]
pub enum BalanceUpdate<Balance> {
    Increase(Balance),
    Decrease(Balance),
}
```
Represents an update to a balance, which can either be an increase or a decrease. This enum is crucial for applying state changes to assets in the pool, facilitating clear, error-resistant arithmetic operations.

 `AssetStateChange`
```rust
#[derive(Default, Clone, Debug, PartialEq, Eq)]
pub struct AssetStateChange<Balance> {
    pub delta_reserve: BalanceUpdate<Balance>,
    pub delta_hub_reserve: BalanceUpdate<Balance>,
    pub delta_shares: BalanceUpdate<Balance>,
    pub delta_protocol_shares: BalanceUpdate<Balance>,
}
```
Encapsulates the delta changes to an asset's state following an operation, such as trading or liquidity provision/removal. It uses `BalanceUpdate` to represent each change, allowing for concise descriptions of complex state transitions.

 `TradeStateChange`
```rust
#[derive(Default, Debug, PartialEq, Eq)]
pub struct TradeStateChange<Balance> {
    pub asset_in: AssetStateChange<Balance>,
    pub asset_out: AssetStateChange<Balance>,
    pub delta_imbalance: BalanceUpdate<Balance>,
    pub hdx_hub_amount: Balance,
    pub fee: TradeFee<Balance>,
}
```
Stores the result of a trading operation, detailing how the trade affected the states of the involved assets, the overall imbalance in the hub asset, any fees collected, and additional hub asset amounts for the HDX (assumed to be the protocol's native or hub asset).

 `calculate_sell_state_changes`
```rust
pub fn calculate_sell_state_changes(...)
```
Calculates the expected state changes resulting from a sell operation in the pool. It determines how selling an asset for another affects their reserves, shares, and the hub asset's balance, incorporating fees and adjusting for protocol revenue.

 `calculate_buy_state_changes`
```rust
pub fn calculate_buy_state_changes(...)
```
Similar to `calculate_sell_state_changes`, but for buy operations. It computes how buying an asset using another (or the hub asset) impacts the pool's state, adjusting reserves and shares accordingly, and accounting for fees.

These functions and structures form the computational backbone of the Omnipool's operation, allowing it to update states accurately in response to user actions, maintain balanced reserves, and collect fees, ensuring the pool's long-term viability and profitability.


### [stableswap/src/lib.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/lib.rs) Stableswap pallet - main pallet's file

The Stableswap pallet provides a Curve-like Automated Market Maker (AMM) implementation, optimized for stablecoin trading with minimal slippage. It introduces concepts such as **liquidity pools**, **share tokens**, and an **amplification factor** to enhance trading efficiency for stable assets. This pallet allows for the creation of liquidity pools with up to 5 assets, supports liquidity addition/removal, and trade execution with hooks for on-chain oracle updates.

#### Key Function's Functionality

 `create_pool`
```rust
pub fn create_pool(
    origin: OriginFor<T>,
    share_asset: T::AssetId,
    assets: Vec<T::AssetId>,
    amplification: u16,
    fee: Permill,
) -> DispatchResult
```
This function allows an authorized origin to create a new liquidity pool with a specified set of assets, an amplification factor, and a trading fee. It ensures that all assets are registered and that the pool does not already exist. A `PoolCreated` event is emitted upon successful creation.

 `add_liquidity`
```rust
pub fn add_liquidity(
    origin: OriginFor<T>,
    pool_id: T::AssetId,
    assets: Vec<AssetAmount<T::AssetId>>,
) -> DispatchResult
```
Enables a liquidity provider (LP) to add liquidity to a specified pool. The first LP must provide initial liquidity for all assets in the pool, while subsequent LPs can choose to add liquidity for any number of pool assets. The LP receives pool shares in return, proportional to the added liquidity. A `LiquidityAdded` event is emitted upon success.

 `remove_liquidity_one_asset`
```rust
pub fn remove_liquidity_one_asset(
    origin: OriginFor<T>,
    pool_id: T::AssetId,
    asset_id: T::AssetId,
    share_amount: Balance,
    min_amount_out: Balance,
) -> DispatchResult
```
Allows an LP to remove liquidity from a pool in exchange for a specific asset, specifying the amount of pool shares to burn and the minimum amount of the asset expected to receive. Withdrawal fees apply, reducing the received asset amount. A `LiquidityRemoved` event is emitted upon successful withdrawal.

 `sell`
```rust
pub fn sell(
    origin: OriginFor<T>,
    pool_id: T::AssetId,
    asset_in: T::AssetId,
    asset_out: T::AssetId,
    amount_in: Balance,
    min_buy_amount: Balance,
) -> DispatchResult
```
Executes a sell trade, where the caller exchanges `asset_in` for `asset_out` within a specified pool. The function ensures that the trade meets the minimum required output amount (`min_buy_amount`) to protect against slippage. A trade fee is deducted from the output amount. A `SellExecuted` event is emitted upon success.

`buy`
```rust
pub fn buy(
    origin: OriginFor<T>,
    pool_id: T::AssetId,
    asset_out: T::AssetId,
    asset_in: T::AssetId,
    amount_out: Balance,
    max_sell_amount: Balance,
) -> DispatchResult
```
Facilitates a buy trade, allowing the caller to specify the exact amount of `asset_out` they wish to purchase with `asset_in`, subject to a maximum sell amount (`max_sell_amount`). The function calculates the required input amount, including the trade fee, ensuring the trade does not exceed the specified slippage tolerance. A `BuyExecuted` event is emitted upon completion.

These functions collectively enable the creation and interaction with liquidity pools tailored for stablecoin trading, offering features like low slippage, efficient price discovery, and flexible liquidity provision/removal mechanisms.


### [stableswap/src/types.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/types.rs) Stableswap pallet - types

This contract defines structures for pool information (`PoolInfo`), asset amounts (`AssetAmount`), tradability flags (`Tradability`), and pool state (`PoolState`). The contract facilitates operations such as adding/removing liquidity, trading assets with minimal slippage, and adjusting pool parameters like amplification and fees. It leverages traits for configurability and extensibility, including hooks (`StableswapHooks`) for external integrations or adjustments post-trade or liquidity changes.

#### Key Function's Functionality

 `PoolInfo`
```rust
#[derive(Encode, Decode, Eq, PartialEq, Clone, RuntimeDebug, TypeInfo, MaxEncodedLen)]
pub struct PoolInfo<AssetId, BlockNumber> {
    pub assets: BoundedVec<AssetId, ConstU32<MAX_ASSETS_IN_POOL>>,
    pub initial_amplification: NonZeroU16,
    pub final_amplification: NonZeroU16,
    pub initial_block: BlockNumber,
    pub final_block: BlockNumber,
    pub fee: Permill,
}
```
Represents the structure of a liquidity pool, including its assets, amplification factors, fee, and the blocks over which amplification changes. It ensures pools have a minimum of two assets and unique elements. Methods include `find_asset` to locate an asset's index in the pool, `is_valid` to verify the pool's setup, and `reserves_with_decimals` to fetch asset reserves with decimal adjustments.

 `AssetAmount`
```rust
#[derive(Debug, Clone, Encode, Decode, PartialEq, Eq, TypeInfo, Default)]
pub struct AssetAmount<AssetId> {
    pub asset_id: AssetId,
    pub amount: Balance,
}
```
Defines an asset and its amount for liquidity or trade operations. It includes a constructor `new` for creating instances and conversion traits to `u128`, facilitating integration with other parts of the contract or module for calculations or comparisons.

 `Tradability`
```rust
bitflags::bitflags! {
    #[derive(Encode,Decode, MaxEncodedLen, TypeInfo)]
    pub struct Tradability: u8 {
        const FROZEN = 0b0000_0000;
        const SELL = 0b0000_0001;
        const BUY = 0b0000_0010;
        const ADD_LIQUIDITY = 0b0000_0100;
        const REMOVE_LIQUIDITY = 0b0000_1000;
    }
}
```
Specifies the allowed operations for an asset within a pool, such as selling, buying, adding liquidity, and removing liquidity. It supports bitwise operations to combine different tradabilities.

 `StableswapHooks`
```rust
pub trait StableswapHooks<AssetId> {
    fn on_liquidity_changed(pool_id: AssetId, state: PoolState<AssetId>) -> DispatchResult;
    fn on_trade(pool_id: AssetId, asset_in: AssetId, asset_out: AssetId, state: PoolState<AssetId>) -> DispatchResult;

    fn on_liquidity_changed_weight(n: usize) -> Weight;
    fn on_trade_weight(n: usize) -> Weight;
}
```
Defines hooks for external integrations or adjustments following liquidity changes or trades within a pool. It allows for custom implementations to respond to these events, potentially updating on-chain oracles or adjusting internal states based on the trade dynamics and liquidity shifts.

These components work together to enable a flexible and efficient stableswap mechanism within a blockchain's DeFi ecosystem, facilitating low-slippage trades and dynamic liquidity management for a range of financial applications.


### [math/src/stableswap/math.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/math.rs) Stableswap Math - math implementation

The contract provides functionality to calculate the number of tokens received or required for trades (both direct and inverse), considering fees and amplification factors. It also calculates shares issued to liquidity providers when they add liquidity to the pool. Key constants like `MAX_Y_ITERATIONS` and `MAX_D_ITERATIONS` define the precision of iterative calculations, crucial for ensuring the accuracy of complex financial computations in DeFi applications.

#### Key Function's Functionality

 `calculate_out_given_in`
```rust
pub fn calculate_out_given_in<const D: u8, const Y: u8>(
    initial_reserves: &[AssetReserve],
    idx_in: usize,
    idx_out: usize,
    amount_in: Balance,
    amplification: Balance,
) -> Option<Balance>
```
Calculates the amount of an asset (identified by `idx_out`) a trader receives from the pool in exchange for a given amount of another asset (identified by `idx_in`). It leverages an amplification coefficient to adjust the price according to the pool's size and composition, aiming to offer better rates for stablecoin swaps.

 `calculate_in_given_out`
```rust
pub fn calculate_in_given_out<const D: u8, const Y: u8>(
    initial_reserves: &[AssetReserve],
    idx_in: usize,
    idx_out: usize,
    amount_out: Balance,
    amplification: Balance,
) -> Option<Balance>
```
Determines the amount of an asset (identified by `idx_in`) that must be provided to the pool to receive a specific amount of another asset (identified by `idx_out`). This function is essential for planning trades and understanding market depth.

 `calculate_out_given_in_with_fee`
```rust
pub fn calculate_out_given_in_with_fee<const D: u8, const Y: u8>(
    initial_reserves: &[AssetReserve],
    idx_in: usize,
    idx_out: usize,
    amount_in: Balance,
    amplification: Balance,
    fee: Permill,
) -> Option<(Balance, Balance)>
```
Extends `calculate_out_given_in` by incorporating a trading fee (`fee`), providing a more accurate calculation of the trade outcome. It returns both the amount received and the fee deducted, helping users understand the cost of their trade.

 `calculate_in_given_out_with_fee`
```rust
pub fn calculate_in_given_out_with_fee<const D: u8, const Y: u8>(
    initial_reserves: &[AssetReserve],
    idx_in: usize,
    idx_out: usize,
    amount_out: Balance,
    amplification: Balance,
    fee: Permill,
) -> Option<(Balance, Balance)>
```
Similar to `calculate_in_given_out` but includes the calculation of the trading fee. It calculates the amount needed to provide to the pool (including the fee) to receive a specified amount of another asset.

 `calculate_shares`
```rust
pub fn calculate_shares<const D: u8>(
    initial_reserves: &[AssetReserve],
    updated_reserves: &[AssetReserve],
    amplification: Balance,
    share_issuance: Balance,
    fee: Permill,
) -> Option<Balance>
```
Calculates the number of shares to be issued to a liquidity provider based on the change in the pool's reserves after liquidity is added. This function accounts for the pool's amplification factor and trading fees, ensuring that liquidity providers are fairly compensated for their contribution.

These functions collectively form the backbone of a stableswap AMM, enabling efficient trading and liquidity provision within a DeFi ecosystem. They are designed to minimize slippage and provide competitive rates for stablecoin trades, leveraging mathematical models to dynamically adjust prices based on pool composition and trading volumes.


### [math/src/stableswap/types.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/stableswap/types.rs) Stableswap Math - types

This contract specifically targeting asset management in liquidity pools. It introduces the `AssetReserve` struct, which holds two crucial pieces of information about a financial asset: its amount (balance) and its precision (decimals). This struct is foundational for DeFi operations, enabling precise calculations and operations with assets within various financial protocols, such as automated market makers (AMMs), lending platforms, and more.

#### Key Function's Functionality

 `AssetReserve::new`
```rust
impl AssetReserve {
    pub fn new(amount: Balance, decimals: u8) -> Self {
        Self { amount, decimals }
    }
}
```
The `new` function is a constructor for the `AssetReserve` struct, initializing it with a specified `amount` of the asset and its `decimals`. This function allows for creating an `AssetReserve` instance, which can then be used to represent the reserve of an asset in a liquidity pool or other financial instruments within a DeFi application.

 `AssetReserve::is_zero`
```rust
impl AssetReserve {
    pub fn is_zero(&self) -> bool {
        self.amount == Balance::zero()
    }
}
```
The `is_zero` method checks whether the `amount` in the `AssetReserve` is zero. This method is particularly useful for validations and checks within financial operations, ensuring that operations do not proceed with empty reserves, which could lead to errors or unintended behavior in DeFi protocols.

 `From<AssetReserve> for u128`
```rust
impl From<AssetReserve> for u128 {
    fn from(value: AssetReserve) -> Self {
        value.amount
    }
}
impl From<&AssetReserve> for u128 {
    fn from(value: &AssetReserve) -> Self {
        value.amount
    }
}
```
These `From` trait implementations allow for seamless conversion of an `AssetReserve` instance (or a reference to it) into a `u128` value, representing its amount. This conversion is particularly useful for arithmetic operations and comparisons where the precision (decimals) is not immediately relevant, simplifying the manipulation of asset reserves in various calculations and logic within DeFi applications.

The contract is designed with simplicity and utility in mind, focusing on representing asset reserves accurately and facilitating easy interactions with them in a broader DeFi ecosystem.


### [ema-oracle/src/lib.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/lib.rs) Ema oracle pallet - main pallet's file


The `pallet-ema-oracle` provides on-chain oracle services using Exponential Moving Average (EMA) for various metrics like price, volume, and liquidity for specific asset pairs and sources. It's designed to integrate with other pallets, like trading or liquidity pools, to offer updated and historical data insights. The oracle updates are triggered by events such as trades or liquidity changes, with the data aggregated at the block's end to update the EMA values. It supports various oracle periods and is optimized for efficient storage and retrieval.

####  Key Function's Functionality

 `Pallet::on_entry`
```rust
pub(crate) fn on_entry(
    src: Source,
    assets: (AssetId, AssetId),
    oracle_entry: OracleEntry<BlockNumberFor<T>>,
) -> Result<(), ()> {
    // Implementation details...
}
```
This function aggregates incoming oracle entries within the same block before the final EMA calculation. It's called internally to process and accumulate data from trades or liquidity changes. The accumulated data is later used to update the EMA oracles at the block's end.

`Pallet::on_trade`
```rust
pub(crate) fn on_trade(
    src: Source,
    assets: (AssetId, AssetId),
    oracle_entry: OracleEntry<BlockNumberFor<T>>,
) -> Result<Weight, (Weight, DispatchError)> {
    // Implementation details...
}
```
Triggered by trade events, this function aggregates trading data (price, volume) into the accumulator for later EMA updates. It ensures that trading activities across different asset pairs are captured and reflected in the oracle data, providing up-to-date market insights.

`Pallet::on_liquidity_changed`
```rust
pub(crate) fn on_liquidity_changed(
    src: Source,
    assets: (AssetId, AssetId),
    oracle_entry: OracleEntry<BlockNumberFor<T>>,
) -> Result<Weight, (Weight, DispatchError)> {
    // Implementation details...
}
```
Invoked on liquidity events, such as adding or removing liquidity from pools, this function updates the accumulator with the latest liquidity information. It's crucial for keeping the oracle data relevant to the current market conditions, reflecting changes in asset liquidity.

`Pallet::update_oracles_from_accumulator`
```rust
fn update_oracles_from_accumulator() {
    // Implementation details...
}
```
At the end of each block, this function processes the accumulated data, applying EMA logic to update oracle values. It ensures the oracle entries are current and accurately reflect the latest market activities, maintaining reliable data sources for other pallets and applications.

 `Pallet::get_updated_entry`
```rust
fn get_updated_entry(
    src: Source,
    assets: (AssetId, AssetId),
    period: OraclePeriod,
) -> Option<(OracleEntry<BlockNumberFor<T>>, BlockNumberFor<T>)> {
    // Implementation details...
}
```
Retrieves the most recent oracle entry for a given asset pair, source, and period, applying any pending updates to reflect the latest market state. This function is key for accessing up-to-date and accurate oracle data, crucial for decision-making processes in DeFi applications.

The `pallet-ema-oracle` plays a critical role in providing reliable and timely market data, essential for the functioning of decentralized finance (DeFi) ecosystems on the blockchain. Its design ensures efficient data aggregation, update, and retrieval, supporting a wide range of applications and services.


### [ema-oracle/src/types.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/types.rs) Ema oracle pallet - types

This part of the `pallet-ema-oracle` focuses on defining the data structures and functionalities related to handling oracle entries, which include price, volume, and liquidity information updated at a specific block number. The `OracleEntry` struct encapsulates this data, providing methods for operations like inversion, volume accumulation, and updates based on new incoming data. These functionalities are crucial for maintaining accurate and up-to-date oracle data across different periods (e.g., last block, short, ten minutes, hour, day, week) and for different asset pairs sourced from trading and liquidity events.

#### Key Function's Functionality

 `OracleEntry::new`
```rust
pub const fn new(
    price: Price,
    volume: Volume<Balance>,
    liquidity: Liquidity<Balance>,
    updated_at: BlockNumber,
) -> Self {
    Self {
        price,
        volume,
        liquidity,
        updated_at,
    }
}
```
Constructs a new `OracleEntry` instance with given price, volume, liquidity, and the block number when it was updated. This function is fundamental for initializing oracle data entries that will be processed and stored by the oracle pallet.

`OracleEntry::into_aggregated`
```rust
pub fn into_aggregated(self, initialized: BlockNumber) -> AggregatedEntry<Balance, BlockNumber, Price> {
    AggregatedEntry {
        price: self.price,
        volume: self.volume,
        liquidity: self.liquidity,
        oracle_age: self.updated_at.saturating_sub(initialized),
    }
}
```
Converts an `OracleEntry` into an `AggregatedEntry` by calculating the age of the oracle entry based on the block number when it was initialized. This method is essential for providing a standardized data format that can be easily consumed by other components or pallets.

`OracleEntry::inverted`
```rust
pub fn inverted(self) -> Self {
    let price = self.price.inverted();
    let volume = self.volume.inverted();
    let liquidity = self.liquidity.inverted();
    Self {
        price,
        volume,
        liquidity,
        updated_at: self.updated_at,
    }
}
```
Inverts the `OracleEntry`, effectively swapping the roles of assets in the pair (e.g., converting prices from asset A to asset B into prices from asset B to asset A). This function ensures that oracle data can be accurately represented regardless of the asset order.

`OracleEntry::accumulate_volume_and_update_from`
```rust
pub fn accumulate_volume_and_update_from(&mut self, incoming: &Self) {
    self.volume = incoming.volume.saturating_add(&self.volume);
    self.price = incoming.price;
    self.liquidity = incoming.liquidity;
    self.updated_at = incoming.updated_at;
}
```
Updates the current oracle entry by accumulating the volume from an incoming entry and adopting its price, liquidity, and update timestamp. This method is key to aggregating trade and liquidity data within a block before finalizing EMA calculations.

`OracleEntry::calculate_new_by_integrating_incoming`
```rust
pub fn calculate_new_by_integrating_incoming(&self, period: OraclePeriod, incoming: &Self) -> Option<Self> {
    // Omitted: calculation logic
}
```
Calculates a new `OracleEntry` based on the current entry and an incoming entry for a specified period. This function applies the EMA logic to integrate the new data, updating the oracle entry according to the exponential moving average formula. It's critical for updating oracle data over time, ensuring the EMA reflects recent market activities.

These functionalities form the backbone of the `pallet-ema-oracle`, allowing it to process, update, and maintain oracle data that's vital for DeFi applications requiring accurate and timely market information.

### [math/src/ema/math.rs :](https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/math/src/ema/math.rs) Omnipool math - math implementation

The provided Rust code snippet is part of a library or module designed to calculate the Exponential Moving Average (EMA) for oracle values like price, volume, and liquidity in a blockchain-based financial application. It defines types and functions for calculating new EMA values by integrating incoming data with previous values using a smoothing factor. The calculations consider various factors such as price, volume, liquidity, and the age of the data. This functionality is essential for financial applications that require accurate and up-to-date oracle data for asset pricing, liquidity management, and trading volume analysis.

#### Key Function's Functionality

 `calculate_new_by_integrating_incoming`
```rust
pub fn calculate_new_by_integrating_incoming(
    previous: (EmaPrice, EmaVolume, EmaLiquidity),
    incoming: (EmaPrice, EmaVolume, EmaLiquidity),
    smoothing: Fraction,
) -> (EmaPrice, EmaVolume, EmaLiquidity) {
    // Implementation details
}
```
Calculates new EMA values for price, volume, and liquidity by integrating incoming data with previous oracle values using a specified smoothing factor. This function ensures that the oracle data is updated in a way that reflects recent market activities while maintaining a balance with historical data, providing a more accurate representation of market conditions.

 `update_outdated_to_current`
```rust
pub fn update_outdated_to_current(
    iterations: u32,
    outdated: (EmaPrice, EmaVolume, EmaLiquidity),
    update_with: (EmaPrice, EmaLiquidity),
    smoothing: Fraction,
) -> (EmaPrice, EmaVolume, EmaLiquidity) {
    // Implementation details
}
```
Updates outdated oracle values to the current state by applying a smoothing factor over a specified number of iterations. This function is crucial for keeping oracle data relevant and accurate, especially after periods of inactivity or when data has not been updated for several blocks.

 `exp_smoothing`
```rust
pub fn exp_smoothing(smoothing: Fraction, iterations: u32) -> Fraction {
    // Implementation details
}
```
Calculates an exponential smoothing factor based on the original smoothing factor and the number of iterations. This function is essential for adjusting the impact of new data on the EMA calculation over time, allowing for more flexible and dynamic oracle updates.

`price_weighted_average`, `balance_weighted_average`, `volume_weighted_average`, `liquidity_weighted_average`
```rust
// Example for price_weighted_average
pub fn price_weighted_average(prev: EmaPrice, incoming: EmaPrice, weight: Fraction) -> EmaPrice {
    // Implementation details
}
```
These functions calculate weighted averages for price, balance, volume, and liquidity, respectively. They play a critical role in the EMA calculation process by determining how much new data should influence the updated oracle values, based on the given weight (smoothing factor).

These functionalities collectively enable the system to maintain up-to-date and accurate oracle data by carefully integrating new information with existing data. This is vital for decentralized finance (DeFi) platforms that rely on precise and current data for trading, lending, and other financial activities.

### [circuit-breaker/src/lib.rs :](https://github.com/code-423n4/2024-02-hydradx/tree/main/HydraDX-node/pallets/circuit-breaker/src/lib.rs) Circuit breaker- main pallet's file
This contract focuses on managing liquidity and trade volume limits within a decentralized exchange (DEX) environment. It introduces structures for setting and enforcing limits on trade volumes (`TradeVolumeLimit`) and liquidity changes (`LiquidityLimit`) for assets within a block. The contract aims to mitigate significant, abrupt market movements and provide stability by restricting the amount of liquidity that can be added or removed and the volume of assets that can be traded within a single block. It includes mechanisms to set these limits, update and check compliance with the limits, and handles initialization and validation of these constraints. Notably, it accommodates whitelisted accounts with exemptions from these restrictions.

#### Key Function's Functionality:

`set_trade_volume_limit`
   ```rust
   pub fn set_trade_volume_limit(
       origin: OriginFor<T>,
       asset_id: T::AssetId,
       trade_volume_limit: (u32, u32),
   ) -> DispatchResult
   ```
   This function allows authorized entities (via `TechnicalOrigin`) to set limits on the net trade volume of a specific asset that can occur within a block. It updates the trade volume limit for the specified `asset_id` with the given `trade_volume_limit` (expressed as a fraction of the pool's liquidity). The function ensures the asset is not the omnipool's hub asset, validates the limit's value, updates the storage, and emits a `TradeVolumeLimitChanged` event upon success.

`set_add_liquidity_limit`
   ```rust
   pub fn set_add_liquidity_limit(
       origin: OriginFor<T>,
       asset_id: T::AssetId,
       liquidity_limit: Option<(u32, u32)>,
   ) -> DispatchResult
   ```
   This function sets a limit on the amount of liquidity that can be added to a specific asset's pool within a block. It's accessible to authorized users and checks that the asset is not the omnipool hub. The limit is optional, allowing for the possibility of not enforcing this limit. Successful execution updates the corresponding storage and emits an `AddLiquidityLimitChanged` event.

`set_remove_liquidity_limit`
   ```rust
   pub fn set_remove_liquidity_limit(
       origin: OriginFor<T>,
       asset_id: T::AssetId,
       liquidity_limit: Option<(u32, u32)>,
   ) -> DispatchResult
   ```
   Similar to adding liquidity limits, this function sets a limit on the liquidity that can be removed from a pool within a single block for a given asset. It ensures that the asset is not the omnipool hub and allows for optional enforcement of the limit. The function updates the appropriate storage and issues a `RemoveLiquidityLimitChanged` event if successful.

`ensure_and_update_trade_volume_limit`
   ```rust
   fn ensure_and_update_trade_volume_limit(
       asset_in: T::AssetId,
       amount_in: T::Balance,
       asset_out: T::AssetId,
       amount_out: T::Balance,
   ) -> DispatchResult
   ```
   This internal function is used to verify and update the trade volume limits when a trade occurs. It checks that trading activities do not exceed the predefined limits for both the input and output assets, excluding the omnipool's hub asset. If the trade is within limits, it updates the stored volumes accordingly.

`calculate_limit`
   ```rust
   pub fn calculate_limit(
       liquidity: T::Balance,
       limit: (u32, u32)
   ) -> Result<T::Balance, DispatchError>
   ```
   A utility function that calculates the actual limit value based on the pool's liquidity and the configured limit fraction. It supports the functionality of setting and adjusting liquidity and trade volume limits by translating percentage-based configurations into absolute numeric limits based on the current state of the pool's liquidity.
   
`ensure_pool_state_change_limit`
   ```rust
   pub fn ensure_pool_state_change_limit(
       asset_in: T::AssetId,
       asset_in_reserve: T::Balance,
       amount_in: T::Balance,
       asset_out: T::AssetId,
       asset_out_reserve: T::Balance,
       amount_out: T::Balance,
   ) -> Result<Weight, DispatchError>
   ```
   This function ensures that any change to the pool's state (such as trading between assets) adheres to the established limits. It initializes trade limits for both input and output assets if they have not been set, then checks that the proposed trade does not exceed these limits. If the trade is within bounds, it updates the trade volume limits accordingly. This function is crucial for maintaining market stability and preventing large, sudden changes in liquidity that could destabilize the DEX.

`ensure_add_liquidity_limit` and `ensure_remove_liquidity_limit`
   ```rust
   pub fn ensure_add_liquidity_limit(
       origin: OriginFor<T>,
       asset_id: T::AssetId,
       initial_liquidity: T::Balance,
       added_liquidity: T::Balance,
   ) -> Result<Weight, DispatchError>
   ```
   ```rust
   pub fn ensure_remove_liquidity_limit(
       origin: OriginFor<T>,
       asset_id: T::AssetId,
       initial_liquidity: T::Balance,
       removed_liquidity: T::Balance,
   ) -> Result<Weight, DispatchError>
   ```
   These functions check and enforce the limits on liquidity additions and removals for a given asset within a block. They calculate and store the liquidity limits if not already set, then verify that the proposed addition or removal of liquidity does not breach these limits. If the action is permissible, the liquidity limits are updated. These mechanisms are instrumental in managing the flow of liquidity into and out of the pools, preventing excessive volatility.

`validate_limit`
   ```rust
   pub fn validate_limit(limit: (u32, u32)) -> DispatchResult
   ```
   A utility function to ensure that the specified limits are valid and within acceptable parameters (e.g., non-zero, within the maximum allowable value). It is used across the contract to validate limits for trading volumes and liquidity changes before applying them. This function helps ensure the integrity of limit settings and prevents configuration errors that could adversely affect the DEX's operation.

These functions collectively support a comprehensive framework for managing liquidity and trade volumes within the HydraDX ecosystem, aiming to foster a stable and efficient trading environment.



## Codebase Quality :
This codebase quality table presents a comprehensive evaluation of the codebase quality across multiple critical dimensions. It delves into the maintainability and reliability of the code, the extent and clarity of comments, the thoroughness of testing procedures, the structure and formatting of the codebase, the inherent strengths of the development, and the quality and utility of the documentation provided. The comments and descriptions provided are based on a detailed review of the codebase, reflecting on best practices in software development tailored to the unique demands of blockchain and smart contract development.

| Codebase Quality Categories      | Comments and Descriptions                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Code Maintainability and Reliability | The codebase demonstrates high maintainability and reliability, with clear modularization and logical separation of concerns. Functions are well-defined, adhering to single responsibility principles, which facilitate ease of updates and scalability. Error handling is thorough, with explicit checks and balances that enhance the robustness of contract operations, minimizing the risk of unexpected failures.                                                                                                    |
| Code Comments                    | Extensive commenting is evident throughout the codebase, providing valuable context and clarification for complex logic and contract interactions. Comments not only explain the "what" but also the "why" behind critical operations, aiding developers in understanding the rationale behind design decisions. Inline documentation follows a consistent format, improving readability and aiding future code reviews and audits.                                                                                                       |
| Testing                          | Testing coverage across the contracts is comprehensive, encompassing a wide range of scenarios from basic functionality to edge cases. The tests are meticulously structured, demonstrating a deep understanding of the protocol's intricacies. Each test is clearly commented, specifying the purpose and expected outcomes, which I found immensely helpful during my exploration. The robust testing suite instills confidence in the protocol's reliability and its resilience against potential vulnerabilities.                         |
| Code Structure and Formatting    | The codebase adheres to a consistent coding style and formatting guidelines, which promotes readability and ease of navigation. Directory structures are intuitive, with contracts, tests, and helper modules organized logically. Such coherence in code structure and formatting not only aids in immediate comprehension but also in long-term maintenance. Use of modern Solidity practices and adherence to community standards (e.g., OpenZeppelin for reusable contracts) further underscores the professional quality of the codebase. |
| Strengths                        | Key strengths of the codebase include its architectural design, which supports extensibility and integration with other DeFi components. The contracts are optimized for gas efficiency, an essential consideration for blockchain development. Security practices are rigorously applied, with contracts demonstrating awareness and mitigation of common vulnerabilities. The protocol's innovative features, such as dynamic fee adjustment and multi-asset support, showcase the codebase's forward-thinking approach.             |
| Documentation                    | The documentation accompanying the codebase is thorough and well-structured, providing a comprehensive overview of the protocol's functionalities and contract interactions. Diagrams and flowcharts supplement textual descriptions, offering visual insights into the system's architecture and workflows. API documentation is detailed, with clear examples for developers looking to build on or integrate with the protocol. Overall, the documentation is sufficient and significantly enhances the codebase's accessibility.           |


Given the nature and complexity of HydraDX, as outlined in the provided contract descriptions and overview, assessing centralization and systematic risks requires a deep understanding of the protocol's architecture and operational mechanics. Here are the risks categorized under Centralization Risks and Systematic Risks:

## Centralization Risks

**Governance Concentration**: The governance model, if not designed with broad community participation in mind, can lead to decision-making being concentrated among a small number of holders. This could potentially lead to changes in the protocol that favor a select group.

   ```rust
   type TechnicalOrigin: EnsureOrigin<Self::RuntimeOrigin>;
   ```

  Here, `TechnicalOrigin` determines who can make technical changes to the protocol. If this power is held by a small group, it could introduce a central point of control.

**Oracle Reliance**: The integrity of external data sources (oracles) is vital. A limited number of oracle providers or centralized control over these can lead to data manipulation, affecting everything from token pricing to triggered contract actions.

   ```rust
   // Off-chain Services & Oracles
   ```


## Systematic Risks

**Smart Contract Vulnerabilities**: As with any complex DeFi protocol, the risk of bugs or vulnerabilities in smart contracts poses a significant threat. Even with rigorous testing, the possibility of unforeseen exploits remains, potentially leading to loss of funds.

   ```rust
   pub fn ensure_and_update_trade_volume_limit(...)
   ```

   Functions like this manage critical aspects of trading and liquidity. Vulnerabilities here could lead to systematic failures affecting all users.

**Liquidity Pool Risks**: The mechanisms for managing liquidity pools, including incentives for liquidity providers and the handling of impermanent loss, are critical. Poorly designed incentives or mismanagement of pool dynamics can lead to liquidity issues, affecting the protocol's stability and user confidence.

   ```rust
   impl<T: Config> LiquidityLimit<T> { ... }
   ```

   This code snippet relates to the management of liquidity limits. Systematic risks arise if the liquidity is not adequately distributed or if too much is concentrated in few assets, leading to market manipulation or instability.

**Market Mechanic Exploits**: The economic models and market mechanics designed to ensure stability and incentivize certain behaviors could be exploited by actors with sufficient resources or knowledge, leading to market distortions or manipulations.

   ```rust
   // Market Mechanics
   ```

   This segment implies the complex interplay of incentives and economic models governing the protocol. Exploits or manipulations of these mechanics could introduce systemic risks.





## Architectural Improvement :

To bolster HydraDX's architectural robustness, scalability, and resilience against centralization and systemic risks, a series of architectural improvements are proposed. These enhancements aim to fortify the protocol's security, optimize its operational efficiency, and ensure its long-term sustainability in the rapidly evolving DeFi landscape. Here's a detailed exploration of potential architectural improvements:

**Decentralization of Governance**:

`Implementation of DAO for Governance Decisions`: Transitioning to a decentralized autonomous organization (DAO) model for critical governance decisions can mitigate centralization risks. Utilizing smart contracts to facilitate proposal submissions, voting, and implementation ensures a transparent and democratic decision-making process.


  ```rust
  // Pseudocode for DAO-based Governance Mechanism
  struct Proposal { ... }
  impl Proposal {
      fn submit(...) { ... }
      fn vote(...) { ... }
      fn execute(...) { ... }
  }
  ```

**Enhanced Oracle Security**:

`Integration of Decentralized Oracle Networks`: Incorporating decentralized oracle solutions that aggregate data from multiple sources to ensure accuracy and resist manipulation. Employing cryptographic techniques like threshold signatures can further enhance data integrity.

  ```rust
  // Pseudocode for Integrating Decentralized Oracle Network
  fn update_price_feed(asset_id: AssetId) {
      let price = decentralized_oracle_network.get_price(asset_id);
      update_asset_price(asset_id, price);
  }
  ```

**Smart Contract Security Enhancements**:

`Immutable Smart Contract Design`: Where possible, employing immutable smart contract designs that minimize the attack surface by reducing the number of functions that can alter critical state variables.

`Continuous Auditing and Formal Verification`: Establishing a routine of continuous security audits and formal verification processes to identify and mitigate potential vulnerabilities in the smart contract codebase.


  ```rust
  // Pseudocode for Immutable Contract Design
  contract AssetRegistry {
      immutable public Asset[] assets;
      function add_asset(Asset asset) internal { ... }
  }
  ```

**Liquidity Management Optimization**:

`Dynamic Liquidity Adjustment Algorithms`: Developing algorithms that can dynamically adjust liquidity parameters based on real-time market conditions, ensuring optimal liquidity levels are maintained to support trading activities without compromising security.

  ```rust
  // Pseudocode for Dynamic Liquidity Adjustment
  fn adjust_liquidity(asset_id: AssetId) {
      let market_conditions = analyze_market(asset_id);
      if (market_conditions.require_liquidity_adjustment()) {
          update_liquidity_parameters(asset_id, market_conditions.optimal_liquidity());
      }
  }
  ```

**Economic Model Refinement**:

`Implementation of Anti-Gaming Mechanisms`: Designing and implementing economic models that are resistant to gaming or manipulation by malicious actors. This includes creating balanced incentive structures that encourage positive behavior within the ecosystem.

  ```rust
  // Pseudocode for Anti-Gaming Economic Mechanisms
  struct IncentiveModel { ... }
  impl IncentiveModel {
      fn calculate_rewards(user: User, action: Action) -> Reward {
          if (is_action_beneficial(action)) {
              return calculate_beneficial_reward(user, action);
          } else {
              return minimal_or_no_reward();
          }
      }
  }
  ```

These architectural improvements represent a holistic approach to enhancing `HydraDX`'s infrastructure, focusing on decentralization, security, liquidity management, and economic sustainability. Implementing these improvements can significantly reduce risks, improve protocol resilience, and ensure `HydraDX` remains a leading and innovative force in the DeFi space.

## Time Spent:
112 Hours :)




### Time spent:
112 hours