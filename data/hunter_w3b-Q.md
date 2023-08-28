# Low

# Summary

| Number | Issue                                                                              | Context |
| :----: | :--------------------------------------------------------------------------------- | :-----: |
| [L-01] | Negative Calculation Result Stored in uint256 Integer Leading to Incorrect Outcome |    1    |
| [L-02] | Incorrect Utility Calculation in `_getUtility` function                            |    1    |
| [L-03] | Potential Underflow Issue in `_getUtility` Function                                |    1    |

## [L-01] Negative Calculation Result Stored in uint256 Integer Leading to Incorrect Outcome

The result of calculation in else-statement in the `_applyFeeByRounding` function store in the `roundedAbsoluteAmount` variable,however the calculation results came negative but the `roundedAbsoluteAmount` variable is uint256 so this will return wrong data.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L844-L847

```solidity
     roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE;
```

## [L-02] Incorrect Utility Calculation in `_getUtility` function

The issue arises from how the quadratic formula results are computed and assigned to r0 and r1. The formula should use the correct signs for the calculations, considering the sign of the coefficient a. However, in the current code, the signs of both a and b are checked together, which results in incorrect utility values when a and b have opposite signs. The calculated utility might be incorrect in cases where a and b have opposite signs, leading to unexpected behavior in the contract functions that rely on utility calculations.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L701-L724

```solidity

        if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;

```

## [L-03] Potential Underflow Issue in `_getUtility` Function

In the `_getUtility` function of the EvolvingProteus.sol file, there is a potential underflow issue that needs to be addressed. The underflow could occur due to subtraction when the multiplication of certain values becomes too large, causing the subtrahend to exceed the minuend. This situation needs careful handling to avoid unexpected behavior.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L701-L723

```soldiity
 function _getUtility(
        int256 x,
        int256 y
    ) internal view returns (int256 utility) {

        int128 a = config.a(); //these are abdk numbers representing the a and b values
        int128 b = config.b();

        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));

        int128 aQuad = (a.mul(b).sub(one));
        int256 bQuad = (a.muli(y) + b.muli(x));
        int256 cQuad = x * y;

        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

        if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
        else utility = (r0 > r1) ? r0 : r1;

        if (utility < 0) revert CurveError(utility);
    }
```
