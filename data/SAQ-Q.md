## [L-1] Zero Liquidity Warning

https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/lib.rs#L397C1-L403C4 

#### File: HydraDX-node/pallets/ema-oracle/src/lib.rs

### Vulnerability Details
The code contains a comment and a warning log indicating that zero liquidity values are assumed to be invalid, and in such cases, a warning is logged. However, the code does not explicitly handle this situation, and it returns an error without further action. This might be considered a low-risk issue as it involves a warning message and assumes zero liquidity is an exceptional case.

```rust
// We assume that zero liquidity values are not valid and can be ignored.
if liquidity_a.is_zero() || liquidity_b.is_zero() {
    log::warn!(
        target: LOG_TARGET,
        "Liquidity amounts should not be zero. Source: {source:?}, liquidity: ({liquidity_a},{liquidity_b})"
    );
    return Err((Self::on_trade_weight(), Error::<T>::OnTradeValueZero.into()));
}
```
### Recommended Mitigation:
* **Clarify the Handling**: Provide additional comments or code to clarify how zero liquidity values should be handled, or if they are expected in certain scenarios.

* **Explicit Handling**: Consider explicitly handling the case of zero liquidity, providing appropriate actions or checks.

* **Documentation**: Update relevant documentation to indicate the expected behavior or conditions regarding liquidity values being zero.

* **Consider Business Logic**: Verify if zero liquidity is expected based on the business logic and adjust the code accordingly.