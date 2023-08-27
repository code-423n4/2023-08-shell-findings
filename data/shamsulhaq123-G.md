No | Issue |Instances|
|-|:-|:-:|:-:|
|G-01| Use calldata instead of memory for function parameters that don’t change|1
|G-02|Unnecessary look up in if condition|2
|G-03|Change public function visibility to external|8
|G-04|Using > 0 costs more gas than != 0 when used on a uint in a require() statement|3
|G-05|Do not calculate constants|4
|G-06|Use constants instead of type(uintx).max|1
|G-07|State variables can be cached instead of re-reading them from storage|2
|G-08|Use assembly to perform efficient back-to-back calls|7
|G-09|storage variable can be replaced with calldata variable|7
|G-10|Using private rather than public for constants, saves gas|1

## [G-1]  Use calldata instead of memory for function parameters that don’t change
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity

file: proteus/EvolvingProteus.sol

65 public view returns (Config memory) 
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L65

## [G-2] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: proteus/EvolvingProteus.sol

251     if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
252     if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

810    if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251-L252
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L810

## [G-3] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file: proteus/EvolvingProteus.sol

59     function newConfig(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 _duration
65    ) public view returns (Config memory)

81    function elapsed(Config storage self) public view returns (uint256) {
        return block.timestamp - self.t_init;
    }

89   function t(Config storage self) public view returns (int128) {
        return elapsed(self).divu(duration(self));
    }

97    function p_min(Config storage self) public view returns (int128)   

106   function p_max(Config storage self) public view returns (int128) 

115   function a(Config storage self) public view returns (int128) {
        return (p_max(self).inv()).sqrt();
    }

 123    function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }

132    function duration(Config storage self) public view returns (uint256) {
        return self.t_final - self.t_init;
    }
       
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L59-65
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L81
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L89
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L97
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L106 
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L115
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L121
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L132

## [G‑4] Using > 0 costs more gas than != 0 when used on a uint in a require() statement
This change saves 6 gas per instance. The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

```solidity

file: proteus/EvolvingProteus.sol

336  require(result > 0);

378     require(result > 0);

414   require(result > 0);
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L378
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L414

## [G-5] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity

file: proteus/EvolvingProteus.sol

151  int256 constant MIN_BALANCE = 10**12;

181   uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;

191   uint256 constant FIXED_FEE = 10**9;

201  int256 constant MAX_PRICE_RATIO = 10**4; 
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L181
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L191
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L201

## [G-6] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

```solidity

file: proteus/EvolvingProteus.sol

146  uint256 constant INT_MAX = uint256(type(int256).max);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L146

## [G-7] State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

```solidity

file: proteus/EvolvingProteus.sol

709  int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
710  int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709

##  [G-8] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

```solidity

file: proteus/EvolvingProteus.sol

81   function elapsed(Config storage self) public view returns (uint256) {
        return block.timestamp - self.t_init;
    }

    /**
       @notice Calculates the time as a percent of total duration
       @param self config instance
    */
    function t(Config storage self) public view returns (int128) {
        return elapsed(self).divu(duration(self));
    }

    /**
       @notice The minimum price (at the x asymptote) at the current block
       @param self config instance
    */
    function p_min(Config storage self) public view returns (int128) {
        if (t(self) > ABDK_ONE) return self.px_final;
        else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
    }

    /**
       @notice The maximum price (at the y asymptote) at the current block
       @param self config instance
    */
    function p_max(Config storage self) public view returns (int128) {
        if (t(self) > ABDK_ONE) return self.py_final;
        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
    }

    /**
       @notice Calculates the a variable in the curve eq which is basically a sq. root of the inverse of y instantaneous price
       @param self config instance
    */
    function a(Config storage self) public view returns (int128) {
        return (p_max(self).inv()).sqrt();
    }

    /**
       @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
       @param self config instance
    */
    function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }


    /**
       @notice Calculates the duration of the curve evolution
       @param self config instance
    */
    function duration(Config storage self) public view returns (uint256) {
        return self.t_final - self.t_init;
134    }


```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L81-L134

## [G-9] storage variable can be replaced with calldata variable
In the Swapper._buildPermitTransferPayload the Collateral struct is passed into the function as a storage variable named collatInfo. But it is only read from and not written to within the scope of the function implementation. Hence it is recommended to make the collatInfo a calldata variable and read from calldata instead, to save on two extra SLOAD operations for the transaction.

```solidity

file: proteus/EvolvingProteus.sol

81     function elapsed(Config storage self) public view returns (uint256) {
        return block.timestamp - self.t_init;
    }

89     function t(Config storage self) public view returns (int128) {
        return elapsed(self).divu(duration(self));
    }

97    function p_min(Config storage self) public view returns (int128) {
        if (t(self) > ABDK_ONE) return self.px_final;
        else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
    }    

106     function p_max(Config storage self) public view returns (int128) {
        if (t(self) > ABDK_ONE) return self.py_final;
        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
    }    

115     function a(Config storage self) public view returns (int128) {
        return (p_max(self).inv()).sqrt();
    }

123     function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }    

132     function duration(Config storage self) public view returns (uint256) {
           
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L81
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L89
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L97
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L106
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L115
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L123
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L132

## [G‑10] Using private rather than public for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

```solidity

file: proteus/EvolvingProteus.sol

186   uint256 public constant BASE_FEE = 800;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L186