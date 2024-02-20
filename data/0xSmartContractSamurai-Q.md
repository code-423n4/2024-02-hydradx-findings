0. QA/LOW: `2024-02-hydradx/HydraDX-node/math/src/ema/math.rs::multiply()` - comments not in line with code implementation below them.

https://github.com/code-423n4/2024-02-hydradx//blob/fadcacc8a90da0166d16f62305e6218fd81ed788/HydraDX-node/math/src/ema/math.rs#L214-L217

```rust
	// n = l.n * f.to_bits /// @audit-issue comment not in line with code implementation below
	let n = r_n.full_mul(U256::from(f.to_bits()));
	// d = l.d * DIV /// @audit-issue comment not in line with code implementation below
	let d = r_d.full_mul(U256::from(crate::fraction::DIV));
```
=======
=======

1. QA/LOW: `2024-02-hydradx/HydraDX-node/pallets/omnipool/src/lib.rs::add_token()` - A potential redundancy in the `ensure!` statement that checks for `amount > 0`, given that `T::MinimumPoolLiquidity::get()` is guaranteed to be greater than zero.

https://github.com/code-423n4/2024-02-hydradx//blob/fadcacc8a90da0166d16f62305e6218fd81ed788/HydraDX-node/pallets/omnipool/src/lib.rs#L471-L474

If `amount >= T::MinimumPoolLiquidity::get()` and since `T::MinimumPoolLiquidity::get()` is guaranteed to be greater than zero, it follows that `amount` MUST be greater than zero too, which makes the second check `&& amount > 0` redundant.

Recommendation:

```diff
	ensure!(
-		amount >= T::MinimumPoolLiquidity::get() && amount > 0,
+		amount >= T::MinimumPoolLiquidity::get(),
		Error::<T>::MissingBalance
	);
```
=======
=======

