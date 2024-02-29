-   [1. System Overview](#1-system-overview)
    -   [Stableswap](#stableswap)
    -   [Omnipool](#omnipool)
-   [2. System Mechanics](#2-system-mechanics)
    -   [Stableswap](#stableswap-1)
    -   [Omnipool](#omnipool-1)
        -   [A-B swaps](#a-b-swaps)
        -   [LRNA-A swaps](#lrna-a-swaps)
-   [3. Threat Modelling](#3-threat-modelling)
-   [4. Centralization Risks](#4-centralization-risks)
-   [5. Further Security Advice](#5-further-security-advice)
    -   [Stableswap](#stableswap-2)
    -   [Omnipool](#omnipool-2)

# 1. System Overview

HydraDx is an automated market maker in the Polkadot ecosystem and is deployed as a parachain designed to bring liquidity to the Polkadot ecosystem. For this purpose there are two separate AMMs in the scope of this audit: Stableswap and Omnipool.

## Stableswap

Stableswap is a fork of the Curve protocol, re-written in rust and designed to be deployed on the Polkadot ecosystem. Similar to the curve protocol, it is a liquidity pool with variable slippage, and is better for assets with correlated values, such as stablecoins.

## Omnipool

Omnipool is a new take on constant product market maker (CPMM) algorithms, and acts similar to the Uniswap protocol, but supports multiple tokens and allows single sided liquidity deposits. The aim of the system is to provide more liquidity between assets within a pool which have beend etermined to have similar risk factors/volatility. This way the system can provide liquidity to multiple pairs simultaneously without fracturing the liquidity.

The system is designed on substrate with pallets, with separate pallets responsible for the different AMMs, tokens as well as the runtimes. Using this architecture allows the protocol to finely control the intricacies of the blockchain environment itself.

In this audit, the following pallets are being Audited:

-   `stableswap`
-   `omnipool`
-   `math`
-   `ema-oracle`
-   `circuit-breaker`

The first three pallets make up the two different AMMs described above. `ema-oracle` is an implementation of an exponential moving average oracle to evaluate time-weighted prices in the pool and act as an oracle. `circuit-breaker` is a pallet that allows the system to pause operations if the system goes outside the defined limits for trading, liquidity etc.

Here we will focus on the two AMMs for the most part, as they are the most complex and critical parts of the system.

# 2. System Mechanics

## Stableswap

Stableswap is a curve protocol fork written in rust for the Polkadot ecosystem. Similar to curve, the stableswap AMM is basically a linear combination of a constant product market maker (CPMM) and a constant sum market maker (CSMM). The system is designed to be better for assets with correlated values, such as stablecoins. The amplification factor `A` determines the weight of the CPMM or the CSMM part, and thus determines the slippage of the pool.

Stableswap also implements some extra functions on top of the curve protocol, allowing liquidity addition in terms of shares and liquidity removal in terms of asset amount.

Stableswap removes some functionality from the curve protocol as well, mainly disallowing the ability to remove liquidity in multiple tokens at once. The protocol only allows for single asset removals. This creates an issue since single asset removals always changes the price in the pool and creates MEV opportunity, giving worse rates to the liquidity provider. This is because on curve, single sided liquidity addition or removal is equivalent to a trade where the single asset is traded for the other assets in the pool incurring some slippage and then adding liquidity. For removals the opposite is done. Thus there is a large swap involved, which creates MEV opportunity and is simply inefficient.

While the protocol supports multi-asset liquidity addition, it does not support multi-asset liquidity removal. This is a design decision, but it is important to note that this is a significant difference from the curve protocol and leads to inefficiencies.

## Omnipool

The omnipool is a CPMM similar to uniswap or balancer, with the main difference being that the liquidity addition and removal happen in a single asset. This asset when deposited is paired with some LRNA tokens which are minted on demand at a specified price. So the liquidity pool holds multiple tokens, as well as LRNA or hub tokens which have been minted on demand and each token has an amount of hub tokens associated with it.

The immediate advantage of this design is that it allows for single sided liquidity operations, and increases the depth of liquidity in the pools. Instead of splitting liquidity of token A into an A-B pool and an A-C pool, thus leading to higher slippage, they can all be combined in a single A-B-C pool with the LRNA token acting as a bridge between the different assets.

A number of issues were discovered in this model, mainly relating to MEV and have been reported. In this section, I would like to present a spreadsheet based model for rough calculations of swaps in the omnipool, which can better help understand the core mechanics of the pool, as well as better explain some of the POCs in the submitted reports.

Lets assume tokens A and B are both priced at 1 usd, and the hub token LRNA is also priced at 1 usd. If we assume that the protocol starts with 20 A tokens and 20 B tokens, the protocol will mint 20 LRNA tokens corresponding to the 20 A tokens and similarly 20 LRNA tokens corresponding to the 10 B tokens. The initial liquidity composition of the pool will look like so:

| Operation | Pool tokenA | Pool LRNA for tokenA | Pool LRNA total | Pool LRNA for tokenB | Pool tokenB | pA  | pB  | pAB |
| --------- | ----------- | -------------------- | --------------- | -------------------- | ----------- | --- | --- | --- |
| Initial   | 20          | 20                   | 40              | 20                   | 20          | 1   | 1   | 1   |

pA is the spot price of LRNA-tokenA, calculated by column_3/column_2.
pB is the spot price of LRNA-tokenB, calculated by column_5/column_6.
pAB is the spot price of tokenA-tokenB, calculated pB/pA.

Different swaps have different effects on the pool. Since buying LRNA tokens from the pool is not allowed, there are only two possible swaps: LRNA-A swaps and A-B swaps.

### A-B swaps

In this scenario, Alice sells B token for A token. The price of A is raised. Now, no of A tokens and no of B tokens must maintain the invariant, while the corresponding hub token amounts remain untouched. Alice sells 5 B tokens, so the reserves grow to 25 B tokens and 16 A tokens.

| Operation | Pool tokenA | Pool LRNA for tokenA | Pool LRNA total | Pool LRNA for tokenB | Pool tokenB | pA   | pB  | pAB  |
| --------- | ----------- | -------------------- | --------------- | -------------------- | ----------- | ---- | --- | ---- |
| Initial   | 20          | 20                   | 40              | 20                   | 20          | 1    | 1   | 1    |
| B -> A    | 16          | 20                   | 40              | 20                   | 25          | 1.25 | 0.8 | 0.64 |

Some observations:

1. pAB has gone down. Since A is being bought, the price of A has gone up wrt B.
2. pA has gone up while pB has gone down. This isbecause the LRNA tokens are untouched while one external token is devalued and the other is bought up. An important point to note, is that in A-B swaps, `pA * pB` is an invariant, neglecting fees. This is evident from this scenario, since `1.25*0.8=1.0`. This is a direct product of the x\*y=k invariant.
3. An arbitrage opportunity is created. Since pAB is not zero, it means the pool pays out more tokenA when tokenB is sold.

### LRNA-A swaps

In this scenario, LRNA is sold for token A. Since this is a CPMM, it means LRNA corresponding to A and the number of token A will be related by the x\*y=k invariant.
Let's assume Alice sells 5 LRNA tokens and ignore any fees. Thus `Pool LRNA for tokenA` becomes 25. Similarly, pool's tokenA then becomes `=20*20/25=16` following the invariant. So Alice receives 4 A tokens from the pool.

| Operation | Pool tokenA | Pool LRNA for tokenA | Pool LRNA total | Pool LRNA for tokenB | Pool tokenB | pA     | pB  | pAB  |
| --------- | ----------- | -------------------- | --------------- | -------------------- | ----------- | ------ | --- | ---- |
| Initial   | 20          | 20                   | 40              | 20                   | 20          | 1      | 1   | 1    |
| LRNA -> A | 16          | 25                   | 45              | 20                   | 20          | 1.5625 | 1   | 0.64 |

Some observations:

1. pAB has gone down. By buying up A, we have increased its worth wrt B. So the price of B has gone down in terms of A.
2. pA has gone up. This is because LRNA is sold and thus devalued.
3. An arbitrage opportunity is created. Since pAB is not zero, it means the pool pays out more tokenB when tokenA is sold.

Evaluating the arbitrage opportunity is quite simple. The arbitrager will sell token A and buy token B. As shown in point 2 in the A-B swap, in A-B swaps the `pA*pB` stays the same. For the final price `pAB` to equal 1, `pA'=pB'`. So the final prices `pA'` and `pB'` will be `=sqrt(1.5625*1)=1.25`. Since LRNA tokens remain untouched in A-B swaps, the arbitraged state is shown below. The return

| Operation  | Pool tokenA | Pool LRNA for tokenA | Pool LRNA total | Pool LRNA for tokenB | Pool tokenB | pA     | pB   | pAB  |
| ---------- | ----------- | -------------------- | --------------- | -------------------- | ----------- | ------ | ---- | ---- |
| Initial    | 20          | 20                   | 40              | 20                   | 20          | 1      | 1    | 1.0  |
| LRNA -> A  | 16          | 25                   | 45              | 20                   | 20          | 1.5625 | 1    | 0.64 |
| Arbitraged | 20          | 25                   | 45              | 20                   | 16          | 1.25   | 1.25 | 1.0  |

These swap tables were used to simulate transactions on excel (sans fees) and is also used in the POCs of a couple submissions.

# 3. Threat Modelling

Mainly these threats were investigated in the audit:

1. Bugs / faulty math in the AMMs
2. Programming issues
3. Economic vulnerabilities
4. MEV / frontrunning / sandwich attacks

Some Bugs and programming issue were found, which have been reported.

Economic vulnerabilities were studied using the excel table method described above. This method removes fees from the calculations and is thus an `optimistic` scenario for value extraction.

The results who that if users are able to token->LRNA swaps, they can extract value from the pool. This is currently turned off in the system, but if it is turned on in the future for any reason, the algorithm needs to be thoroughly verified to prevent abuse of the system.

This method also showed that the `ensure_price` method in the code can be violated. This was an important finding since this allowed arbitrary amount of slippage losses to the user. This was discovered based on a few observations:

1. `ensure_Price` only looked at `pA` from the table above
2. `pA` increases in LRNA-A swaps, and decreases in A-B swaps

Further economic analyses were done to try and profit from the system, and situations where small profits can be gained risk-free have been reported.

For studying MEV, the same approach was used, but with two actors. The objective was to increase the value of the attacker while decreasing the value of the user or the pool. Multiple combinations of transactions were studied with the excel table method, and any results where the attacker gained value were reported.

This analysis was also done on admin-only functions, and some situations where even the admin could extract value from the pool were reported.

# 4. Centralization Risks

The system is permissionless. There are no black/whitelists involved, and admins cannot forbid any user from participating in the system.

However, the system is centralized. Tokens have individual permissions for removing and adding liquidity, as well as buying or selling. Centralization can be reduced if the various origins, like authority origin and technical origin are voting contracts with timelocks.

# 5. Further Security Advice

## Stableswap

This pallet can benefit more from a thorough fuzz test. The pallet is pretty similar to the curve protocol, however there are some extra rounding operations here and there. An example can be seen [here](https://github.com/code-423n4/2024-02-hydradx/blob/603187123a20e0cb8a7ea85c6a6d718429caad8d/HydraDX-node/math/src/stableswap/math.rs#L498-L502). There is an assumption here that the rounding is sufficient provided there's more than 2 wei of liquidity. The issue is that assumptions and small roundings like these are extremely difficult to validate manually. This is especially true in the curve protocol, there the solution does not come from a simple equation, but is actually converged upon by newton-raphson iterations. Thus the best way to validate these are either via formal verification (too expensive maybe) or via a robust and heavy fuzzing/invariant test suite. While there are proptests in the test suite with 1000 iterations, they are only good for checking the protocol under normal operations. These fuzz tests are insufficient to hit the edge cases which really trigger the rounding errors.

I am also unfamiliar with the robustness of rust fuzzers, but industry leading `smart`-fuzzers like medusa, echidna etc which try to optimise code coverage and edge cases are the way to go. The pallet can heavily benefit from a good fuzzing suite with such fuzzers which also do random operations on the state, and not just random inputs.

For this module, I would advise the devs to build some test cases specifically catered to hitting edge cases, like low liquidity conditions, or single/few wei operations. These are the conditions where the rounding errors are most likely to occur. Another option would be to maybe contact a team with robust fuzzing experince to build a fuzzing suite.

## Omnipool

The omnipool math is easier to deal with, since its the same old x\*y=k invariant. However, this module can also benefit from fuzz tests with random operations on the state. Since the math of this module is very close to uniswap, a battle-tested mechanism, this is less prone to breaking under such edge cases. The test suite in omnipool is too `optimistic`, which is good for normal operations, but does not test heavily skewed pool conditions with very extreme prices.

The test suite and the protocol was also built with the intention of stopping MEV bots from making a profit as the first priority. This is a good design, however, this is unfortunately not enough. If a user is able to cause harm to another user at a loss to themselves, that is still a valid issue since it impacts the second user. The protocol should be designed to prevent such situations as well. This is evident from missing slippage protections, and loose price checks, which will prevent attackers from profiting after considering the fees, but can still impact other users negatively.

For a robust system, the protocol should instead ensure that users or the pool cannot be harmed, or doesn't leak value. This will always ensure that attackers cannot profit, since the value has to come from somewhere.

Hours spent: 120


### Time spent:
120 hours