# Table of contents

## Low severity issues

| Code | Title                                                                                                                                           |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| L01  | [`MinimumPoolLiquidity` can be violated in omnipool contract](#l01-minimumpoolliquidity-can-be-violated-in-omnipool-contract)                   |
| L02  | [`remove_token` feature can be blocked easily](#l02-remove_token-feature-can-be-blocked-easily)                                                 |
| L03  | [Omnipool NFTs vulnerable in flashloanable protocols](#l03-omnipool-nfts-vulnerable-in-flashloanable-protocols)                                 |
| L04  | [Missing `InsufficientLiquidity` check in omnipool `sell`](#l04-missing-insufficientliquidity-check-in-omnipool-sell)                           |
| L05  | [LRNA tokens can still be bought from omnipool by manipulating price](#l05-lrna-tokens-can-still-be-bought-from-omnipool-by-manipulating-price) |

## Informational

| Code | Title                                                                                                                 |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| I01  | [Inconsistent hooks call position](#i01-inconsistent-hooks-call-position)                                             |
| I02  | [Incorrect Natspec](#i02-incorrect-natspec)                                                                           |
| I03  | [`InsufficientLiquidity` check in `buy` function too low](#i03-insufficientliquidity-check-in-buy-function-too-low)   |
| I04  | [Missing 0 amount check in `withdraw_protocol_liquidity`](#i04-missing-0-amount-check-in-withdraw_protocol_liquidity) |
| I05  | [Unused pallet indices](#i05-unused-pallet-indices)                                                                   |

# Low

## L01 `MinimumPoolLiquidity` can be violated in omnipool contract

### Description

The `add_liquidity` function in the omnipool contract enforces a minimum deposit amount of the liquidity token as shown below.

```rust
ensure!(
        amount >= T::MinimumPoolLiquidity::get(),
        Error::<T>::InsufficientLiquidity
    );
```

However, there is no such check during removal of liquidity when calling the `remove_liquidity` function. Thus a user can add liquidity of 100 tokens (assume 100 > `MinimumPoolLiquidity`) and then remove 99 tokens, leaving 1 token in the pool which is less than the `MinimumPoolLiquidity` requirement.

Since the developer confirmed that the runtime forbids users from sending registered assets to the omnipool, this cannot be used to carry out an inflation attack, and is thus a low severity issue by itself.

### Mitigation

Consider adding a check that the remaining amount of liquidity after removal is above the `MinimumPoolLiquidity` requirement.

## L02 `remove_token` feature can be blocked easily

### Description

The Omnipool contract has a `remove_token` feature that allows the owner to remove a token from the pool. However, this requires the protocol to own every share of the liquidity provided. Thus if any user has even a single share of liquidity with them, this feature cannot be used.

```rust
ensure!(
        asset_state.shares == asset_state.protocol_shares,
        Error::<T>::SharesRemaining
    );
```

It is quite impractical to expect every user in a decentralised system to sacrifice or remove their liquidity to allow the owner to remove a token. Users can just be offline with a small dust amount of liquidity and that would be enough to block this feature from being used.

Since there is no material impact on the pool as the asset can be frozen and the pool can continue to operate, this is a low severity issue.

### Mitigation

Consider adding a force removal option that allows the owner to remove a token after a certain time period.

## L03 Omnipool NFTs vulnerable in flashloanable protocols

### Description

The Omnipool contract mints users position NFTs when they add liquidity to the pool. When they remove liquidity, the same NFT is modified instead of destroying and minting a new one. This can be a problem if the NFT is used in other defi protocols which allows flashloaning of NFTs.

If such a liquidity provision NFT ends up in such a defi protocol, any user can flashloan the NFT, take out all the liquidity from the pool, and then return the NFT back to the original owner. Since the tokenId remains the same, this is allowed by virtually all NFT handling protocols.

### Mitigation

Consider burning and re-creating the NFT when a user removes liquidity. Otherwise, this will be an acceptable risk for the protocol.

## L04 Missing `InsufficientLiquidity` check in omnipool `sell`

### Description

The `sell` function in the omnipool contract does not check if it has enough `asset_out` tokens in the pool to pay out the user after the sale. This is present in the `buy` function, but is absent in the `sell` function.

```rust
// buy function
ensure!(asset_out_state.reserve >= amount, Error::<T>::InsufficientLiquidity);
```

This can lead to incorrect errors being thrown when the pool does not have enough tokens to pay out the user.

### Mitigation

Consider adding this check before transferring the tokens to the user.

## L05 LRNA tokens can still be bought from omnipool by manipulating price

### Description

Buying of LRNA tokens from the omnipool is forbidden by the code. But LRNA tokens are still paid out to the liquidity provider if they remove liquidity at a higher price than they put in. This is because due to the increase in price there are fewer asset tokens to give out, and thus some LRNA tokens are given out to the user to compensate them.

So a user can add liquidity in asset_a, then buy up asset_a using asset_b. Then when they remove liquidity, they get back asset_a as well as LRNA tokens. Assuming 0 slippage, from the user's perspective, they have just swapped in asset_as for LRNA tokens.

This is not exploitable since the price ratios in the pool still remain the same, which would have been the same in a direct buy. However this is still possible while being explicitly blocked by the protocol.

# Informational

## I01 Inconsistent hooks call position

Ideally, hooks should be called at the very end of the function operation, after all the variables have been updated. This is because if the hooks try to query the contract, they should be accessing the latest updated values. While this is done in some functions, hooks are called before the end in certain functions.

List of functions where hooks are called before the end/update of storage variables:

-   [ ] `add_token` in Omnipool (hook called before asset update)
-   [ ] `sell` in Omnipool
-   [ ] `buy` in Omnipool
-   [ ] `update_hdx_subpool_hub_asset` in Omnipool (hook called before processing trade fee)
-   [ ] `sell_hub_asset`
-   [ ] `buy_asset_for_hub_asset`

## I02 Incorrect Natspec

Incorrect/Missing/Inconsistent Natspec in certain areas of the code.

Missing input parameter documentation:

-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L869-L875
-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1362-L1370
-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1428-L1438
-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L847

Natspec order of input parameters different from the function's order of input parameters:

-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L437-L440
-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1127-L1130

Incorrect Natspec:

-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L399-L402

Natspec uses ' instead of \`:

-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L543-L544
-   [ ] https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L630-L631

## I03 `InsufficientLiquidity` check in `buy` function too low

The `InsufficientLiquidity` check in the omnipool buy function is present [here](https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/omnipool/src/lib.rs#L1173). This is below the `buy_asset_for_hub_asset` call, and thus functions will revert with incorrect error messages, probably due to underflow in the math functions. Consider moving this up higher in the function. Also do the same for the `sell` function.

## I04 Missing 0 amount check in `withdraw_protocol_liquidity`

The `withdraw_protocol_liquidity` function in omnipool does check if the admin is passing in 0 as the amount to withdraw. Consider adding this check.

## I05 Unused pallet indices

The omnipool contract has some missing/unused pallet indices. `pallet::call_index(0)` is not used. `pallet::call_index(10)` is skipped over while both the indexes 9 and 11 are used.
