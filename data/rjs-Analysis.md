# [01] Summary

## 1.1. Introduction

Evolving Proteus is an AMM primitive for Shell Protocol's The Ocean. It complements the Static Proteus AMM primitive.

The principal innovation introduced by Evolving Proteus is a stateless AMM primitive contract that can gas-efficiently passively update liquidity concentrations over time i.e. the bonding curve which maps balances to prices can shift from the specified initial values to final values over a particular duration. This is particularly useful for implementing Dutch auctions and dynamic pools.

An [animated Desmos graph](https://www.desmos.com/calculator/anuttcbpcu) of the concept was provided by the Shell Protocol team during the contest.

## 1.2. Approach

**Day 1 (4 hours)**
- Read Shell Protocol v2 Part 1
- Read Shell Protocol v2 Part 1 Update 2
- Read Shell Protocol v2 Part 2

**Day 2 (4 hours)**
- Read Shell Protocol v1
- Read [Cowri Labs Shell Protocol v2 Security Assessment](https://github.com/trailofbits/publications/blob/master/reviews/ShellProtocolv2.pdf) by Trail of Bits

**Day 3 (4 hours)**
- Read Shell Protocol v2 Part 1 Update 3
- Read additional notes in `readme.md`
- Read pinned messages from Shell Protocol team on Discord
- Studied the Desmos graph demo and the code
- Researched bonding curve vulnerabilities
- Mapped out potential attack vectors

**Day 4 & 5 (8 hours)**
- Code review
- Attempted to fuzz `EvolvingProteus` using `echidna`, specifically focusing fuzzing values for `py_init_val`, `px_init_val`, `py_final_val`, `px_final_val` -- these were all hardcoded in the `forge` fuzz tests

## 1.3. Attack Vectors

The following attack vectors were pursued:
- Is the fundamental concept of an evolving bonding curve mathematically sound?
- Are there trivial swaps that result in a loss of user funds e.g. a swap in at init. time and a reverse swap at final time?
- Is it possible to perform a sequence of swaps that result in a loss of user funds?
- Is fee accounting performed correctly?
- Do the the fuzz tests explore a significant portion of the problem domain?

# [02] Codebase Quality Analysis

## `EvolvingProteus.sol`

**Complexity**
- This is was the only contract in scope
- It is completely stateless and math heavy
- Function complexity was kept low through extensive composition

**Documentation**
- Unfortunately, the only documentation about Evolving Proteus is in the `readme.md` and that makes the code very challenging to review
- There were numerous comment corrections posted to the Discord chat; wardens unaware of this would've been at a disadvantage
- All other Shell Protocol and Proteus documentation was out of scope code (i.e. The Ocean & Static Proteus)

**Testing**
- There are extensive unit and fuzz tests, but the initial configuration was hardcoded and this should be expanded into the fuzz testing scope

**Dependencies**
- Heavy dependency on `ABDKMath64x64`, which had no unit tests and minimal documentation
- In it's defense, `ABDKMath64x64` has been subject to extensive fuzz testing as part of the `echidna` and `medusa` community

# [03] Architecture Recommendations

## Interface
`EvolvingProteus` implements The Ocean primitive interfaces. This is a well thought out interface, but otherwise it is out of scope for further analysis.

## Implementation Verification
One downside of the stateless approach is that it is very difficult to understand the implementation as no logging is possible.

I would recommend a `forge script` to run through multiple in-scope scenarios and output curve parameters and prices.  This data can be fed into a graphing script so that the algorithm's performance in various scenarios can be visually analyzed and documented.

## Gas Optimization
It was very difficult to do meaningful gas optimization tests due to the heavy reliance on fuzz testing. This is an interesting outcome, because the main selling point of The Ocean's primitives is gas savings.

# [04] Risks

## Centralization Risks
As this is a stateless contract, there are no centralization risks.
## Systemic Risks

**Gas Prices**
Gas prices are a significant component of an AMM. Evolving Proteus targets Arbitrum, which is a reasonable choice.

**Ecosystem Reliance**
As an AMM primitive in the Shell Protocol's Ocean, there may be vulnerabilities in Ocean which impact Evolving Proteus.

### Time spent:
24 hours