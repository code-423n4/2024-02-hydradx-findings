## Table of Contents:

1. **Overview**
   - Introduction to HydraDX's decentralized trading infrastructure.
   - Core components such as AMMs, oracles, and bridges.
   
2. **Auditing Approach**
   - Methodology overview covering architecture analysis, safety assessments, and economic model evaluation.
   - Utilization of whitebox static analysis and grey box techniques.

3. **Strengths**
   - Strong mathematical foundations with protected calculations.
   - Access-controlled state transitions for risk mitigation.
   - Extensive testing coverage including unit and integration tests.
   - Modular architecture enabling independent analysis.
   - Custom optimizations like efficient pooled AMM designs.

4. **Weaknesses**
   - Risks from complex integrations.
   - Potential sustainability issues with dynamic fee models.
   - Scope for additional verification of custom algorithms.
   - Lack of negative test cases and fuzzing in certain areas.

5. **Contract Summaries**
   - **Omnipool Pallet**: Features and capabilities of the custom AMM model.
   - **Stableswap Pallet**: Overview of the Curve style stableswap AMM protocol.
   - **EMA Oracle Pallet**: Decentralized on-chain price oracle solution using exponential moving averages.
   - **Circuit Breaker Pallet**: Implementation of security circuit breakers to halt operations based on risk parameters.

6. **Architecture Analysis**
   - **Business Logic**: Key workflows and governing parameters.
   - **Composition**: Overview of hardened core logic components and dependencies.
   - **Component Coupling**: Interactions, separation of concerns, and extensibility mechanisms.

7. **Code Quality**
   - Adherence to Rust standards and code quality.
   - Isolation of math into hardened libraries.
   - Logical partitioning into pallets aiding analyzability.
   - Testing strategy for critical paths.

8. **Admin Controls**
   - Key admin roles and governance council.
   - Administration actions and related vulnerabilities.

9. **Related Bugs and Issues**
   - Possible attacks and vulnerabilities related to governance and upgrades.

10. **Mitigations**
    - Strategies to mitigate vulnerabilities and risks.

11. **Recommendations**
    - Suggestions for improvement including refactoring, error handling, and architectural patterns.

## Overview

HydraDX is building a decentralized trading infrastructure enabling cross-chain liquidity and asset exchanges. The modular runtime comprises of core components like AMMs, oracles, bridges etc.

## Auditing Approach

The audit methodology covered - architecture analysis, data and control flow assessments, math safety analysis, economic model viability evaluation etc across the codebase. Both whitebox static analysis and grey box techniques employed.

## Strengths

1. Strong mathematical foundations with overflow protected calculations.
2. Access controlled state transition functions minimize risks. 
3. Extensive unit and integration test coverage across implementations.
4. Sound modular architecture enables independent analysis.
5. Custom optimizations like efficient pooled AMM designs.

## Weaknesses

1. Complex integrations across modules can induce risks from errors.
2. Dynamic fee model can lead to loss of sustainability if misconfigured. 
3. Scope for additional verification of math intensive custom algorithms.
4. Lack of negative test cases and fuzzing in certain areas.


## Contract Summaries

**Omnipool Pallet**

Implements the custom Omnipool automated market maker (AMM) model allowing swaps between any token pairs through pooled liquidity. Key capabilities:

- Manages the pooled liquidity reserves across deposited tokens.
- Handles minting and burning of pool share tokens to users providing liquidity. 
- Settles token swaps by accounting for price impact, fees, slippage thresholds etc.
- Supports pluggable hooks to integrate external logic on operations.

**Stableswap Pallet** 

Implements a Curve style stableswap AMM protocol optimized for minimal slippage across stablecoin pairs. Core functions:

- Provides factory contract to create new stableswap pools and related share tokens.
- Manages deposit and withdrawal of liquidity into created pools.
- Calculates swaps between the assets in the pools based on the invariant formula.
- Allows dynamic runtime configurability of trading fees and amplification factors.

**EMA Oracle Pallet**

Provides a decentralized on-chain price oracle solution using exponential moving averages customized to needs of AMMs. Key capabilities:

- Tracks time weighted average prices and related metrics across different window periods.
- Aggregates and filters data from callback hooks of other pallets like AMMs.
- Configurable oracle smoothing factors and supported averaging periods.
- Handles lazy updates to mitigate sandwich attacks on oracle consumers.

