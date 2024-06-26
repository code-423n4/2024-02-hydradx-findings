# [L-01] Incorrect rounding in `calculate_out_given_in` and `calculate_in_given_out`

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/math/src/stableswap/math.rs#L38
https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/math/src/stableswap/math.rs#L69

In the Stableswap math's `calculate_out_given_in`, the user provides an amount of `in` token and the function calculates the corresponding `out` token.

When `in` has to be adjusted to a lower number of decimals, it is rounded up:
```
File: math.rs
23: pub fn calculate_out_given_in<const D: u8, const Y: u8>(
---
29: ) -> Option<Balance> {
---
34: 	let amount_in = normalize_value(
35: 		amount_in,
36: 		initial_reserves[idx_in].decimals,
37: 		TARGET_PRECISION,
38: 		Rounding::Up,
39: 	);
```
this rounding causes the protocol to calculate the output for an input potentially higher than what the user actually provided.

Similarly, in `calculate_in_given_out`, the given `out` is rounded down, causing the math to calculate the `in` tokens corresponding to a lower amount than what the user will get, again going in the protocol's disfavor:

```
File: math.rs
54: pub fn calculate_in_given_out<const D: u8, const Y: u8>(
---
60: ) -> Option<Balance> {
---
65: 	let amount_out = normalize_value(
66: 		amount_out,
67: 		initial_reserves[idx_out].decimals,
68: 		TARGET_PRECISION,
69: 		Rounding::Down,
70: 	);
```

Consider changing these roundings to be in the protocol's favor.

# [L-02] Unsafe usage of `saturating_mul` in Stableswap math's `normalize_value`

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/math/src/stableswap/math.rs#L635

In the Stableswap math's `normalize_value` function, when upscaling an amount (`target_decimals > decimals`), the given input is multiplied by a power of 10 using `saturating_mul`. This can be exploited to artificially deflate the output amount versus the input amount.

```
File: math.rs
629: pub(crate) fn normalize_value(amount: Balance, decimals: u8, target_decimals: u8, rounding: Rounding) -> Balance {
630: 	if target_decimals == decimals {
631: 		return amount;
632: 	}
633: 	let diff = target_decimals.abs_diff(decimals);
634: 	if target_decimals > decimals {
635: 		amount.saturating_mul(10u128.saturating_pow(diff as u32))
636: 	} else {
637: 		match rounding {
638: 			Rounding::Down => amount.div(10u128.saturating_pow(diff as u32)),
639: 			Rounding::Up => amount
640: 				.div(10u128.saturating_pow(diff as u32))
641: 				.saturating_add(Balance::one()),
642: 		}
643: 	}
644: }
```

Even though this is unlikely to be exploited in a real-world scenario where this logic is integrated with safe token transfers, consider using a `checked_mul` instead of `saturating_mul` at L635.

# [L-03] Stableswap amplification changes can cut previous ramps short

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/stableswap/src/lib.rs#L426-L438

Amplification changes are applied to stableswap pools with a linear interpolation of amplification of values over time. 

Whenever a new amplification value is set, the Stableswap logic freezes the current amplification until `start_block` is reached:

```
File: lib.rs
408: 		pub fn update_amplification(
---
414: 		) -> DispatchResult {
---
433: 				pool.initial_amplification =
434: 					NonZeroU16::new(current_amplification.saturated_into()).ok_or(Error::<T>::InvalidAmplification)?;
435: 				pool.final_amplification =
436: 					NonZeroU16::new(final_amplification).ok_or(Error::<T>::InvalidAmplification)?;
437: 				pool.initial_block = start_block;
438: 				pool.final_block = end_block;
```

This can cut short a pre-existing ramp, freezing entirely its evolution between in the time between the `update_amplification` and `start_block`.
Consider letting the pre-existing ramp evolve until `start_block` is reached, and only at this point, freeze the previous ramp's current value to become the new ramp's `initial_amplification`.

# [L-04] Token transfers to stableswap pools are not filtered

https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/runtime/hydradx/src/system.rs#L65-L87

The HydraDX runtime configuration prevents token transfers to the Omnipool account. For stableswap pools no similar measure is taken, allowing anyone to transfer tokens to stableswap pool accounts, and consequently altering the swap spot price.

While this issue is per se low-risk (because the account transferring tokens would lose them to arbitrageurs and liquidity providers), it can be leveraged to increase the severity of an attack based on other vulnerabilities.