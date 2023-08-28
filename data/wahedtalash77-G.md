|     |     |     |
| --- | --- | --- |
| ID | Title | Instances |
| \[G‑01\] | Using `bool`s for storage incurs overhead | 2   |
| \[G‑02\] | instead of using double `if` it's efficient to use `if` and `else` | 4   |
| \[G-03\] | For `uint` use `!= 0` instead of `> 0` | 9   |
| G-04\] | \[Use Shift Right/Left instead of Division/Multiplication | 2   |
| \[G-05\] | Optimize names to save gas |     |
| \[G-06\] | Booleans use `8 bits` while you only need `1 bit` |     |
| \[G‑07\] | choose the appropriate version of the Solidity form the list |     |
| \[G‑08\] | Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement |     |

## \[G‑01\] Using `bool`s for storage incurs overhead

```
// Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**<ins>100 gas</ins>**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past.

```Solidity
bool constant FEE_UP = true;
    /** 
      @notice 
      flag to indicate decrease of the pool's perceived input or output
    */ 
    bool constant FEE_DOWN = false;
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206C1-L211C36

`bool negative = amount < 0 ? true : false;`

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L829

## \[G‑02\] instead of using double `if` it's efficient to use `if` and `else`

it's efficient because there is no need for chinking the condition again

```Solidity
if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

        // at all times x price should be less than y price
        if (py_init <= px_init) revert InvalidPrice();
        if (py_final <= px_final) revert InvalidPrice();
```

## https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251C8-L256C57

## \[G-03\] For `uint` use `!= 0` instead of `> 0`

`require(result < 0);`

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L296

`require(result > 0);`

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L336

`require(result > 0);`

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L378

`require(result > 0);`

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L414

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L451

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L488

`if (x0 < 0) revert CurveError(x0);`

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L786

` if (utility < 0) revert CurveError(utility);`

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L723

## \[G-04\] Use Shift Right/Left instead of Division/Multiplication

A division/multiplication by any number x being a power of 2 can be calculated by shifting to the right/left. While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas.

Furthermore, Solidity’s division operation also includes a division-by-0 prevention which is bypassed using shifting.

`int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));`

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709

`if (absoluteValue < FIXED_FEE * 2) revert AmountError();`

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L834

## \[G-05\] Optimize names to save gas

> Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

> See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

## \[G-06\] Booleans use `8 bits` while you only need `1 bit`

Under the hood of solidity, Booleans (`bool)` are `uint8` which means they use `8 bits` of storage. A Boolean can only have two values: True or False. This means that you can store a Boolean in only a single bit. You can pack 256 Booleans in a single word. The easiest way is to take a `uint256` variable and use all `256 bits` of it to represent individual Booleans

https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6

## \[G-07\] choose the appropriate version of the Solidity form the list

instead of using `pragma solidity =0.8.10`

https://github.com/ethereum/solidity/blob/develop/Changelog.md