# Summary

| Number | Gas Optimization Details                                                      | Context |
| :----: | :---------------------------------------------------------------------------- | :-----: |
| [G-1]  | Stack Variable used as a cheaper cache for a state variable is only used once |    1    |
| [G-2]  | Using bools for storage incurs overhead                                       |    2    |
| [G-3]  | Use assembly for math (add, sub, mul, div)                                    |    2    |
| [G-4]  | Use != 0 instead of > 0 for unsigned integer comparison                       |    3    |
| [G-5]  | Not using the named return variables when a function returns                  |    1    |
| [G-6]  | Use uint256(1)/uint256(2) instead for true and false boolean states           |    2    |

## [G-1] Stack Variable used as a cheaper cache for a state variable is only used once

If a state variable is only used, it's generally a good practice to use a stack variable instead of directly reading from storage.
This optimization can help reduce the number of SLOAD operations and save gas costs.

```solidity
File: src/proteus/EvolvingProteus.sol

    uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L181

## [G-2] Using bools for storage incurs overhead

```solidity
File: src/proteus/EvolvingProteus.sol

206   bool constant FEE_UP = true;

211   bool constant FEE_DOWN = false;
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206

## [G-3] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

```solidity
File: src/proteus/EvolvingProteus.sol

530   int256 fixedPoint = xi + roundedSpecifiedAmount;

537   int256 fixedPoint = yi + roundedSpecifiedAmount;
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L530

## [G-4] Use != 0 instead of > 0 for unsigned integer comparison

It's generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

```solidity
File: src/proteus/EvolvingProteus.sol

336  require(result > 0);

378  require(result > 0);

414  require(result > 0);
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336

## [G-5] Not using the named return variables when a function returns

```solidity
File: src/proteus/EvolvingProteus.sol

663     return uf;
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L663

## [G-6] Use uint256(1)/uint256(2) instead for true and false boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Using a bool type for storage variables can be more expensive in terms of gas costs for state changes, especially when changing from true to false. Additionally, using a uint256 to represent boolean values is more efficient in terms of gas

```solidity

206    bool constant FEE_UP = true;

211    bool constant FEE_DOWN = false;

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206C1-L212C9