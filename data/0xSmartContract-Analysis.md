# 🛠️ Analysis - Hydra DX Audit 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|a) | Analysis of the code base  |  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports  | What are the previous Audit reports and their analysis |
|f) |Centralization Risks |  |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2024-02-hydradx?tab=readme-ov-file

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-02-hydradx?tab=readme-ov-file#running-pallet-tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Hydra DX](https://docs.hydradx.io/) |Provides a basic architectural teaching for General Architecture|
|3|Test Suits|[Tests](https://github.com/code-423n4/2024-02-hydradx?tab=readme-ov-file#running-integration-tests)|In this section, the scope and content of the tests of the project are analyzed.|
|4|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-02-hydradx?tab=readme-ov-file#scope)|
|5|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|6|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-02-hydradx?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs)||

## b) Analysis of the code base


</br>
</br>

###  Entry Point : HydraDX Omnipool pallet

- The Omnipool palette is the main palette/module in this project. It is an omnipool type AMM, meaning all assets are pooled in a single pool.
 
- Liquidity providers can add any asset to the pool and receive pool shares in return. The position is represented as NFT token.

- Each asset can be sold or bought in the pool to facilitate trading.

- The imbalance mechanism is used to balance the LRNA value.

- Attachment points for pallet, fluid control etc. features are available.

- The project was written in Rust and developed on the substrate framework.

- Project information and dependencies are defined in the Cargo.toml file. The project was developed for benchmarking with rust features.

###  Provider.rs:

Functions :

- **pair_exists**: This function checks whether two specific asset pairs (asset_a and asset_b) exist in the system. Queries whether a record exists in the `Assets` storage for both assets. Returns true if both entities exist; otherwise returns false.

- **spot_price**: This function calculates the spot price between two assets. The function handles different scenarios:

   - If `asset_a` and `asset_b` are the same, it directly returns the price of 1 (FixedU128::one()) because the price of an asset with itself is always 1.
  
   - If `asset_a` is a hub asset (identified by T::HubAssetId), it calculates a price based on `asset_b`'s hub reserve and its own reserve. This represents the ratio of the `asset_b` asset to the hub asset.
  
   - If `asset_b` is a hub asset, it calculates a price based on `asset_a`'s own reserve and the hub reserve. This represents the ratio of the `asset_a` asset to the hub asset.
  
   - If neither are hub assets, it calculates the prices for both assets and multiplies them to find the ratio of `asset_a` to `asset_b`. This is an indirect ratio calculation and allows two assets to be compared through the hub asset.


This smart contract checks whether Asset pairs exist and calculates instant prices between the two assets. Calculations are performed on the hub entity, which determines how different entities relate to each other and to a common reference point.

