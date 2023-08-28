# Gas Optimization

# Summary

| Number | Gas                                                                                              | Context |
| :----: | :----------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Require() or revert() statements that check input arguments should be at the top of the function |    2    |
| [G-02] | The result of function calls should be cached rather than re-calling the function                |    2    |
| [G-03] | Don't initialize variables with default value                                                    |    2    |
| [G-04] | Use != 0 instead of > 0 for unsigned integer comparison                                          |    3    |
| [G-05] | Use constants instead of type(uintx).max                                                         |    1    |
| [G-06] | State variables should be cached in stack variables rather than re-reading them from storage     |    6    |
| [G-07] | Duplicated require()/if() checks should be refactored to a modifier or function                  |    2    |

## [G-01] Require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas\*) in a function that may ultimately revert in the unhappy case.

```solidity
file: /src/proteus/EvolvingProteus.sol

590        require(result < INT_MAX);

842        require(roundedAbsoluteAmount < INT_MAX);

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L590

## [G-02] The result of function calls should be cached rather than re-calling the function

The cases below indicate additional calls to the function within a single function.

```solidity
file: /src/proteus/EvolvingProteus.sol

681    function _findFinalPoint(

652       function _getUtilityFinalLp(

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L681

## [G-03] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```solidity
file: /src/proteus/EvolvingProteus.sol

206    bool constant FEE_UP = true;

211    bool constant FEE_DOWN = false;

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206

## [G-04] Use != 0 instead of > 0 for unsigned integer comparison

It's generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, Swhile the > 0 comparison requires an additional subtraction operation.

```solidity
File: src/proteus/EvolvingProteus.sol

336  require(result > 0);

378  require(result > 0);

414  require(result > 0);
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336

## [G-05] Use constants instead of type(uintx).max

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

```solidity
File: src/proteus/EvolvingProteus.sol

146   uint256 constant INT_MAX = uint256(type(int256).max);
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L146

## [G-06] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function.Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read.

```solidity
file: /src/proteus/EvolvingProteus.sol

834       if (absoluteValue < FIXED_FEE * 2) revert AmountError();

836        uint256 roundedAbsoluteAmount;

837        if (feeUp) {
            roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +
                FIXED_FEE;
            require(roundedAbsoluteAmount < INT_MAX);
        } else
            roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
847                FIXED_FEE;

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L834-L847

## [G-07] Duplicated require()/if() checks should be refactored to a modifier or function

```solidity
file: /src/proteus/EvolvingProteus.sol

529        if (specifiedToken == SpecifiedToken.X) {
547        if (specifiedToken == SpecifiedToken.X) {

626        if (specifiedToken == SpecifiedToken.X)]
640        if (specifiedToken == SpecifiedToken.X)

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L529