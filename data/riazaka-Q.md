In the Cargo.toml file, the overflow-checks=true flag is not set, which means overflow checks are disabled by default in optimized release builds. This can lead to unexpected behavior if an overflow occurs

 Links:
 - https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/Cargo.toml
 - https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/Cargo.toml
 - https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/Cargo.toml
 - https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/circuit-breaker/Cargo.toml