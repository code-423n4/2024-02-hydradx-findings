# Overview of HydraDX 
HydraDx omnipool is cutting edge AMM that utilizes omnipool to gather all assets into one single pool which allow seemless trading between asset and facilates buy and selling assets in polkadot ecosystem 

HydraDx protocol offers stapleswap pallet which is offering users the ability to trade stablecoins with an exceptionally low price slippage . 

the scope of this contest consists of 4 main components which are :
1) **omnipool pallet** 
2) **Stableswap pallet**
3) **EMA oracle** 
4) **Circuit breaker**

 ### **1) the omnipool pallet :** 
 the main pallet which implements all the logic of the omnipool and has all the math , types ,and traits files that are neccessary of the implementation of the omnipool . 
   - **lib.rs :** the main file of the pallet which contains all the contants and Errors ,and all the callable function for users such as `sell()` , `buy()` , `add_liquidity()` , `remove_liquidity()` and also the governence functions of the protocol such as `add_token()` , `withdraw_protocol_liquidity()` .     
   - **math.rs :** implements all the calculations which are required to perform swapping , liquidity provision and removal , and calculations of the fees taken from each swap and liquidity removal . 
   - **types.rs :** contains the custom types used in the omnipool such as `Tradability` , `AssetState<Balance>` ,`AssetReserveState<Balance> ` , and their implementation traits and all the functionallity required to deal with them such as Arithmatic implementation of type `SimpleImbalance` which are `add()` , and `sub()`, and price calculation functions to the type `AssetReserveState<Balance>` such as `price_as_rational()` and `delta_update()`.
   -  **traits.rs :** which has the `omnipoolHooks` such as `on_liquidity_changed()` , `on_trade` , and implements the logic of `ensure_price()` functions with ensure that the price is within the bounds of the current spot price and the external oracle price.
   -  **types.rs :** contains all the custom functions of each type such as `add()` which the adding operation for the type `BalanceUpdate` , and custom types that represent state changes such as `HubTradeStateChange` , `LiquidityStateChange` , `TradeStateChange` . 
   
### 2) stabeswap pallet : 
- **lib.rs :** this file contains the main implementations of the pallet which include liquidity provision and removal , swapping (buy/sell) , and create liquidity pools .
- **math.rs :** this file contains all math implementations which are required for the pallet such as `calculate_withdraw_one_asset` ,and `calculate_shares_for_amount` .
- **types.rs :** this file contains the custom type related to the pallet and its implementations such as `AssetReserve` .
# System overview and risks 
HydraDX is well-designed AMM that implement cutting edge technology of trading assets , the system architechture consists of pallets , each pallet contains the implementations of a specific component which are configered in the runtime pallet , the main pallets for this audit are the onmipool pallet and the stableswap pallet .  

## Omnipool Pallet :

Omnipool is type of AMM where all assets are pooled together into one single pool.
the main files that implement the most of the logic are **lib.rs** and **math.rs**.
### 1) Analysis and Security considerations about each function :
1) **`add_token()`** : this function is used to add new asset to the omnipool , the asset to be added should be risgtered in the asset registry first and then the token can be added to the omnipool by the `authority_origin` , and the token should have initial position to be created with amount of asset is greater than zero . 

 - **security considerations** : 
  This function does not have any checks that are implemented in the `add_liuqidity()` function although that the function `add_token()` perform the same logic of adding liquidity so the lack of those checks can result in leaving the asset pool in bad state and prevent liquidity provision . 
 also the function should check that `hub_asset` LRNA is not added to the pool to prevent withdrawal of all the hub_asset exist in the pool which will leave the pool in bad state , so this function should implement a check to prevent this from happened , it also should check that the `max_weight_cap` is not exceed . 
  
