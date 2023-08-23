
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Multiplication and Division by 2 Should use in Bit Shifting | [Multiplication and Division by 2 Should use in Bit Shifting](#multiplication-and-division-by-2-should-use-in-bit-shifting) |2 | 40 |
|[G-2] Using bools for storage incurs overhead | [Using bools for storage incurs overhead](#using-bools-for-storage-incurs-overhead) | 2 | 34200 |
|[G-3] Greater or Equal Comparison Costs Less Gas than Greater Comparison | [Greater or Equal Comparison Costs Less Gas than Greater Comparison](#greater-or-equal-comparison-costs-less-gas-than-greater-comparison) | 1 | 3 |
|[G-4] Use Small Integer For Efficiency | [Use Small Integer For Efficiency](#use-small-integer-for-efficiency) | 3 | 18 |
|[G-5] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 1 | - |
|[G-6] Initialize Variables With Default Value | [Initialize Variables With Default Value](#initialize-variables-with-default-value) | 1 | 6 |

Total: 6 issues

#

## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 709          int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER))
```
Unnecessary cast: `uint256(2 * MULTIPLIER)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709)


</details>

# 


## Multiplication and Division by 2 Should use in Bit Shifting
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 40

### note 
reported only on 2 instances that was missed in the wining bot.

### Description
The expressions 'x * 2' and 'x / 2' can be optimized for gas efficiency by utilizing bitwise operations. In Solidity, you can achieve the same results by using bitwise left shift (x << 1) for multiplication and bitwise right shift (x >> 1) for division.

Using bitwise shift operations (SHL and SHR) instead of multiplication (MUL) and division (DIV) opcodes can lead to significant gas savings. The MUL and DIV opcodes cost 5 gas, while the SHL and SHR opcodes incur a lower cost of only 3 gas.

By leveraging these more efficient bitwise operations, you can reduce the gas consumption of your smart contracts and enhance their overall performance.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 709          int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER))
```
 instead `2` use bit shifting `1` 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 716          int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))))
```
 instead `4` use bit shifting `2` 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716)
 


</details>

# 



## Using bools for storage incurs overhead
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 34200

### Description
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 206          bool constant FEE_UP = true
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 211          bool constant FEE_DOWN = false
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L211](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L211)


</details>

# 


## Greater or Equal Comparison Costs Less Gas than Greater Comparison
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 3

### Description
In Solidity, the >= operator costs less gas than the > operator. This is because the Solidity compiler uses the LT opcode for >= comparisons, which requires less gas than the combination of GT and ISZERO opcodes used for > comparisons. Therefore, replacing > with >= when possible can reduce the gas cost of your contract.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 829          bool negative = amount < 0 ? true : false
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L829](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L829)


</details>

# 


## Short-circuit rules can be used to optimize some gas usage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 21000

### Description
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a `stateVariable` (a variable stored in contract storage) and a `localVariable` (a variable in memory). 

If you have a condition like `stateVariable > 0 && localVariable > 0`, if `localVariable > 0` is false, the Solidity runtime will still execute `stateVariable > 0`, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to `localVariable > 0 && stateVariable > 0`, the `stateVariable > 0` check won't happen if `localVariable > 0` is false, saving you the SLOAD gas cost.

Similarly, for the `||` operator, if you have a condition like `stateVariable > 0 || localVariable > 0`, and `stateVariable > 0` is true, the Solidity runtime will still execute `localVariable > 0`. But if you reorder the condition to `localVariable > 0 || stateVariable > 0`, and `localVariable > 0` is true, the `stateVariable > 0` check won't happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of `&&` and `||`.

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 279          require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        )
```


```
 // @audit: Switch yBalance < INT_MAX && inputAmount < INT_MAX && xBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279-L281](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279-L281)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 319          require(
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        )
```


```
 // @audit: Switch yBalance < INT_MAX && outputAmount < INT_MAX && xBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L319-L321](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L319-L321)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 361          require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch yBalance < INT_MAX && depositedAmount < INT_MAX && xBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 361          require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch totalSupply < INT_MAX && depositedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 397          require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch yBalance < INT_MAX && mintedAmount < INT_MAX && xBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 397          require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch totalSupply < INT_MAX && mintedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 434          require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch yBalance < INT_MAX && withdrawnAmount < INT_MAX && xBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 434          require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch totalSupply < INT_MAX && withdrawnAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 471          require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch yBalance < INT_MAX && burnedAmount < INT_MAX && xBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 471          require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```


```
 // @audit: Switch totalSupply < INT_MAX && burnedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX 
```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476)


</details>

# 


## Use Small Integer For Efficiency
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 18

### Description
This detector flags contracts that inefficiently use `uint` or `int` of sizes smaller than 32 bytes. The EVM operates on 32 bytes at a time, thus using elements smaller than this may cause your contract's gas usage to be higher. Refer to the Solidity documentation for more details: https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 49          int128 constant ABDK_ONE = int128(int256(1 << 64))
```
 `ABDK_ONE` use 256 bites instead of 128

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L49](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L49)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 157          int128 constant MAX_M = 0x5f5e1000000000000000000
```
 `MAX_M` use 256 bites instead of 128

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L157](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L157)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 163          int128 constant MIN_M = 0x00000000000002af31dc461
```
 `MIN_M` use 256 bites instead of 128

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L163](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L163)


</details>

# 

