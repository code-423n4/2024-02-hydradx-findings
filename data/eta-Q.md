Addressing Rounding Discrepancies in Fee Calculations in  `math.rs::calculate_fee_amount_for_buy`

## Summary

The protocol notice specifies rounding fees up, contrary to the current practice using the div() in `math::calculate_fee_amount_for_buy`.

## Vulnerability Details

During fee calculations, it has been observed that the fees are rounded down using the `div()` method. However, the protocol notice specifies that fees should be rounded up. This discrepancy could potentially lead to exploitation of the system, albeit insignificantly. 

To maintain consistency and adhere to the protocol notice, it is recommended to use `div_ceil` instead of `div()` for fee calculations. This ensures that fees are rounded up, thus providing better protection against any potential system exploitation, as mentioned in the notice.

[math::calculate_fee_amount_for_buy](https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/math/src/omnipool/math.rs#L145C1-L149C34)
```Rust
pub(crate) fn calculate_fee_amount_for_buy(fee: Permill, amount: Balance) -> Balance {
	if fee.is_zero() {
		return Balance::zero();
	}
	if fee == Permill::one() {
		return amount;
	}

	let (numerator, denominator) = (fee.deconstruct() as u128, 1_000_000u128);
	// Already handled but just in case, so div is safe safe. this is 100%
	if numerator == denominator {
		return amount;
	}
	// Round up
+	numerator
+		.saturating_mul(amount)
+		.div_ceil(denominator.saturating_sub(numerator))
+		.saturating_add(Balance::one())

-	numerator
-		.saturating_mul(amount)
-		.div(denominator.saturating_sub(numerator))
-		.saturating_add(Balance::one())

}
```

## Impact
The contradiction to round against the user to prevent any potential system exploitation as mentioned in the notice.

## Recommendation
It is advised to use `div_ceil()` to round up the fee as shown in Vulnerability Details.