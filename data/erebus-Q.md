# Non-critical

## [NC-01] Unreachable code

In

[**circuit-breaker/src/lib.rs, functions check_\*_limit**](https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/pallets/circuit-breaker/src/lib.rs#L66C1-L86C3)

```rs
	pub fn check_outflow_limit(&self) -> DispatchResult {
		if self.volume_out > self.volume_in {
			let diff = self
				.volume_out
				.checked_sub(&self.volume_in)
				.ok_or(ArithmeticError::Underflow)?;
			ensure!(diff <= self.limit, Error::<T>::TokenOutflowLimitReached);
		}
		Ok(())
	}

	pub fn check_influx_limit(&self) -> DispatchResult {
		if self.volume_in > self.volume_out {
			let diff = self
				.volume_in
				.checked_sub(&self.volume_out)
				.ok_or(ArithmeticError::Underflow)?;
			ensure!(diff <= self.limit, Error::<T>::TokenInfluxLimitReached);
		}
		Ok(())
	}
```

reverts if the diference between the volumes in and out underflows. However, it is redundant as they are doing the substraction if `a > b`, which cannot underflow. Consider doing the substraction as is, instead of using the safe version.