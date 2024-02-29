


# Issue 1: HALF_LIFE_IN_BLOCKS Parameter Adjustment
## Description
 The HALF_LIFE_IN_BLOCKS parameter (https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/ema-oracle/src/lib.rs#L30) used to calculate the EMA determines the aging factor. However, this parameter may need regular evaluation, as changes in network conditions (e.g., slower or faster block times) can lead to inaccurate EMA values.

## Impact
 Inaccurate EMA values due to a misconfigured aging factor can lead to a mismatch between current market conditions and the EMA Oracle values, potentially exposing users to financial losses.

## Proof of Concept

- If the network conditions change but the HALF_LIFE_IN_BLOCKS parameter remains unchanged, the EMA calculation may not provide accurate results.
- Overestimated or underestimated aging factors misrepresent EMA values, affecting their relevance to current market conditions.


## Recommendation:

-  Establish a monitoring system for network conditions to identify changes in block times.
- Regularly evaluate and adjust the HALF_LIFE_IN_BLOCKS parameter based on the observed network conditions to ensure accurate EMA calculations.






# Issue 2: Front-Running Attack in Deposit Function
## Description
 The deposit function of the omnipool (https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/omnipool/src/lib.rs#L1036) does not have a minimum deposit amount and lacks privacy-preserving solutions. Consequently, an attacker can monitor incoming transactions and manipulate their deposit amounts, securing a more favorable exchange rate, which leads to an attack on the integrity of the liquidity pool.

## Impact
 Front-running can lead to financial losses for users attempting to make deposits while allowing attackers to gain a potentially unfair advantage in the exchange rates.

## Proof of Concept:

1 . A user initiates a deposit at a specific rate (R1).
2 . An attacker observes the incoming transaction and quickly submits a deposit at the same rate (R1).
3 . The two deposits conflict, and the attacker's deposit with a later timestamp is processed first.
4 . The attacker's deposit is executed, changing the exchange rate or liquidity pool composition, which adversely affects the original user's intended transaction.


## Recommendation

- Implement a minimum deposit amount, protecting users from potential front-running attacks and making it more difficult for attackers to manipulate exchange rates.
- Integrate privacy-preserving solutions that hide or obfuscate user transactions to prevent attackers from monitoring and manipulating the system. This can help deter observation-based attacks and preserve the integrity of deposit processes.