2) **`add_liquidity()`:**  this function is called when the user want to provide liquidity and due to that this function is public and can be called by anyone this function should be well protected , It takes assets from the user and mint him a correspoding amount of shares and mint a NFT instance that will corespond to this position and send it to the user , the function does not have slippage protection parameter so it is exposered to slippage , but due to the checks of the price difference between `oracle_price` and `spot_price`.  
the function calculate the shares to be minted to the user by call the function `calculate_add_liquidity_state_changes()` , then the function create the position , and insert it in the position mapping , then transfer the tokens from the user . 
- **Security considerations :**
   This function is vulnerable to reentrancy due to partial storage update , the function calculates the shares of the user and then construct the position and mint the nft to the user and then update the storage , then the function transfer the assets from the user to the protocol . This patial update of storage allow the user to own a position and shares without updating the `asset_state` which allow the user to remove more liquidity from the pool than what is supposed to be removed , by using the hooks the supported by the assets contract .
   The function should first update the stoarge completely and then transfer the token and follow check-affect-interact patteren to secure the pool against reenterancy .
3) **`remove_liquidity()`:** this function is used to remove liquidity from the pool and burn the nft instance in case that the shares remained in the position is equal to zero , and takes the wthdrawal fees from the asset removed , a portion of the fees will remain in the pool as a reward for the lps and another portion will be send to the stakers and referrers .
- **Security considerations :**
this function should be protected against slippage and price manipulation , so it has a 2 layers of protection :
   1) `ensure_price()` that makes sure that price is within the bounds of the `oracle_price` and `spot_price` so this will protect against the price manipulation but this check will be deactivated if the tradability flag is only `REMOVE_LIQUIDITY` or `REMOVE_LIQUIDITY` and `ADD_LIQUIDITY` so the `technical origin` should make sure that the price is not manipulated when he set the tradability to this flags .
   2) `withdrawal_fee` this fee is implemented to prevent any price manipulation from be profitable so it will povide a protection layer for this function  .
4) **`Sell()` :** this function is used to sell asset from the omnipool for another asset which is the main functionallity of the omnipool , this function swap an amount of asset and ensure that the amount out from this trade is greater than or equal to the amount specified by the user which protects from slippage or price manipulation . this function has a specific logic if the asset in is the `hub_asset` but it has not yet impelemented the logic in case that the asset out is the `hub_asset` , this function also calculates the asset fees that are taken from the asset out and also the protocol fees which are taken from the asset in , and then transfer the assets from the user and then transfer the assets out to the user . then update the state of the Pool . 
- **Security considerations :**
- It will be more safe to implement check-affect-interact pattern that will protect against reentrancy attacks , so the protocol should update the state of each asset first and then transfer the tokens to the user , to prevent any price manipulation . 
5) **`buy()` :** this function has the same functionality as `sell()` function except that it take the amount out as a parameter and calculte the amount in , this function has a slippage parameter .
### Roles :
Roles is provided by the omnipool pallet and .....  , the Roles are generally associated with different entities that have specific permissions or responsibilities. Here are the roles and their associated functionalities in the pallets :
1) authority Origin :
   - this role is governance role that has the permissions to call the function `add_token()` to add new token to the omnipool and can also call the function `refund_refused_asset` to send the token back to the liquidity provider in case that token is refused to be added to Omnipool, and initial liquidity of the asset has been already transferred to pool's account by the liquidity provider , this role also can `withdraw_protocol_liquidity` which allow to withdraw the shares from the sacrificed positions .
2) technical Origin :
   - this role is responsable for pausing/unpausing any functionality of each asset on the omnipool , this role can call `set_asset_tradability` function which sets the tradability flag of each asset to be one of those flags `FROZEN` , `SELL` , `BUY` ,`ADD_LIQUIDITY` , `REMOVE_LIQUIDITY` .

## Stableswap pallet : 
1) **create_pool** : this function can only be called by the authority origin and checks that asset to be added to the pool are registered before and there are no multiple pools with the same share asset to prevent set an unapproved asset to the pool and prevent any misconfiguration between pools and also this fucntion set the amplification and fees parameter for the pool
- **security considerations** :
- this function does not have enough checks to verify the `share_asset` parameter since the function does not check that the share asset has 18 decimals because the developers assumes that all shares asset have 18 decimals .
- also this function does not check that share_asset did not get added before in the pools so this will cause a huge loss of funds if happened .
2) **add_liquidity :** this function is callable by any user on the parachain , and this enable the users to provide liquidity to the pools by specifing the asset_id to be added and the amount of the assets , this function call the internal function `do_add_liquidity` to perform all the logic of adding liquidity , this function calculate the initial and updated reserves and pass them with the total issuance of the share_asset to the `calculate_shares()` function which calculates the D parametrs for the inital and the updated reserves then calculate the shares to be minted to the user by this formula : `(d1-d0)*total_issuance/d0` .
  - **security consideratons :** this function is well-secured by slippage parameter to protect the user against price manipulation and it also perform all balances checks to make sure that the user has a balance of shares that is above ED .
