
# 1- Users can bypass min trading limit imposed in `add_liquidity` by using `add_liquidity_shares`

## Issue Description

When adding liquidity with the `add_liquidity` function a minimum amount is required to be deposited for each asset included in the `assets` array, this done to ensure that user will not create positions with dust amounts:

```rust
fn do_add_liquidity(
    who: &T::AccountId,
    pool_id: T::AssetId,
    assets: &[AssetAmount<T::AssetId>],
) -> Result<Balance, DispatchError> {
    let pool = Pools::<T>::get(pool_id).ok_or(Error::<T>::PoolNotFound)?;
    ensure!(assets.len() <= pool.assets.len(), Error::<T>::MaxAssetsExceeded);
    let mut added_assets = BTreeMap::<T::AssetId, Balance>::new();
    for asset in assets.iter() {
        ensure!(
            Self::is_asset_allowed(pool_id, asset.asset_id, Tradability::ADD_LIQUIDITY),
            Error::<T>::NotAllowed
        );
        //@audit minimum deposit check
        ensure!(
            asset.amount >= T::MinTradingLimit::get(),
            Error::<T>::InsufficientTradingAmount
        );
        ensure!(
            T::Currency::free_balance(asset.asset_id, who) >= asset.amount,
            Error::<T>::InsufficientBalance
        );
        if added_assets.insert(asset.asset_id, asset.amount).is_some() {
            return Err(Error::<T>::IncorrectAssets.into());
        }
    }

    ...
}
```

But there is another function that enable adding liquidity, the `add_liquidity_shares` function, this allows a user to specify the liquidity shares that he wants to get by provididing one asset to the pool, the issue in this function is that after calculating the required amount of asset to mint the specified shares it doesn't make sure that the asset amount is above the minimum deposit amount (as it's doen in `add_liquidity`), thus user might be able to create liquidity position with very small asset amounts (dust):

```rust
fn do_add_liquidity_shares(
    who: &T::AccountId,
    pool_id: T::AssetId,
    shares: Balance,
    asset_id: T::AssetId,
    max_asset_amount: Balance,
) -> Result<Balance, DispatchError> {
    ensure!(
        Self::is_asset_allowed(pool_id, asset_id, Tradability::ADD_LIQUIDITY),
        Error::<T>::NotAllowed
    );
    let pool = Pools::<T>::get(pool_id).ok_or(Error::<T>::PoolNotFound)?;
    let asset_idx = pool.find_asset(asset_id).ok_or(Error::<T>::AssetNotInPool)?;
    let share_issuance = T::Currency::total_issuance(pool_id);
    let amplification = Self::get_amplification(&pool);
    let pool_account = Self::pool_account(pool_id);
    let initial_reserves = pool
        .reserves_with_decimals::<T>(&pool_account)
        .ok_or(Error::<T>::UnknownDecimals)?;

    // Ensure that initial liquidity has been already provided
    for reserve in initial_reserves.iter() {
        ensure!(!reserve.amount.is_zero(), Error::<T>::InvalidInitialLiquidity);
    }

    let (amount_in, _) = hydra_dx_math::stableswap::calculate_add_one_asset::<D_ITERATIONS, Y_ITERATIONS>(
        &initial_reserves,
        shares,
        asset_idx,
        share_issuance,
        amplification,
        pool.fee,
    )
    .ok_or(ArithmeticError::Overflow)?;

    ensure!(amount_in <= max_asset_amount, Error::<T>::SlippageLimit);

    ensure!(!amount_in.is_zero(), Error::<T>::InvalidAssetAmount);
    let current_share_balance = T::Currency::free_balance(pool_id, who);

    //@audit did not check that amount_in is above min trading limit

    ensure!(
        current_share_balance.saturating_add(shares) >= T::MinPoolLiquidity::get(),
        Error::<T>::InsufficientShareBalance
    );

    T::Currency::deposit(pool_id, who, shares)?;
    T::Currency::transfer(asset_id, who, &pool_account, amount_in)?;

    //All done and update. let's call the on_liquidity_changed hook.
    Self::call_on_liquidity_change_hook(pool_id, &initial_reserves, share_issuance)?;

    Ok(amount_in)
}
```

## Impact

The current implementation allows users to bypass the minimum liquidity deposit requirement by using the `add_liquidity_shares` function. This enables them to create positions with dust amounts, contrary to the intended behavior of the protocol.

## Tools Used

Manual review, VS Code

## Recommended Mitigation

A straightforward solution to address the aforementioned issue is to add a check to ensure the calculated `amount_in` is above the minimum limit :

```diff
fn do_add_liquidity_shares(
    who: &T::AccountId,
    pool_id: T::AssetId,
    shares: Balance,
    asset_id: T::AssetId,
    max_asset_amount: Balance,
) -> Result<Balance, DispatchError> {
    ...

    ensure!(amount_in <= max_asset_amount, Error::<T>::SlippageLimit);

    ensure!(!amount_in.is_zero(), Error::<T>::InvalidAssetAmount);
    let current_share_balance = T::Currency::free_balance(pool_id, who);

++  ensure!(
++      amount_in >= T::MinTradingLimit::get(),
++      Error::<T>::InsufficientTradingAmount
++  );

    ensure!(
        current_share_balance.saturating_add(shares) >= T::MinPoolLiquidity::get(),
        Error::<T>::InsufficientShareBalance
    );

    ...
}
```

# 2- The dynamic fee should only be applied when withdrawal is not safe in `remove_liquidity` function

## Issue Description

