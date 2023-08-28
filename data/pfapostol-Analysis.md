# Any comments for the judge to contextualize your findings

For the 4th find in the gas report, it was quite difficult to calculate the amount of gas saved, since replacing the `library` with a full `contract` requires the developer to deploy a separate contract, which will cost more gas than deploying the `library` + `struct` and additional gas is required in order to gain access to an external contract. But since all variables are immutable, this allows you to save gas in some cases.

Even more optimization could be achieved if we included/inherited the entire `Config` contract in the `EvolvingProteus` contract, in which case the cost of deploying and accessing functions would be about the same as accessing the `library`, with a huge amount of gas would be saved thanks to immutable variables. 

I decided to abandon this option, as it requires a complete redesign of the protocol. But in the future, sponsors may consider this option.

# Approach taken in evaluating the codebase

First, I studied the contest page, analyzed the information presented there. Next, I begen to study the code base: paying attention to the fact that the most of the scope is a mathematical formulas for calculation i extracted all mathematics separately, this allowed me to understand the logic of the functions, and check their compatibility with the documentation:

$t=\dfrac{ts-ti}{tf-ti}$

$p\_min =xi(1-t) + xf*t$

$p\_max = yi(1-t) + yf*t$

$a = \sqrt{\dfrac{1}{p\_max}} = \sqrt{\dfrac{1}{yi(1-t) + yf * t}}$

$b = \sqrt{p\_min} = \sqrt{xi(1-t) + xf*t}$

$aQuad = ab-1$

$bQuad=ay + bx$

$cQuad = xy$

$Disc = \sqrt{bQuad^{2}-4*aQuad*cQuad}$

$r0=-bQuad + Disc$

$r1 = -bQuad - Disc$

$utility = r0 \text{ or } r1$

$y=(\dfrac{2}{\dfrac{x}{utility}+a}-b) * utility$

$x=(\dfrac{2}{\dfrac{y}{utility} + b} - a)*utility$

having understood all the mathematical conditions, I began to study all external functions (actually they `view`, but I mean 6 ones that supposed to be top level interfaces.) and write a description of what each function does.
 
# Architecture recommendations

It is difficult to give architectural recommendations when there is only one contract in the area, which only determines the mathematical dependence of tokens. 

But it is worth noting that some functions (like `_getUtility` and `_findFinalPoint` ( with callback `_getPointGivenXandUtility` or `_getPointGivenYandUtility`)) that are called sequentially evaluate the same parameters (`a` and `b`) it would be better to either pass these parameters as function arguments or combine functions.

# Codebase quality analysis

In general, the codebase is not badly written, the code could be improved by using more constants and good descriptions for them.

Using `ABDKMath64x64` and `openzeppelin` `Math` at the same time can confuse developers and users who will learn the contract to use.

`newConfig` builds `Config` with `t_final` and `t_init` obtained from user provided `_duration`, but then subtracts `t_init` from `t_final` to get the duration using `duration` function, which is a bit of overhead.

The `Config` structure stores 6 variables that define the entire logic of the contract. Since the initial state of the contract should not change in the future, the config should be immutable.

The `swapGivenInputAmount` and `swapGivenOutputAmount` functions do the swap calculation, for this they use the same functions with the same checks and math operations, the only thing that changes is the negative sign. Accordingly, these functions could be combined into one function by adding an additional parameter `direction`. 

```solidity
enum Direction {
    Input,
    Output
}
```

The same is true for (`depositGivenInputAmount` and `withdrawGivenOutputAmount`)  and (`depositGivenOutputAmount` and `withdrawGivenInputAmount`)

# Centralization risks

Contract is permissionless, and do not change state anyway.

# Mechanism review

The protocol uses a quadratic curve, which allows the contract to change the liquidity of the protocol over time within predetermined limits.

### Time spent:
30 hours