![image](https://github.com/code-423n4/2024-02-hydradx/assets/104318932/043bff6b-d0ff-49fc-a705-69ed3d2cec72)

##### Note: Please click on the image to enlarge it:

</br>
</br>

###  router_execution.rs:

Functions :

**calculate_sell and calculate_buy**:
- These functions calculate how much assets users can receive or give when they want to buy or sell a certain amount of assets.
- `calculate_sell` calculates how much asset the user will receive when selling a particular asset. `calculate_buy` calculates how much the user must pay to buy a particular asset.
- Both functions check whether transactions can be made through the hub entity and give special treatment to such transactions.
- In both cases, the fees to be applied are calculated and the final cost or return is calculated based on the changes applied.

**execute_sell and execute_buy**:
- These functions execute a sale or purchase transaction under the specified conditions.
- The user can set minimum or maximum limits, which gives the user protection in bad market conditions.
- Depending on the transaction type, after the necessary checks are made, the sale or purchase transaction is carried out.

**get_liquidity_depth**:
- This function returns the depth of liquidity for a given asset. Liquidity depth represents the amount of assets available in a pool.
- This allows users to evaluate the size of the liquidity pool and potential slippage.



This smart contract forms the basis of trading on HydraDX Omnipool. It allows users to sell and buy their assets and control the depth of liquidity.

![image](https://github.com/code-423n4/2024-02-hydradx/assets/104318932/1b717ae5-3d77-4c10-a1df-18947f84c2b9)

##### Note: Please click on the image to enlarge it:

</br>
</br>

###  traits.rs:

This Rust module defines hooks and price control mechanisms for HydraDX Omnipool that are triggered in response to events such as liquidity changes, trade transactions, and price updates.

### AssetInfo Structure:
- This structure holds asset information: `asset_id`, asset status in the liquidity pool before and after (`after`), changes made (`delta_changes`) and whether safe withdrawals can be made (`safe_withdrawal`).

### OmnipoolHooks Trait:
- `on_liquidity_changed`: Called when there is a liquidity change. Takes an `AssetInfo` object and returns the weight of the transaction (`Weight`).
- `on_trade`: Called when a trade transaction occurs. Retrieves two `AssetInfo` objects containing information about incoming and outgoing assets.
- `on_hub_asset_trade`: Called in trade transactions with the Hub asset.
- `on_trade_fee`: Calculates the fee applied during the trade transaction.

### ExternalPriceProvider Trait:
- It is used to obtain price information from an external price provider. The `get_price` function returns the current price between two assets.

### ShouldAllow Trait:
- Contains functions that control whether price changes will be accepted or not.
- `ensure_price`: Controls whether a user will accept a certain price for a given asset pair.

### EnsurePriceWithin Structure:
- It ensures that the current spot price and the external oracle price are within a certain percentage, as long as it does not exceed the specified maximum acceptable price difference.
- `WhitelistedAccounts`: Provides exemption from price control for certain accounts.

### General Summary:
This module allows HydraDX Omnipool to respond to various events such as liquidity changes, trade transactions and price updates. It defines hooks for liquidity and trading operations, receives price information from external price providers, and controls whether price changes will be accepted according to certain rules. This helps the platform provide a fair and secure trading environment for liquidity providers and traders.


![image](https://github.com/code-423n4/2024-02-hydradx/assets/104318932/33daceb4-78a2-4646-9417-e42dfa0b2022)


##### Note: Please click on the image to enlarge it:

</br>
</br>


###  types.rs:

This Rust module defines various data structures and helper functions for HydraDX Omnipool. These structures and functions manage the state of the liquidity pool, the tradability of assets, and the positions of users.

### Balance and Price Type Definitions:
- `Balance`: Represents the balance type used in Omnipool (usually `u128`).
- `Price`: A fixed-point number type used to represent asset prices (`FixedU128`).

### Tradability Structure:
- Contains flags that define how assets can be traded in Omnipool (for example, they can be bought or sold, liquidity added or removed).
- `Tradability::is_safe_withdrawal`: Controls whether safe withdrawals can be made (permissions to add or remove liquidity).

### AssetState Structure:
- Represents an asset's status in Omnipool, includes fields such as hub reserve, number of shares, number of protocol shares, weight limit, and tradability status.
  
### Position Structure:
- Represents the positions users hold in Omnipool. It includes an asset ID, quantity, number of shares, and price at the time liquidity was provided.

### SimpleImbalance Structure:
- Represents a positive or negative imbalance. This is used to monitor how the balance of the liquidity pool changes.

### AssetReserveState Structure:
- Represents the asset's reserve status in Omnipool. This includes the asset's current reserve, hub reserve, share counts, and tradability.

### Functions and Implementations:
- `price_from_rational`, `delta_update`, `price_as_rational`, `price`, `weight_cap`: These functions perform various calculations on entity states and positions.
- `Add` and `Sub` operations are defined for the `SimpleImbalance` structure and are used to update the imbalance amounts.


![image](https://github.com/code-423n4/2024-02-hydradx/assets/104318932/59ddde61-6522-4d21-a3db-ffdbf33d607a)



</br>
</br>
</br>
</br>



## c) Test analysis

### Quality of Tests

1. **Comprehensiveness**: Shared tests cover various functions of the project; There are tests including asset registration, asset updates, liquidity limitations, claim transactions and more. This is critical to verify that important functionalities of the project are working correctly.

2. **Scenario Diversity**: In addition to successful transactions, the tests also test the behavior of the system in the face of incorrect or unexpected situations (for example, invalid signatures, unauthorized access, overflow situations). This increases the code's robustness and resistance to abuse.

3. **Detailed Assertions**: Tests verify the expected results of transactions in detail. This helps determine precisely whether the code is executing the expected behavior.

### Areas for Improvement

1. **Verifying Error Messages**: Some tests only check whether the operation failed; However, there are few assertions verifying whether the failure occurred with the expected error message. Verifying error messages helps understand whether errors occur for expected reasons or due to other problems.

2. **Organization of Test Scenarios**: The organization and grouping between test files and test cases can be further improved. For example, test modules grouped by functionality can increase the understandability and manageability of tests.

3. **Extension of Negative Test Scenarios**: The system must also behave appropriately under negative conditions. Although negative scenarios have been considered in current tests, further expansion of such scenarios could increase the robustness of the system.


</br>
</br>
</br>
</br>

## d) Security Approach of the Project

### Successful current security understanding of the project;

1-The audit of the project was carried out by "Runtime Verification" and "BlockScience", in addition, it will increase its security with Code4rena, where many auditors will audit the code. Also, Project has Immunefi bug bounty reward program.


### What the project should add in the understanding of Security;


1- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

2- We also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)


3 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19

4 -Another security approach I can recommend is 0xDjango's security approaches in a project. This approach divides security into layers (Test, Automatic Tool, Individual Audit, Corporate Audit, Economic Audit) and aims to have each part done separately by experts in the field. I can also recommend this approach to you;

https://x.com/0xDjangoOnChain/status/1748402771684909094?s=20

</br>
</br>

## e) Other Audit Reports

**Reports :**
https://github.com/galacticcouncil/HydraDX-security/tree/main/audit-reports

#### 230724-Runtime-Verification-Stableswap-Security-Audit:
This audit report covers the security audit of the Stableswap component of the HydraDX project.
The report was delivered on July 24, 2023.



#### 230619-Runtime-Verification-EMA-Oracle-Security-Audit :
This audit report covers the security audit of the EMA Oracle component of the HydraDX project.
The report was delivered on June 19, 2023.



#### 220322-BlockScience-Omnipool-Report+addendum-by-HydraDX :
This audit report contains the economic and technical analysis of the Omnipool component of the HydraDX project.
The report was delivered on March 22, 2022.



</br>
</br>
</br>
</br>

## f) Centralization Risk

 **Root Permissions**: Functions like `set_remove_liquidity_limit` are restricted to `RuntimeOrigin::root()`, indicating that only privileged accounts (administrators) can adjust liquidity limits. This centralizes decision-making power over critical financial parameters.



#### Code Pattern Used
By using `require(msg.sender == _ownerAddress);` it follows a pattern that ensures that certain functions can only be called by the contract owner. This indicates a centralized management model.

#### Suggestions

1. **Multisig Mechanism:** A multi-signature mechanism can be used that shares the powers of the contract owner among more than one address. This reduces the risk of a single point of failure and prevents abuse of authority.

2. **Transition to DAO Structure:** Moving project management and decision-making processes to a DAO (Decentralized Autonomous Organization) structure gives community members more say and reduces centralization.

3. **Time Lock and Escalation Mechanisms:** A time lock mechanism can be implemented so that critical functions can be called. This prevents sudden changes and gives the community a chance to review upgrades and changes.



</br>
</br>
</br>
</br>





### Time spent:
22 hours