When users attempt to withdraw liquidity from the omnipool by calling the `remove_liquidity` function, a dynamic withdrawal fee is calculated based on current asset prices. This fee is calculated using the `calculate_withdrawal_fee` function from the omnipool math module, which takes into account the current pool price (spot price) and an external oracle price:

```rust
pub fn remove_liquidity(
    origin: OriginFor<T>,
    position_id: T::PositionItemId,
    amount: Balance,
) -> DispatchResult {
    ...

    let safe_withdrawal = asset_state.tradable.is_safe_withdrawal();
    // Skip price check if safe withdrawal - trading disabled.
    if !safe_withdrawal {
        T::PriceBarrier::ensure_price(
            &who,
            T::HubAssetId::get(),
            asset_id,
            EmaPrice::new(asset_state.hub_reserve, asset_state.reserve),
        )
        .map_err(|_| Error::<T>::PriceDifferenceTooHigh)?;
    }
    let ext_asset_price = T::ExternalPriceOracle::get_price(T::HubAssetId::get(), asset_id)?;

    if ext_asset_price.is_zero() {
        return Err(Error::<T>::InvalidOraclePrice.into());
    }
    //@audit dynamic fee should only be applied if withdrawal is not safe
    let withdrawal_fee = hydra_dx_math::omnipool::calculate_withdrawal_fee(
        asset_state.price().ok_or(ArithmeticError::DivisionByZero)?,
        FixedU128::checked_from_rational(ext_asset_price.n, ext_asset_price.d)
            .defensive_ok_or(Error::<T>::InvalidOraclePrice)?,
        T::MinWithdrawalFee::get(),
    );

    ...
}
```

According to comments in the `remove_liquidity` function, the dynamic fee should only be applied when the withdrawal is not safe, meaning when the asset is not in the FROZEN state, indicating that trading is disabled:

```rust
/// Dynamic withdrawal fee is applied if withdrawal is not safe. It is calculated using spot price and external price oracle.
/// Withdrawal is considered safe when trading is disabled.
```

However, in the current implementation of the function, the dynamic fee is applied in all conditions, irrespective of whether the withdrawal is safe or not. This discrepancy may result in users attempting to withdraw their liquidity paying the wrong fee and receiving an inaccurate amount of assets in return.

Moreover, the function only checks the difference between the spot price and oracle price when the withdrawal is not safe. It does not perform this check when the withdrawal is safe. This omission further increases the likelihood of users paying a higher fee, as the `calculate_withdrawal_fee` function utilizes these prices to calculate the fee without validating them first.

## Impact

The application of the dynamic fee even when the withdrawal is safe may result in users paying the wrong fee and receiving an inaccurate amount of assets when calling `remove_liquidity`.

## Tools Used

Manual review, VS Code

## Recommended Mitigation

To address this issue, the dynamic fee should only be implemented when the withdrawal is safe `safe_withdrawal == true`.

# 3- The calculated `c` value is not divided by `amplification * n^n`

## Issue Description

In both the `calculate_spot_price` and `calculate_share_price` functions, the calculated value for the parameter `c` is not divided by the factor `amplification * n^n` as it was done when calculating the Y value in the `calculate_y_internal` function. This oversight could lead to incorrect prices being returned by these functions, potentially impacting other parts of the protocol that rely on them.

```rust
pub fn calculate_spot_price(
    reserves: &[AssetReserve],
    amplification: Balance,
    d: Balance,
    asset_idx: usize,
) -> Option<(Balance, Balance)> {
    let n = reserves.len();
    if n <= 1 || asset_idx > n {
        return None;
    }
    let ann = calculate_ann(n, amplification)?;

    let mut n_reserves = normalize_reserves(reserves);

    let x0 = n_reserves[0];
    let xi = n_reserves[asset_idx];

    let (n, d, ann, x0, xi) = to_u256!(n, d, ann, x0, xi);

    n_reserves.sort();
    let reserves_hp: Vec<U256> = n_reserves.iter().map(|v| U256::from(*v)).collect();
    //@audit c not divided by ann
    let c = reserves_hp
        .iter()
        .try_fold(d, |acc, val| acc.checked_mul(d)?.checked_div(val.checked_mul(n)?))?;

    let num = x0.checked_mul(ann.checked_mul(xi)?.checked_add(c)?)?;
    let denom = xi.checked_mul(ann.checked_mul(x0)?.checked_add(c)?)?;

    Some(round_to_rational(
        (num, denom),
        crate::support::rational::Rounding::Down,
    ))
}
```

```rust
pub fn calculate_share_price<const D: u8>(
    reserves: &[AssetReserve],
    amplification: Balance,
    issuance: Balance,
    asset_idx: usize,
    provided_d: Option<Balance>,
) -> Option<(Balance, Balance)> {
    let n = reserves.len() as u128;
    if n <= 1 {
        return None;
    }
    let d = if let Some(v) = provided_d {
        v
    } else {
        calculate_d::<D>(reserves, amplification)?
    };
    let n_reserves = normalize_reserves(reserves);
    //@audit c not divided by ann
    let c = n_reserves
        .iter()
        .try_fold(FixedU128::one(), |acc, reserve| {
            acc.checked_mul(&FixedU128::checked_from_rational(d, n.checked_mul(*reserve)?)?)
        })?
        .checked_mul_int(d)?;

    let ann = calculate_ann(n_reserves.len(), amplification)?;

    let (d, c, xi, n, ann, issuance) = to_u256!(d, c, n_reserves[asset_idx], n, ann, issuance);

    ...
}
```

## Recommended Mitigation

It's crucial to review the code of both functions and ensure that the division by the `amplification * n^n` factor is included if it was forgotten.