**Circuit Breaker Pallet** 

Implements security circuit breakers to halt operations based on monitoring various risk parameters. Main functions:  

- Pauses activity if liquidity or price fluctuations breach defined threat thresholds per token pair.  
- Whitelists trusted actors exempted from circuit breaker actions.
- Works in conjunction with other pallets to take risk mitigating actions.

## Architecture Analysis

**Business Logic**

*Key Workflows*

- Core business processes centred around liquidity provision, trading, settlement and data feeds.

- Handled via key pallets - Omnipool, Stableswap, EMA Oracle etc plus associated libraries.

- Governing of parameters and treasury administration through Governance pallet.

*Composition*  

- Hardened core logic components and isolated reusable libraries.

- Dependencies limited to essential underlying capabilities like token transfers.

- Avoidance of unnecessary complexity or intermediary layers.


**Component Coupling**

*Interactions*

- Components interact via dispatchable calls and events for transactions.

- Oracle pallet provides necessary price feed data into the AMM pools.

- Tokens moved across platform boundaries using bridges.

*Separation of Concerns*

- Clear segregation between business logic, storage, token handling etc via pallet boundaries.

- Algorithmic complexity and custom traits abstracted into libraries preventing leakages.

- Explicit trait interfaces lower coupling across components.


**Extensibility Mechanisms** 

*Evolution*

- Components don't have interdependencies allowing independent upgrades.

- Leverages forkless runtime upgrades of Substrate for non-breaking changes.

*Modifiability*

- Traits allow adding new providers without changing implementations.

- Events and hooks enable reactive custom logic integration minimizing intrusion.


**Summary**

HydraDX demonstrates careful considerations across business flows, coupling and extensibility dimensions in its architecture.

## Code Quality**

1. Excellent adherence to Rust standards bolstering quality and security. 
2. Math isolated into hardened libraries establishing trust.
3. Logical partitioning into pallets aids independent analyzability. 
4. Testing strategy provides reasonable confidence on critical paths.

## Admin Controls

- Key admin roles are the root account and the governance council originating privileged extrinsics.

- The council membership adding/removal controls the governance power concentration.

- No bake-in time delays on admin actions or multi-sig based approvals by default.

## Related Bugs and Issues

- Possibility of governance proposal attacks by malicious council members.

- Lack of protection against rapid mass migrations by the root.

- Upgradability allows forceful runtime upgrades without decentralized consensus. 

## Tests

**Setting up Rust/Substrate environment**

**Getting the code**
**Clone this repository**
```
git clone https://github.com/code-423n4/2024-02-hydradx/
```

Enter into the directory

```
cd HydraDX-node
```

Running pallet tests
Omnipool

```
cargo test -p pallet-omnipool 
```
Stableswap

```
cargo test -p pallet-stableswap 
```
EMA Oracle

```
cargo test -p pallet-ema-oracle 
```
Circuit breaker

```
cargo test -p pallet-circuit-breaker 
```

Running math tests
You can focus on math for each pallet separately.

Omnipool math

```
cargo test omnipool -p hydra-dx-math
```

Stableswap math

```
cargo test stableswap -p hydra-dx-math
```

EMA Oracle math

```
cargo test ema -p hydra-dx-math
```

**Running integration tests**
These tests focus on integration of a pallet in HydraDX runtime, interactions with other pallets and configuration.

```
cargo test -p runtime-integration-tests
```

## Resulting Vulnerabilities

- Parameter modifications through proposals can indirectly induce economic attacks.

- Central governance group could push proposals helping their interests.

- Governance module configuration changes can disable security controls like circuit breakers.

- No protection against rapid value withdrawals by privileged accounts to exchanges.

**Mitigations

- Time delays, multi-sig approvals before sensitive admin actions like migrations, upgrades.

- Vesting schedules limiting withdrawal volumes for privileged accounts.

- Decentralized off-chain governance processes to establish proposal intents. 

- Parameter bound verifications for critical configurations.

Additional decentralized governance mechanisms can help mitigate excessive centralization risks in admin capabilities.

## Recommendations

1. Refactoring complexity hotspots into separate child pallets.  
2. Custom errors to avoid leaky abstractions across call stacks.
3. Architectural patterns to lower inter-component coupling.
4. Expanded negative test cases.
5. Formal verification of custom algos where feasible.

### Time spent:
39 hours