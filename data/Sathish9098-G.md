# GAS OPTIMIZATION

##

## [G-1] Caching storage variables in memory when making multiple consecutive calls

#### Saves ``4000 GAS``

When the second call involves reading the same storage variable, it consumes more gas compared to caching it with a memory variable.

```diff
FILE: Breadcrumbs2023-08-shell/src/proteus/EvolvingProteus.sol

97: function p_min(Config storage self) public view returns (int128) {
98:       
+  Config memory _self = self; 
- 99:        if (t(self) > ABDK_ONE) return self.px_final;
+ 99:        if (t(_self ) > ABDK_ONE) return _self.px_final;
- 100:        else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
+ 100:        else return _self.px_init.mul(ABDK_ONE.sub(t(_self))).add(_self.px_final.mul(t(_self)));
101:    }

106: function p_max(Config storage self) public view returns (int128) {
+  Config memory _self = self; 
- 107:        if (t(self) > ABDK_ONE) return self.py_final;
+ 107:        if (t(_self ) > ABDK_ONE) return _self.py_final;
- 108:        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
+ 108:        else return _self.py_init.mul(ABDK_ONE.sub(t(_self))).add(_self.py_final.mul(t(_self)));
109:    }

```   
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L106-L109

## [G-2] Efficient Packing of ``Structs`` for Reduced ``Slot Consumption`` 

#### Saves ``2000 GAS``

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas)

### ``t_init,t_final`` can be used uin128 instead of uint256 : Saves ``2000 GAS , 1 SLOT``

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L10-L17

#### Explanations

The ``block.timestamp`` in Solidity represents time in seconds since the ``Unix epoch (January 1, 1970)``. If we convert the maximum value that can be stored in a ``uint128 (2^128 - 1)`` to years, we can get an estimate of the maximum representable time span.

the current estimated age of the universe is about ``13.8 billion years``. The maximum uint128 timestamp value is astronomically larger than 13.8 billion years any practical application scenario you're likely to encounter.

```diff
FILE: 2023-08-shell/src/proteus/EvolvingProteus.sol

10: struct Config {
11:    int128 py_init;
12:    int128 px_init;
13:    int128 py_final;
14:    int128 px_final;
- 15:    uint256 t_init;
- 16:    uint256 t_final;
+ 15:    uint128 t_init;
+ 16:    uint128 t_final;
17: }

```

##

## [G-3] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas. Saves ``14 Gas ``for every instances.

```diff
FILE: 2023-08-shell/src/proteus/EvolvingProteus.sol

- 146: uint256 constant INT_MAX = uint256(type(int256).max);
+ 146: uint256 constant INT_MAX = 115792089237316195423570985008687907853269984665640564039457584007913129639935;

- 151: int256 constant MIN_BALANCE = 10**12;
+ 151: int256 constant MIN_BALANCE = 1,000,000,000,000;

- 181: uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
+ 181: uint256 constant MAX_BALANCE_AMOUNT_RATIO = 100,000,000,000;

- 191: uint256 constant FIXED_FEE = 10**9;
+ 191: uint256 constant FIXED_FEE = 1,000,000,000;

- 201: int256 constant MAX_PRICE_RATIO = 10**4;
+ 201: int256 constant MAX_PRICE_RATIO = 10,000;

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L196

##

## [G-4] Function calls should be cached instead of re-calling the function

Consider caching the result instead of re-calling the function when possible. Note: this also includes casts, which cost between 42-46 gas, depending on the type.

```diff
FILE: 2023-08-shell/src/proteus/EvolvingProteus.sol
+ uint128 _tself = t(self) ;
- 98: if (t(self) > ABDK_ONE) return self.px_final;
+ 98: if (_tself > ABDK_ONE) return self.px_final;
- 99: else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
+ 99: else return self.px_init.mul(ABDK_ONE.sub(_tself)).add(self.px_final.mul(_tself));

106: function p_max(Config storage self) public view returns (int128) {
+ uint128 _tself = t(self) ;
- 107:        if (t(self) > ABDK_ONE) return self.py_final;
+ 107:        if (_tself  > ABDK_ONE) return self.py_final;
- 108:        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
+ 108:        else return self.py_init.mul(ABDK_ONE.sub(_tself)).add(self.py_final.mul(_tself));
109:    }

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L106-L108

##

## [G-5] Optimize names to save gas

public/external function names and public member variable names can be optimized to save gas. See this link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted


```solidity
FILE: Breadcrumbs2023-08-shell/src/proteus/EvolvingProteus.sol

///@audit swapGivenInputAmount(),swapGivenOutputAmount(),depositGivenInputAmount(),depositGivenOutputAmount(),withdrawGivenOutputAmount(),withdrawGivenInputAmount(),
137: contract EvolvingProteus is ILiquidityPoolImplementation {

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L272

##

## [G-6] Ternary unnecessary

z = (x == y) ? true : false => z = (x == y)

```solidity
FILE: Breadcrumbs2023-08-shell/src/proteus/EvolvingProteus.sol

- 829:  bool negative = amount < 0 ? true : false;
+ 829:   bool negative = (amount < 0) ;

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L829

##

## [G-7] Ordering struct members in decreasing size can reduce operations gas cost

Ordering the members of a struct by size can help ensure that they are aligned to word boundaries as much as possible. By placing larger members before smaller members, it can reduce the amount of unused space within each 32-byte word and increase the likelihood that each member will be aligned to a word boundary.

```diff
FILE: Breadcrumbs2023-08-shell/src/proteus/EvolvingProteus.sol

10: struct Config {
+ 15:    uint256 t_init;
11:    int128 py_init;
12:    int128 px_init;
13:    int128 py_final;
14:    int128 px_final;
- 15:    uint256 t_init;
16:    uint256 t_final;
17: }

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L10-L17

##

## [G-8] Use constants instead of type(X).max

```solidity
FILE: 2023-08-shell/src/proteus/EvolvingProteus.sol

146: uint256 constant INT_MAX = uint256(type(int256).max);

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L146










