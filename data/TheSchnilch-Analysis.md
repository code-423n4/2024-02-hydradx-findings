## Summary of the In-Scope part of the codebase

The code for this audit can be divided into two main parts: Omnipool, Stableswap, Oracle and Circuit Breaker

### Omnipool

This is a pool into which all possible assets can be added by governance. Since it is a pool with all assets, there is a subpool in the omnipool for each asset, which consists of the asset and LRNA (hub token). Since all assets have LRNA as a second asset in their subpool, you can simply swap all assets by first swapping the asset that the user puts in into LRNA and then swapping this LRNA into the asset that should come out. This is done automatically by the sell and buy functions. After the state of the pool has been modified, a hook is called to update the oracle. In addition, the circuit breakers also check in these hooks that the amount traded does not exceed a limit that is measured per block.

- A brief overview of all external callable functions of the omnipool
    - add_token - This function is there so that governance can add a new token with initial liquidity to the pool
    - remove_token - With this function, governance can remove a token if all shares that still exist belong to the protocol
    - add_liquidity - With this function, every user can add liquidity to a specific token and receive shares in the form of an NFT
    - remove_liquidity - With this function, a liquidity provider can remove liquidity from his position
    - sell - Swap from one asset to another where the user specifies the amount of asset in
    - buy - Swap from one asset to another where the user specifies the amount of asset out
    - sacrifice_position - Liquidity providers can transfer their shares to the protocol by deleting their position and giving the shares to the protocol without receiving anything in return
    - set_asset_tradable_state - With this function, governance can determine the tradability of an asset
    - refund_refused_asset - If the initial liquidity has already been sent to the pool for a new asset and the asset was not added, the initial liquidity can be refunded using this function
    - set_asset_weight_cap - This function is for governance to set the weight cap for an asset
    - withdraw_protocol_liquidity - With this function the liquidity belonging to the protocol with the protocol shares of sacrifice_position can be withdrawn

### Stableswap

Stableswap is also a pool that can hold 2-5 assets. However, these assets must be fixed when the pool is created and can no longer be changed. The pool is designed to contain only assets with the same price, e.g. DAI, USDC and USDT. However, several stable pools can be created. Hooks are also used here to update the oracle.

- A brief overview of all external callable functions of stableswap
    - create_pool - This allows governance to create a new pool with a few tokens that have the same price. The amplifiction and fees are also set
    - update_pool_fee - This allows governance to set a new fee value for a specific pool
    - update_amplification - With this function, governance can set a new value for amplification for a specific pool
    - add_liquidity - This allows a liquidity provider to add liquidity for one or more tokens by specifying how many of them they would like to provide. In return they then receive share assets
    - add_liquidity_shares - The same as add_liquidity only that the liquidity provider can only add tokens to a single asset and that he specifies the number of shares he would like to receive
    - remove_liquidity_one_asset - With this function, a liquidity provider can burn a certain number of shares and receive a part of the liquidity in return
    - withdraw_asset_amount - The same as remove_liquidity_one_asset but the liquidity provider does not specify the number of shares to burn but rather the number of assets that he would like to receive.
    - sell - This allows a user to swap a specific token for another by specifying the amount they want to sell
    - buy - This allows a user to swap one token for another by specifying the number of tokens they want to buy
    - set_asset_tradable_state - With this function, governance can determine the tradability of an asset

### Circuit Breaker

The circuit breaker limits how much liquidity can be traded, added and removed within a block. This is done by storing all amounts that are added, removed and traded within a block and ensuring that they are under a certain limit. After a block these values are reset.

### Oracle

The oracle receives the data through the hooks after liquidity has been added, removed or traded.  This data is not saved directly and used by the oracle but only at the end of the block using exponential moving average logic. The oracle also stores the price for different time periods. So you should note that the price of Oracle is at least a block old.

## Architecture and code analysis

- The code was built using substrate and the substrate-node-template, which already implements core features of a blockchain and is very configurable. In this case, it was used to build a parachain for Polkadot. You can find out more about it here: https://docs.substrate.io/, https://github.com/substrate-developer-hub/substrate-node-template
- The code is well structured by dividing important components such as omnipool, stableswap, ema-oracle and circuit-breaker into different pallets. There is also an extra crate for math.
- The code also looks well written and wasn't too difficult to read. This is also because larger functions are separated into different parts, e.g. Calculations were not done in the main function and the fact that the oracles were updated via hook is a good idea in my opinion
- The error handling in calculations could be improved. Often the error ArithmeticError::Overflow is simply returned, even if it is a completely different error. This makes testing more difficult because you have to find the right error yourself. I would recommend creating custom error messages for this.
- Events are always emitted at the end of the function and contain the important information for the offchain services.

## Protocol Risks & Recommendations

- Even though the code already validates values in many places, there are still some places where this could still be improved:
    - remove_liquidity: I would recommend a check like in the other functions that the amount entered must be more than a minimum number of shares.
    - sell_hub_asset: Here, as in the sell function, there should also be a check that state_changes.asset.delta_reserve is greater than 0 to ensure that the user does not get no assets out even though he sells a few assets
    - calculate_add_liquidity_state_changes: There should be a check whether shares_hp is zero, since if this is the case the liquidity provider would add its tokens to the pool but would not receive any shares in return. This scenario can occur in the unlikely event that the initial liquidity provider, if whitelisted, removes all of its liquidity after the pool is created
