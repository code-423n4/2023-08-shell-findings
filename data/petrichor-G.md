# gas

# summary
|no |issue|instance|
|---|-----|--------|
[G-01]| Not using the named return variables when a function returns, wastes deployment gas|1|
|[G-02]|Use constants instead of type(uintx).max|1|
|[G-03]|Use assembly for math (add, sub, mul, div)|2|
|[G-04]|Use != 0 instead of > 0 for unsigned integer comparison|3|
|[G-05]| Using bools for storage incurs over|2|



## [G-01]Not using the named return variables when a function returns, wastes deployment gas

When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost

```solidity
File: src/proteus/EvolvingProteus.sol
663     return uf;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L663



## [G-02] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.


```solidity
File: src/proteus/EvolvingProteus.sol
146   uint256 constant INT_MAX = uint256(type(int256).max);
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L146


## [G-03] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.



```solidity
File: src/proteus/EvolvingProteus.sol
530   int256 fixedPoint = xi + roundedSpecifiedAmount;

537   int256 fixedPoint = yi + roundedSpecifiedAmount;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L530
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L537




## [G-04]Use != 0 instead of > 0 for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

```solidity
File: src/proteus/EvolvingProteus.sol
336  require(result > 0);

378  require(result > 0);

414  require(result > 0);
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L378
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L414



## [G-05] Using bools for storage incurs over

```solidity
File: src/proteus/EvolvingProteus.sol
206   bool constant FEE_UP = true;

211   bool constant FEE_DOWN = false;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L211

```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```    