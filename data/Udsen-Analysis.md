## Summary

The `EvolvingProteus` is a solution to passively update the liqudity concentration over time. It can be thought of as a hybrid implementation between Balancer liquidity bootstrapping pool and Uniswap v3. Here the starting liquidty concentration and ending liquidty concentration is set by the creator who sets the starting timestamp and ending timestamp as well. Within this duration the liquidity concentratino evolves.

The main smart contract which is within the scope for this audit is `EvolvingProteus.sol`.

The `EvolvingProteus.sol` is a smart contract that implements a liquidity pool with a time-dependent bonding curve. The bonding curve is used to determine the price of tokens in the pool based on the current supply. The curve evolves over time according to the parameters specified at the contract's deployment.

The contract uses the `ABDKMath64x64` library for `fixed-point arithmetic` and the `OpenZeppelin Math library` for basic mathematical operations. It also imports an interface for a liquidity pool implementation and a specified token type.

The contract includes functions for swapping tokens, depositing tokens into the pool, and withdrawing tokens from the pool. Each of these functions can be called with either a specified input amount or a specified output amount. The contract calculates the corresponding output or input amount, respectively, based on the current state of the bonding curve.

The contract also includes a library, `LibConfig`, which is used to `calculate the parameters of the bonding curve` at any given time.

The contract includes several checks to ensure that transactions do not result in an invalid state. For example, it checks that the amount of tokens being swapped, deposited, or withdrawn is not too small or too large relative to the size of the reserves in the pool. It also checks that the ratio between the two reserves does not fall below or exceed certain limits.

## Any comments for the judge to contextualize your findings

The majority of my findings have been based on how the key invariants are broken due to edge cases that `EvolvingProteus.sol` contract could face. A brief summary of each of the findings is given below:

H1 : How the `absoluteValue < FIXED_FEE * 2` key invariant is broken with in the `_applyFeeByRounding` function. This function is used to charge fees by rounding values.

H2 : How the key invariants related to reserve balances in the `_checkBalances` function can be broken due to the usage of errorneous values.

H3: `_reserveTokenSpecified` function does not check whether reserve balance invariants hold after the function exeuction is completed.

M1: The `utility` is calculated using a quadratic equation. It's solution gives two answers based on the sign of the `discriminant`. The unsafe casting of `discriminant` to `uint256` value could result in a errorneous value thus providing wrong output

M2: Condtional check which excludes the exact value `finalBalanceRatio` in the `_checkBalances` function is errorneous

M3 : Important conditinal check to ensure final LP token supply is greater than the `MIN_BALANCE` is not performed inside the `_reserveTokenSpecified`. Hence the respective invariant could be broken.

## Approach taken in evaluating the codebase

I initially started understanding the protocol by referring the documentation. I initially learnt the key workings of the `EvolvingProteus` solution. Then I understood the key applications of the `EvolvingProteus` solution such as the `Dutch auctions` and `Dynamic pools`.

Then I learnt about the key parameters of the solution related to prices, liquidity concentration and timestamps.
Then I learnt how these parameters are interconnected through the time-weighted interpolation of prices, bonding curve equation related to the liquidity concentration and quadratice equation related to the utility calculations.

Then I analyzed the key invariants of the protocol, price range limitations and types of errors anticipated in the protocol.

After gaining enough knowledge about the protocol I started auditing the `EvolvingProteus.sol` smartcontract (Inscope smartcontract). I initially focussed on the state changing functions. I analyzed whether it is possible to change the state of the contract to an invalid state via those functions.

Then I focused on key invariants of the contract. I tries to think of various methods and edge cases in breaking the given invariants. I was able to find several vulnerabilities related to breaking invariants in edge cases.

Later I checked whether all the required conditional checks and invariant checks are performed to ensure the contract states are within required boundaries after specific transactions are sucesfully executed.

By following the above mentioned approach I was able to find vulnerabilities of this project.

## Architecture recommendations

The architecture used in the `EvolvingProteus` solution is a robust one. It is heavily dependent on mathematics. It uses a novel mehtod of evolving liquidity concentration across a predetermined time duration. A bonding curve equation is used to calculate the current liquidity concentration by taking into consideration the price of tokens (in the units of  numeraire token) obtained via time-weighted interpolation of prices. The `utility` is used as the measurement of value of the liquidity pool. The `utility` is calculated based on a quadratic equation.

## Codebase quality analysis

Considering the complexity of the algorithms associated with the protocol and interlinks among different components of the procotol such as `pricing`, `liquidity concentration`, `utility`, it must be stated that the codebase quality is high and commendable. 

It is recommended to focus more on invariant checks and verify that the key invariants are held after each swap, deposit or withdraw transaction is completed. It is also recommended to implement logic in the `EvolvingProteus.sol` contract to ensrue key invariants are not broken via edge cases, becuase few vulnerabilities were found related to this during the audit.

Since there are many hard and soft invariants in this protocol it is recommended to focus more on foundry invariant tests to ensure the codebase is highly robust.

More attention can be paid on the `Natspec` comments since there were numerous typos found in them which I have included in my QA report. Since developers and auditors rely on the `Natspec` comments to get a general idea about the codebase, accurate `Natspec` commnets will increase the quality of the codebase. 

## Centralization risks

There were no centralization risks found within the `EvolvingProteus.sol` smart contract. The `external` functions of the contract were used for `swaps`, `deposits` and `withdrawals`. They were not controlled via any `access control`. And these `external` functions were `view` functions used for calculations.

Further the `internal` functions used as helper functions for above `external` functions were also `view` functions and were not controlled by any access control.

Hence there was no risk of centralization since there were `no admin controlled functions` or any role based access control within these functions of the `EvolvingProteus.sol` smart contract.

## Mechanism review

The `EvolvingProteus.sol` contract and the `EvolvingProteus` solution is heavily dependent on mathematics. There are bonding curves, time-weighted interpolation and quadratic equations used for various calculations within its scope. The majority of calculations were based on fixed-point arithmetic using the `ABDKMath64x64` library. Hence extra attention needed to be paid on the decimal precision and accuracy of resultant values of the calculations.

Since the `EvolvingProteus` solution is similar to a hybrid solution of ` liquidity bootstrapping pools` and `Uniswap V3`, the solution can be used for price discovery of newly created tokens and to execute large buy or sell order without significant price impact. The `passive liqudity concentration` management of the solution can be used to allow the `LPs` to earn more fees on thier provided liquidity since thier liquidity will be concentrated near the spot price of the asset continously.

As a result the uses cases of this protocol are on the `Dutch Auctions` and `Dynamic Pools`, which can make use of the above mentioned features of the protocol.

## Systemic risks

The systemic risks of this protocol are related to breaking of key invariants under edge cases. Possible errors related to calculations of fixed-point arithmetic and its precision and accuracy.

Missing checks to verify key invariants are holding after succesful execution of the deposit, withdraw and swap operations could lead to vulnerabilities.

This contract uses Fixed-point arithmatic. Hence the precision errors, rounding errors, underflow and overflow errors could occur. Hence need to pay extra attention on above areas when dealing with arithmatic operations in this contract

In Fixed-point arithmatic if you dedicate more bits to the fractional part for higher precision, you reduce the number of bits available for the integer part, limiting the range of whole numbers you can represent. Conversely, if you want to represent a larger range of whole numbers, you reduce the precision of the fractional part.

### Time spent:
25 hours