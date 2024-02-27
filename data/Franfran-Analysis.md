## Summary

[HydraDX](https://hydradx.io/) is a DEFI protocol on the Polkadot blockchain which promises to solve many issues of the current protocols such as Impermanent loss, expensive interactions, high slippage, and security at the protocol level by leveraging Polkadot parachains.
Parachains can be thought as specialized blockchains that can be deployed and are validated by the consensus layer of Polkadot, namely the relay chain.
They allow an unparalleled freedom to developers because applications are not limited anymore by the rules that have been defined by the application layer and can now go out-of-the sandbox and define their own rules. But with great powers comes great responsibilities: the halting problem is one issue that may compromise the security of the parachain, since they define their own weights (which is analogous to the gas on Ethereum). If these rules are not correctly defined or that the state bloat is not controlled, then the chain may quickly either become unusable or vulnerable to DOS attacks.
Smart contract are written by the use of contract pallets, which is an utility for writing smart contract in Rust offered by Substrate.

## Approach

- Getting more familiar with the docs of Substrate https://docs.substrate.io/reference/rust-api/ as well as existing tools from the Polkadot ecosystem such as https://assethub-polkadot.subscan.io/ to explore and know more about other parachains, which are crucial for understanding how nodes work
- Learning more about HydraDX though their docs https://docs.hydradx.io/
- Setup of tools that will be useful to dive into the code such as [`rust-analyzer`](https://rust-analyzer.github.io/) as well as doing specific operations. Launching of [`cargo audit`](https://crates.io/crates/cargo-audit), `cargo test` and use of [`tarpaulin`](https://github.com/xd009642/tarpaulin) in order to get the coverage of tests
- Line-by-line manual review of the 4 systems in scope: Omnipool, Stableswap, EMA oracle, and the circuit breaker while looking at [invariants](https://github.com/galacticcouncil/HydraDX-security/tree/main/invariants) and the mathematical models expressed by [past audit reports](https://github.com/galacticcouncil/HydraDX-security/tree/main/audit-reports)
- Diff of each part of the code from the last commit of each audits fixes to highlight new, unaudited changes. `git blame` is useful to find culprit commits and their underlying PRs

### Cargo audit

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

### Test coverage

 ```sh
cargo tarpaulin --tests -t 9999 --out lcov
```

Note that this is a very long process !

With the output

```sh
59.33% coverage, 8851/14917 lines covered
```

```sh
# convert the lcov report to html
genhtml lcov.info -o coverage
```

Also note that these results may not be accurate, since the coverage for macros cannot be done without [instrumentation](https://github.com/xd009642/tarpaulin?tab=readme-ov-file#procedural-macros) of the code.

Nonetheless, a report has been uploaded on [github](https://github.com/iFrostizz/hydradx-coverage)

The above command may be specialized to the audit scope in order to cut a bit on the generation time

```sh
cargo tarpaulin -t 9999 --out lcov -p pallet-circuit-breaker -p pallet-omnipool -p pallet-stableswap -p pallet-ema-oracle -p hydra-dx-math
```

By looking at the coverage, we can see that most conversion trait implementations are untested.
If these branches are never used, you should consider removing them or adding a simple unit test or completely ignoring coverage for these functions by the use of attributes such as `#[cfg(not(tarpaulin_include))]` or `#[no_coverage]`. For more information, see [Ignoring code in files](https://github.com/xd009642/tarpaulin?tab=readme-ov-file#ignoring-code-in-files).

### Risks

Pallets are by nature upgradeable without the need of a hard fork. 
Upgrades actioned by the council are democratic since people can vote on [opengov](https://polkadot.polkassembly.io/opengov), but they may vastly change the logic of pallets. Great care has to be taken when reviewing these changes. 


Here is a list of potentially risky operations that can be executed by the privileged parties
#### Adding a new token to the Omnipool / Create a stableswap pool
##### Role 
Authority
##### Risk
Only the authority is able to add tokens to the pool. This is to make sure that malicious tokens cannot be added by anyone to steal shares from the protocol because the price of the asset is arbitrarily set that the time of the operation.
Having a democratic governance role behind this operation is a nice mitigation of this, so that people have the time to do their due diligence before that a new token is added.

Note that there is a caveat here, which is that if the initial price may have changed since the time of the proposal and its execution, which is very likely, then first buyers of the token in the omnipool could arbitrage it and effectively steal tokens until that the price has been equalized with other markets.
##### Recommendation
Use custom referendums to add new tokens or create a new pool. This way, you can define a range of the price that people would be agreeing on in order to address the caveat which is a timing issue. That way, the authority who could execute the proposal could set a price strictly defined in that range and minimize cold-liquidity losses. If you can use an oracle, it's even better.

#### Change the state of the tradability of assets, including freezing the swap, or liquidity managing operations
##### Role 
Technical
##### Risk
The "tradability" which is defined for both the Omnipool and the Stableswap has 5 different states represented as a bitmap.
This flag is used to have a more fine-grained control about what are the allowed operations on the system.
When a "state" is created, the flags are the most permissive; Everything is turned on.
There is a rather important risk here though. For instance, if the `REMOVE_LIQUIDITY` flag is turned off, people won't be able to remove their liquidity from pools, and this could create panic within liquidity providers.
If trading is still allowed, they would probably swap their tokens in order to save the most value and the price of it would drop significantly.
There is a very sensible use of this flag though in the case of the removal of tokens, which requires the asset to be completely frozen and the pool to own 100% of the liquidity of this token.
##### Recommendation
Ban the `REMOVE_LIQUIDITY` flag to guarantee that people will always be able to remove their assets from the pool.
Alternatively, make sure that a relatively low amount of liquidity is in the pool for this flag to be allowed turned off.

#### Notes
Most of the other important state changes are already sufficiently mitigated by the fact that the roles with a lot of powers are behind a governance.
The time it takes for a proposal to be in an executable state (which is currently 28 days) makes is safe enough to give time to people to move their funds if they wish to.
In general, HydraDX is very good at following a trustless protocol model.

### Diffs

For the diffs, here are the relevant commits that were used as a starting point to compare the changes added since that date.

| Pallet     | Last PR                                                  | Commit                                   |
| ---------- | -------------------------------------------------------- | ---------------------------------------- |
| Stableswap | https://github.com/galacticcouncil/HydraDX-node/pull/636 | 8cbd7676050f4b2894d7cadc490ec05cdd7bedcc |
| Oracle     | https://github.com/galacticcouncil/HydraDX-node/pull/617 | dba0f35de87554aa7cd09df62329f00fc2f438fb |
| Omnipool   | https://github.com/galacticcouncil/HydraDX-node/pull/460 | 88e58a29c1c6ef0b532085a7de85393d5e4cc0eb |

- The omnipool had two public audits. The one from Runtime Verification was last, so it supersedes the one from Blockscience.
- The circuit breaker is not in the list simply because it has, to my knowledge, not been audited before.

## Additional references

https://wiki.polkadot.network/docs/build-smart-contracts

### Time spent:
35 hours