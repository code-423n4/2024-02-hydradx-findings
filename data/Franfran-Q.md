## [Low 1] `normalize_value` cannot round up when normalizing with more decimals, leading to correctness issues

The function `normalize_value`  is used for the reserves to be compatible with the internal math of the protocol, since it has been intended to be working for unsigned integers of a precision of 18 decimals.
In a lot of places, it is used for rounding up, but you should note that it is not rounding as intended if the target decimals are greater than the decimals and that the intention is to round up.
The reason is that the implementation of the function which uses the `saturating_mul` which **rounds down** by default.
This correctness error only happens if $decimals_{target} > decimals$ though and that the intention of the caller is to round `UP`.

See the implementation of the function here

```rust
pub(crate) fn normalize_value(amount: Balance, decimals: u8, target_decimals: u8, rounding: Rounding) -> Balance {
	if target_decimals == decimals {
		return amount;
	}
	let diff = target_decimals.abs_diff(decimals);
	if target_decimals > decimals {
		amount.saturating_mul(10u128.saturating_pow(diff as u32))
	} else {
		match rounding {
			Rounding::Down => amount.div(10u128.saturating_pow(diff as u32)),
			Rounding::Up => amount
				.div(10u128.saturating_pow(diff as u32))
				.saturating_add(Balance::one()),
		}
	}
}
```

This, though, doesn't appear to be severe enough to be exploited.
For this to have serious consequences, we would either need some insanely extreme market conditions, or specific pool configurations such as very few liquidity in the pool. But because fees are charged when interacting with the protocol, they swallow this imprecision.
It would be a good choice to make sure that the multiplication result is ceiling if the intent is to round `UP`, though.

## [Low 2] Fee calculations are sometimes rounded towards the wrong way

Fees should profit the protocol and be rounded up to charge the user when normalized. In multiple places, the fee calculated from the amount moved in / out of the pool in rounded down and this will cause events to emit with a fee that is a little bit inferior to the expected value in some cases when the precision adding is applied.

## [Low 3] No deadline parameters for swap operations

Uniswap is an exchange which popularized the "deadline" parameter.
This parameter has been created to avoid people to do bad trades by having them settling in the mempool and once that the gas price become such that miners would pick up this transaction, then the price of the asset would change price and don't capture the intent of the trader anymore.
It may be a good addition to the Omnipool and the Stableswap to avoid these edge cases from happening when the mempool is extremely solicited.

## [Low 4] Cargo audit

