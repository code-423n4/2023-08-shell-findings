# gas

# summary

|      | issue  |  instance  |
|------|--------|------------|
|[G-01]| Using bools for storage incurs overhead |2|
|[G-02]| Use != 0 instead of > 0 for unsigned integer comparison|3|
|[G-03]| Use constants instead of type(uintx).max|1|
|[G-04]| Not using the named return variables when a function returns, wastes deployment gas|1|
|[G-05]| State variables should be cached in stack variables rather than re-reading them from storage|14|
|[G-06]| Use assembly for math (add, sub, mul, div)|2|

## [G-01] Using bools for storage incurs overhead
```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```    
```solidity
File: src/proteus/EvolvingProteus.sol
206   bool constant FEE_UP = true;

211   bool constant FEE_DOWN = false;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206


## [G-02]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```
```solidity
File: src/proteus/EvolvingProteus.sol
336  require(result > 0);

378  require(result > 0);

414  require(result > 0);
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336

## [G-03] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.


```solidity
File: src/proteus/EvolvingProteus.sol
146   uint256 constant INT_MAX = uint256(type(int256).max);
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L146

## [G-04]Not using the named return variables when a function returns, wastes deployment gas
When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost

```solidity
File: src/proteus/EvolvingProteus.sol
663     return uf;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L663


## [G‑05] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

Note: These are instances missed by the Bot Race.

MULTIPLIER is state varible
```solidity
File: src/proteus/EvolvingProteus.sol
709      int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
710      int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));


717      int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
718      int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709-L718


 MULTIPLIER is state varible
```solidity
File: src/proteus/EvolvingProteus.sol
746      int256 a_convert = a.muli(MULTIPLIER);
747      int256 b_convert = b.muli(MULTIPLIER);

750      int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
751      int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
752      int256 f_2 = (f_1 * utility) / MULTIPLIER; 
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L746-L752

MULTIPLIER is state varible
```solidity
File: src/proteus/EvolvingProteus.sol
777      int256 a_convert = a.muli(MULTIPLIER);
778      int256 b_convert = b.muli(MULTIPLIER);

781      int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
782      int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
783      int256 f_2 = (f_1 * utility) / (MULTIPLIER)
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L777-L783


## [G-06] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in SolidityCache multiple accesses of a mapping/array
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Subtraction
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```
Gas: 300
```
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Multiplication
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

Division
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

```solidity
File: src/proteus/EvolvingProteus.sol
530   int256 fixedPoint = xi + roundedSpecifiedAmount;

537   int256 fixedPoint = yi + roundedSpecifiedAmount;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L530