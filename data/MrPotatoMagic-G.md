# Gas Optimizations Report

| Gas Optimizations | Issues                                                                      | Instances |
|-------------------|-----------------------------------------------------------------------------|-----------|
| [G-01]            | Store common numerator and denominator expressions in variables to save gas | 1         |
| [G-02]            | Refactor if-else block to save gas                                          | 1         |
| [G-03]            | Refactor statements to a single equation to save gas                        | 2         |

**Total issues: 4 instances across 3 issues
Total gas saved: 20582 gas**

## [G-01] Store common numerator and denominator expressions in variables to save gas

**Total gas saved: 12376 gas**

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L717C1-L718C93

Before VS After
Deployment cost: 1721745 - 1710738 = 11007 gas saved
Function execution cost: 14561 - 13192 = 1369 gas saved (min.)

Both Line 717 and 718 use the below equation to find roots: 

$\frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$

We can see that in the equation, the roots are only differentiated by $\pm$. Thus, in order to save gas, we can store the $-b$, $\sqrt{b^2 - 4ac}$ and $2a$ in variables num1, num2 and denQuad respectively. num1 and num2 represent the two expressions of the numerator while denQuad represents the denominator.

Instead of this:
```solidity
File: src/proteus/EvolvingProteus.sol
717:      int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
718:      int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```
Use this:
```solidity
File: src/proteus/EvolvingProteus.sol
716:      int256 denQuad = aQuad.mul(two).muli(MULTIPLIER);
717:      int256 num1 = -bQuad*MULTIPLIER;
718:      int256 num2 = disc*MULTIPLIER;
719:      int256 r0 = (num1 + num2) / denQuad; 
720:      int256 r1 = (num1 - num2) / denQuad;
```

## [G-02] Refactor if-else block to save gas

**Total gas saved: 6230 gas saved**

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L812C4-L813C72

Before VS After
Deployment cost: 1721745 - 1715538 = 6207 gas saved
Function execution cost: 477 - 454 = 23 gas saved (min.)

The if-else block follows the pattern if(A) then do X, else if(B) then do X. If either of the conditions are met, we still execute the same statement X. Thus, it would make sense to refactor the conditions in one if block with the || operator.

Instead of this:
```solidity
File: src/proteus/EvolvingProteus.sol
812:       if (finalBalanceRatio < MIN_M) revert BoundaryError(x,y);
813:       else if (MAX_M <= finalBalanceRatio) revert BoundaryError(x,y);
```
Use this:
```solidity
File: src/proteus/EvolvingProteus.sol
812:       if (finalBalanceRatio < MIN_M || MAX_M <= finalBalanceRatio) revert BoundaryError(x,y);
```

## [G-03] Refactor statements to a single equation to save gas

**Total gas saved: 1976 gas**
There are 2 instances of this:

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L750C1-L752C52
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L781C1-L783C54

Here is the simplified equation (where ac = a_convert, bc = b_convert, u = utility, M = MULTIPLIER): https://drive.google.com/file/d/1qVgf9ycl4tnCrRR1TR19WIt47YM-xHpd/view?usp=sharing

**Note: I have purposely not further simplified the equation since a_convert and b_convert use the [muli() function](https://github.com/abdk-consulting/abdk-libraries-solidity/blob/5e1e7c11b35f8313d3f7ce11c1b86320d7c0b554/ABDKMath64x64.sol#L163) from the ABDK library and not the * operator for multiplication. Therefore, in order to not interfere with the ABDK multiplication it has been kept like this. Here is what the most simplified equation looks like (although we will not be using it)**: 

Here u = utility, and M = MULTIPLIER (a and b do not represent a_convert and b_convert),

$f2 = u*M * (u + a * y0 + a*b*u)$

The following changes below  are done for the [_getPointGivenYandUtility() function](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L770C14-L770C39). The same can be done for the [_getPointGivenXandUtility() function](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L739).

Instead of this:
```solidity
File: src/proteus/EvolvingProteus.sol
781:      int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
782:      int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
783:      int256 f_2 = (f_1 * utility) / (MULTIPLIER); 
```
Use this (check last line of simplified equation to understand this):
```solidity
File: src/proteus/EvolvingProteus.sol
781:    int256 uM = (utility*MULTIPLIER);
782:    int256 f_2 = (uM*uM - uM * a_convert * y0 + utility*utility*a_convert*b_convert)/MULTIPLIER;
```