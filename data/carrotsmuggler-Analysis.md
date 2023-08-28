# Shell Protocol Analysis

## Table of Contents

-   [Shell Protocol Analysis](#shell-protocol-analysis)
    -   [Table of Contents](#table-of-contents)
    -   [Introduction](#introduction)
    -   [Overview](#overview)
    -   [Uniswap V2 and V3](#uniswap-v2-and-v3)
    -   [Evolving Proteus](#evolving-proteus)
    -   [Liquidity addition/removal](#liquidity-additionremoval)

## Introduction

Since the actual implementation of the contracts and the actual AMM is out of scope, this analysis report will focus on the core mathematical concepts of the time based bonding curve approach implemented in the EvolvingProteus contract.

## Overview

The Evolving Proteus contract implements a concentrated AMM where the bonding curve of the AMM changes with time. The user defines an initial and final condition based on prices, and a time window. The AMM bonding curve then changes from the initial condition to the final condition over a that fixed time window. At every instant, the equation of the bonding curve is found by solving a quadratic equation, and that curve is then used to price the swap.

## Uniswap V2 and V3

Uniswap V2 is an AMM with a rectangular hyperbola bonding curve. The equation of the curve is given by `x*y=k`, and has two solutions, one in negative x,y and another in positive x,y. The relevant solutions are all in the first quadrant, with x and y being positive numbers. Below is the graph of such a function, with k=1. The curve is symmetric about the line y=x, and has asymptotes at x=0 and y=0.

![Uniswap V2](https://gist.github.com/carrotsmuggler/4b6b9aa7fc88cc7a678f547259f05ced/raw/a43fda149b880a8c6a89a6735b0a2c28c741060a/univ2.png)

Uniswap V3 is a concentrated AMM which builds on the same curve, by shifting the asymptotes. This gives rise to intercepts in the y and x axes, which serve as price "bounds", enabling the liquidity to trade actively only within these bounds. The curve is the same hyperbola shifted in x and y, given by `(x+a)(y+b)=k`. Below is the graph of such a function, with k=1, a=0.5 and b=0.5. The curve is symmetric about the line y=x, and has asymptotes at x=-a and y=-b.

![Uniswap V3](https://gist.github.com/carrotsmuggler/4b6b9aa7fc88cc7a678f547259f05ced/raw/a43fda149b880a8c6a89a6735b0a2c28c741060a/uniV3.png)

## Evolving Proteus

Evolving proteus builds on the same curve, but is time dependent. There is also an extra paramter `u` or utility, which defines the shape of the bonding curve at any given time. The equation of the curve is given by $$(\frac{x}{u}+a)(\frac{y}{u}+b)=k$$. The `u` here includes the `ki` from the docs.

For a value of u=1, the graph is identical to the uniswap v3 curve. In evolving proteus, the values of `a` and `b` change over time. These values are set by the user initially, and is linearly interpolated over a specified time window. So the exercise is to find a value of utility `u`, such that a bonding curve equation is found given the price limits `a` and `b`, and composition `x` and `y` at any given time.

From the equation, we can find the value of `u` by simplifying it to the following equation. $$u^2(ab-1)+u(ay+bx)+xy=0$$. Thus given `a,b,x,y`, this is a quadratic equation in u and has at most two solutions. Each solution of `u` defines a separate bonding curve so the choice of the two solutions must be made intelligently. Since `a*b<1` always, we can choose the larger root and make sure the roots are always positive.

We now introduce a point `(x,y)=(2,5)`. This represents some composition x,y in the first quadrant. We will try to find values of `u` which makes the bonding curve pass through this point `A`. We assume `a=0.4` and `b=2` for this example.

For `ab<1`, we get two solutions of `u`. One positive, another negative. `ab<1` implies that the product of the solutions of the quadratic has a negative sign, which fits this case. We reject the negative solution and choose the positive one, as shown in the graph below.

![ab<1 +ve u](https://gist.github.com/carrotsmuggler/4b6b9aa7fc88cc7a678f547259f05ced/raw/a43fda149b880a8c6a89a6735b0a2c28c741060a/abneg_posu.png)

## Liquidity addition/removal

Liquidity addition and removal is done in a one sided manner. For liquidity addition, we define another point B, the final state of the pool after liquidity removal. If we are removing `1` unit of X, and if A is at (2,5), then B will be at (1,5) (ignoring fees). The exercise now is to try and find a new value of `u` such that the new bonding curve passes through this point B. In the graph below, the red line is the bonding curve before addition, and the blue line is the bonding curve after addition.

![Liquidity addition](https://gist.github.com/carrotsmuggler/4b6b9aa7fc88cc7a678f547259f05ced/raw/b4fd03ac45e048c2448fda0badf340ea9609708a/liq_Add.png)

If we travel along the original curve to point C using some swaps and then try to remove the same 1 unit liquidity while the pool is at state C, we will arrive at point D, which is 1 unit to the left of C. The point D is not on the final curve. This implies that the final bonding curve after liquidity addition depends on the price in the AMM as well. Thus users can manipulate the price inside the AMM by large swaps which will influence the amount of LP tokens, or the value of utility `u` users get when exiting the pool. This can be used to cause losses to pool exiters.


### Time spent:
10 hours