The usage of `cargo audit` has found **3 vulnerabilities** as well as **6 warnings** on currently utilized crates. You can find the full output [here](https://gist.github.com/iFrostizz/1915145857edf6ebb199db261f5f12da)
Here is a breakdown of the issues found.

| Crate       | Title                                                                                                                           |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------- |
| h2          | [Resource exhaustion vulnerability in h2 may lead to Denial of Service (DoS)](https://rustsec.org/advisories/RUSTSEC-2024-0003) |
| shlex       | [Multiple issues involving quote API](https://rustsec.org/advisories/RUSTSEC-2024-0006)                                         |
| snow        | [Unauthenticated Nonce Increment in snow](https://rustsec.org/advisories/RUSTSEC-2024-0011)                                     |
| ansi_term   | [ansi_term is Unmaintained](https://rustsec.org/advisories/RUSTSEC-2021-0139)                                                   |
| mach        | [mach is unmaintained](https://rustsec.org/advisories/RUSTSEC-2020-0168)                                                        |
| parity-wasm | [Crate `parity-wasm` deprecated by the author](https://rustsec.org/advisories/RUSTSEC-2022-0061)                                |
| atty        | [Potential unaligned read](https://rustsec.org/advisories/RUSTSEC-2021-0145)                                                    |

You may temporarily replace these crates if it can be done or update if a fix has already been provided.
In the worst case, make sure that none of these vulnerabilities can be triggered from the project.

## [Info 1] Suggested refactors

```diff
diff --git a/math/src/stableswap/math.rs b/math/src/stableswap/math.rs
index 171845b8..e1ca5771 100644
--- a/math/src/stableswap/math.rs
+++ b/math/src/stableswap/math.rs
@@ -733,7 +733,7 @@ pub fn calculate_share_price<const D: u8>(
                let num = num.checked_div(p_diff)?;
                (num, denom)
        };
-       let (num, denom) = round_to_rational((num, denom), crate::support::rational::Rounding::Down);
+       let (num, denom) = round_to_rational((num, denom), Rounding::Down);
        //dbg!(FixedU128::checked_from_rational(num, denom));
        Some((num, denom))
 }
```

```diff
diff --git a/pallets/stableswap/src/lib.rs b/pallets/stableswap/src/lib.rs
index bfb32f65..afae35f7 100644
--- a/pallets/stableswap/src/lib.rs
+++ b/pallets/stableswap/src/lib.rs
@@ -1224,19 +1224,14 @@ impl<T: Config> Pallet<T> {
                                amount: reserve,
                                decimals,
                        });
-                       if let Some(liq_added) = added_assets.remove(pool_asset) {
-                               let inc_reserve = reserve.checked_add(liq_added).ok_or(ArithmeticError::Overflow)?;
-                               updated_reserves.push(AssetReserve {
-                                       amount: inc_reserve,
-                                       decimals,
-                               });
+                       let amount = if let Some(liq_added) = added_assets.remove(pool_asset) {
+                               reserve.checked_add(liq_added).ok_or(ArithmeticError::Overflow)?
                        } else {
                                ensure!(!reserve.is_zero(), Error::<T>::InvalidInitialLiquidity);
-                               updated_reserves.push(AssetReserve {
-                                       amount: reserve,
-                                       decimals,
-                               });
-                       }
+                               reserve
+                       };
+
+                       updated_reserves.push(AssetReserve { amount, decimals });
                }
```

Note that there is a very similar case in the `calculate_shares()` function of the same file (`pallets/stableswap/src/lib.rs`).
## [Info 2] Untested yet critical operations

`omnipool/src/lib.rs`

![Pasted image 20240226011612](https://github.com/iFrostizz/rustry/assets/51274081/2cf210f2-73be-4f32-8a24-d780fb89b773)

![Pasted image 20240226011859](https://github.com/iFrostizz/rustry/assets/51274081/0351bc9d-7f11-4dd3-ac53-d0bf00fd10f4)


`omnipool/src/traits.rs`

![Pasted image 20240226012049](https://github.com/iFrostizz/rustry/assets/51274081/570b8506-2f6a-4e74-b59e-0265b543c6bd)

![Pasted image 20240226012119](https://github.com/iFrostizz/rustry/assets/51274081/c7f91bbb-557e-46ec-a4c6-193a49bf5cea)

`math/src/omnipool/math.rs`

![Pasted image 20240226011245](https://github.com/iFrostizz/rustry/assets/51274081/cfa137a0-3ca4-489c-b0bf-414ed711711d)

`stableswap/src/lib.rs`

![Pasted image 20240226012240](https://github.com/iFrostizz/rustry/assets/51274081/e725fa44-a7fb-4654-99e4-2ea33e8991c8)

![Pasted image 20240226012254](https://github.com/iFrostizz/rustry/assets/51274081/91759306-dab9-4c03-92ec-54b2605d9499)

![Pasted image 20240226012311](https://github.com/iFrostizz/rustry/assets/51274081/a2ac795d-87ad-44b0-ab09-a171f1af2cab)

![Pasted image 20240226012456](https://github.com/iFrostizz/rustry/assets/51274081/9ec201c3-4ac2-45fb-aa5b-1c32dea3fc2f)

`math/src/stableswap/math.rs`

![Pasted image 20240226013431](https://github.com/iFrostizz/rustry/assets/51274081/71e05ce8-8065-44da-8201-d2079e3d43b3)

`math/src/ema/math.rs`

![Pasted image 20240226013626](https://github.com/iFrostizz/rustry/assets/51274081/b0e8249d-2bfa-4671-b427-0473bbd4f25f)

## [Info 3] Dead code

`math/src/omnipool/math.rs`

![Pasted image 20240226011352](https://github.com/iFrostizz/rustry/assets/51274081/00c4d6e8-434c-4051-8be7-2ed4f40e5732)

## [Info 4] Misleading variable name change

The function `calculate_delta_imbalance` is called in two places in the omnipool pallet in the file `pallets/omnipool/src/lib.rs`. Here are those occurences

```rust
			let delta_imbalance = hydra_dx_math::omnipool::calculate_delta_imbalance(
				hub_reserve,
				I129 {
					value: current_imbalance.value,
					negative: current_imbalance.negative,
				},
				current_hub_asset_liquidity,
			)
			.ok_or(ArithmeticError::Overflow)?;
```

```rust
			let delta_imbalance = hydra_dx_math::omnipool::calculate_delta_imbalance(
				asset_state.hub_reserve,
				I129 {
					value: imbalance.value,
					negative: imbalance.negative,
				},
				hub_asset_liquidity,
			)
			.ok_or(ArithmeticError::Overflow)?;
```

The `hub_reserve` is misleading and should probably be renamed to `delta_hub_reserve` since it looks like the variables `hub_reserve` and `current_hub_asset_liquidity` have been swapped in the `calculate_delta_imbalance` function

```rust
pub fn calculate_delta_imbalance(
	delta_hub_reserve: Balance,
	imbalance: I129<Balance>,
	hub_reserve: Balance,
) -> Option<Balance> {
```

The calculation, though is correct, since it conforms to the specs. See the formula in the "Add Tokens" operation

![Pasted image 20240227111337](https://github.com/iFrostizz/rustry/assets/51274081/99f95a2d-1dbc-47c7-891d-37daf755eeb5)