3) **add_liquidity_shares :** this function add liquidity by specifing the shares needed by the user so this function uses a different logic from the one implemented in `add_liquidity` function , this function calculates `d0` and `d1` then apply the fees on the reserves of the pool and then calculate the `y1` which represents the new reserve of the asset to be added .
4) **buy / sell** functions : those functions can be used to swapping assets from the pool by specifing the asset_id and the amount_in or amount_out , this functions calculate the D and Y parameters for each function call to return the right value of the trade .
5) **remove_liquidity_one_asset :** allows a user to remove liquidity from a specific asset in a stableswap pool by burning a specified amount of pool shares and receiving a corresponding amount of the asset in return, while ensuring the operation adheres to the pool's rules and updates the pool's state accordingly , this function calls `calculate_withdraw_one_asset` function which performs all the math to calculate the amount to be taken from the user ,The function calculates the new value of D1 after the shares are removed. It then calculates the amount of the asset that can be withdrawn  without considering fees, by adjusting the reserves to exclude the asset being withdrawn and recalculating D .
## system improvements 
1) allow partial distruction. 
this will be useful to allow donation to the protocol , if the protocol entered a bad state , so instead of only allow `full distruction` of position , allow also partial distruction and it should be done only by the owner of the position , this improvement will add an new feature to support the protocol and donate to it . 
2) set slippage parameter in `add_liquidity` and `remove_liquidity` functions .
The protocol should provide an additional layer of protection to the liquidity provider from slippage and price manipulation by allow them to set a safty parameter to ensure that the amount out from the protocol is equal to greater than what lps expected , so this will enhance the user experience in dealing with this omnipool . 
3) use dynamic and asset specific value for `minimumTradingLimit` and `minimumPoolLiquidity`
using dynamic minimum limit and make the it specific for each asset , will decrease the possibilty of preventing the token from being used , because each token in the ecosystem has diffrerent decimals and the value of each token differs also , so specifing a constant minimum limit for all asset can prevent some users from using this token because the minimum limit worth a big value or providing the minimum limit will exceed the `max_weight_cap` of this token .
4) add comments to all math files since they have a complex logic but there is no enough amount comments to describe all the calculations done .    
## Approach Taken-in Evaluating hydraDX Protocol 
Accordingly, I analyzed and audited the subject in the following steps:
1) omnipool pallet overview :
   - I started with this pallet because it has the main implementation that this audit is for and play critical role in hydraDx protocol , then I started by examining each function and I focused on the main functions that perform all the logic of the liquidity pool , and then I moved to the governance functions to examin how they are implemented ,and then I moved to reviewing the math of the omnipool in the `math.rs` file , which has all math implementation of adding and removing liquidity and also has the logic of selling and buy token from the pool 2) stableswap pallet overview :
   - I frist went line by line through the code to understand the functions and the implementation of the pallet , then I read [curve finance white paper](https://curve.fi/files/stableswap-paper.pdf) to get better understanding of stapleswap mechanism , then I reviewed math file to understand the complex math implementation of the pallet , and I wrote my notes which related to the vulnerabilities that I found  .
   - then I started to read tests and run them and then write my own tests to test edge cases . 
3) Documentation Review:
    then went to review the [docs](https://docs.hydradx.io/)  for a more detailed and technical explanation of Olas protocol.
4) compiling code and running the provided tests.
5)  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.
   - **Line by Line Analysis:**  Pay close attention to the pallet’s intended functionality and compare it with its actual behavior on a line-by-line basis.
   - **Comparison Mode:**  Compare the implementation of each function with established standards or existing implementations such as `sell` vs `buy` , `add_liuqidity` vs `remove_liquidity` ,and `add_token` vs `remove_token`.
6) **Reporting and writing POC** : I started collect my notes from the files and wrote a POC to my findings .

## time spent 
176 hours over 25 days 


### Time spent:
175 hours