- The circuit breakers protect against the liquidity in the omnipool changing too much within a block to protect the pool. It would be advisable to adapt this mechanism and then also protect the stableswap with it.
- In stableswap a function to remove a single token would be good. In a case where a token is depegged or decommissioned, it would be practical if governance could simply remove that token instead of having to set up the entire pool again without the one token.

## Centralization & Special Roles

- The protocol is controlled by democratic governance (not in scope). This is mainly built with the substrate democracy module. All users that hold HDX can take part through referendums. These can be all sorts of proposals, including changes to the runtime code. Voting period of a referendum is 3 days and then an enactment period which lasts another 6 days. There is also an emergency referendum that can be used by the technical committee to make important changes to the code, such as fixing a security hole. The voting period is then only 3 hours. The technical commitee then consists of a few of the core developers of the protocol. This emergency mechanism is very good for reacting to security gaps or other events that require quick action. In a normal referedum, however, only one can be done in the voting period. All other referenda are in a queue at this time. Every 3 days, the referendum with the greatest support is moved into the voting period. This mechanism could be improved by making it possible for several referenda to be in the voting period at the same time. The protocol could be improved more quickly through the proposals. I wouldn't recommend putting all proposals into the voting period at the same time, but 3-4 should be good.
    
    Here is a list of functions from Omnipool that can be used by governance to set new parameters for the protocol and the impact these functions can have:
    
    - add_token - Impact: This means that governance has control over which new tokens are added. Here governance gets a lot of power because it can set the initial price. If this is wrong there can be quite large arbitrage opportunity. If weight_cap is set to 0 then it is not possible for liquidity to be added for this asset.
    - remove_token - Impact: This can be used by governance to remove a token from the pool when an asset is frozen and whatever liquidity is still there belongs to the protocol. So this shouldn't have a big impact on users.
    - set_asset_tradable_state - Impact: This function can only be used by the technical commitee. This allows an asset to be given different states so that liquidity can be removed, added, sold and bought. In addition, an asset can also be frozen, which means you can't do any of these things with the asset. So the technical commitee has the power to make an asset almost unusable if they freeze it.
    - refund_refused_asset - Impact: Since this can only be used to refund tokens of an asset that has not been added to the pool, this function has no impact on users of the pool or other parts of the protocol.
    - set_asset_weight_cap - Impact: If the weight cap is set incorrectly here, i.e. too low or 0, it can happen that it is only possible to add very little or no liquidity to an asset.
    - withdraw_protocol_liquidity - Impact: This function can have a big impact on the protocol and the liquidity provider, because if the price is set incorrectly, a lot more liquidity can be withdrawn from an asset than should actually be possible. Tokens are stolen from liquidity providers and the price of the asset is manipulated. This could be mitigated by storing the price that the protocol shares had when they were given to the protocol.
    
    Here is a list of functions from Stableswap that can be used by governance to set new parameters for the protocol and the impact these functions can have:
    
    - create_pool - Impact: Here the protocol has the option of creating a new pool with any tokens. Governance is responsible for ensuring that only tokens with the same price enter a pool. Amplification can only be set in a certain area, so it shouldn't have a very big impact. However, the fee can be set very high, which means that a pool becomes unusable, because a user would get almost nothing from a swap and would only pay fees.
    - update_pool_fee - Impact: As with create_pool, the fee can be set here without any validation, which can result in it being 100% and the pool becoming unusable and users only paying fees and getting nothing in return.
    - update_amplification - Impact: Here too, the amplification can only be set in a certain area and should not have a very large impact.
    - set_asset_tradable_state - Impact: This function has the same impact as the one in the Omnipool with the exception that not only the technical commitee is allowed to use this function.
    
    Even with the circuit breaker, governance has the power to restrict the Omnipool very much if the limits are very low. This can make the omnipool almost unusable.
    

## Testing

- There are unit tests for all pallets that are in-scope and ensure that the functions do what they are supposed to.
- There are also integration tests for the omnipool, circuit-breaker and ema-oracle pallets, which test whether the pallets also work with all the others in runtime. I would also recommend adding integration tests for stableswap as this is a very important part of the protocol. Such tests can also help when the code is updated so that you can see directly whether the tests still work or whether an error was made during the update.

## Code documentation and comments analysis

- The short explanations above the most important external callable functions in Omnipool and Stableswap were very helpful for getting a brief impression of what the function does. I would recommend doing this for other important functions such as the hooks and the calculation functions, which would make reading the code even easier.
- There is a README for all pallets that briefly explains what a pallet is for. This came in handy when I wanted to understand the meaning of a pallet without reading the entire code. (E.g. for pallets that are out of scope for this contest)
- Apart from the comments about the external callable functions, there weren't many others. For a few important sections within a function, it would be good if there were a few more comments.
- There were some very good explanations for the design of the Omnipool in the docs (https://docs.hydradx.io/omnipool_design). It would be good if there was a page like this in the docs for stableswap too.

### Time spent:
25 hours