C4 finding submitted:
     risk = G (Gas Optimization)

     # Gas Optimization

# Summary

|   no   |  Issue   |  Instance  |
|------|----------|------------|
|[G-01]|Use constants instead of type(uintx).max|1|
|[G-02]|Duplicated require()/if() checks should be refactored to a modifier or function|11|
|[G-03]|Use assembly to perform efficient back-to-back calls|2|
|[G-04]|Using bools for storage incurs overhead|5|
|[G-05]|Do not calculate constants|4|
|[G-06]|Use assembly for math (add, sub, mul, div)|10|
|[G-07]|Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead|4|


## [G-01] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.
The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

```solidity
146      uint256 constant INT_MAX = uint256(type(int256).max);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L146


## [G-02] Duplicated require()/if() checks should be refactored to a modifier or function

To optimize the gas cost of a contract and reduce redundant code, it's often recommended to refactor duplicated require() or if() checks into a modifier or function. This can help to reduce the size of the contract and make it easier to read and maintain.

```solidity
529   if (specifiedToken == SpecifiedToken.X) {
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L529

```solidity
547   if (specifiedToken == SpecifiedToken.X) {
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L547

```solidity
577  if (specifiedToken == SpecifiedToken.X) {
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L577

```solidity
626   if (specifiedToken == SpecifiedToken.X)
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L626

```solidity
640  if (specifiedToken == SpecifiedToken.X) { 
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L640

```solidity
296  require(result < 0);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L296

```solidity
451  require(result < 0);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L451

```solidity
488    require(result < 0);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L488

```solidity
336  require(result > 0);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L336

```solidity
378  require(result > 0);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L378

```solidity
414  require(result > 0);                             
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L414


## [G-03] Use assembly to perform efficient back-to-back calls

Using assembly to perform efficient back-to-back calls can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, when a contract calls another contract or making back to back function calls, it typically uses the CALL opcode. However, using CALL can be expensive in terms of gas cost, particularly if the contract frequently makes back-to-back calls to the same contract.


```solidity
563           function _reserveTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {
        int256 xf;
        int256 yf;
        int256 ui;
        int256 uf;
        {
            // calculating the final price points considering the fee
            if (specifiedToken == SpecifiedToken.X) {
                xf = xi + _applyFeeByRounding(specifiedAmount, feeDirection);
                yf = yi;
            } else {
                yf = yi + _applyFeeByRounding(specifiedAmount, feeDirection);
                xf = xi;
            }
        }

        ui = _getUtility(xi, yi);
        uf = _getUtility(xf, yf);

        uint256 result = Math.mulDiv(uint256(uf), uint256(si), uint256(ui));
        require(result < INT_MAX);   
        int256 sf = int256(result);

        // apply fee to the computed amount
        computedAmount = _applyFeeByRounding(sf - si, feeDirection);
    }
    
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L563C1-L595C6

```solidity
607       function _lpTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {
        // get final utility considering the fee
        int256 uf = _getUtilityFinalLp(
            si,
            si + _applyFeeByRounding(specifiedAmount, feeDirection),
            xi,
            yi
        );

        // get final price points
        int256 xf;
        int256 yf;
        if (specifiedToken == SpecifiedToken.X)
            (xf, yf) = _findFinalPoint(
                yi,
                uf,
                _getPointGivenYandUtility
            );
        else
            (xf, yf) = _findFinalPoint(
                xi,
                uf,
                _getPointGivenXandUtility
            );

        // balance checks with consideration the computed amount
        if (specifiedToken == SpecifiedToken.X) {
            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
            _checkBalances(xi + computedAmount, yf);
        } else {
            computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
            _checkBalances(xf, yi + computedAmount);
        }
    }
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L607C3-L647C6


## [G-04] Using bools for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from 'false' to 'true', after having been 'true' in the past.

```solidity
507        bool feeDirection,
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L507

```solidity
566        bool feeDirection,
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L566

```solidity
610        bool feeDirection,
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L610

```solidity
824        function _applyFeeByRounding(int256 amount, bool feeUp)
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L824

```solidity
829        bool negative = amount < 0 ? true : false;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L829


## [G-05] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
151        int256 constant MIN_BALANCE = 10**12;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L151

```solidity
181         uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L181

```solidity
191         uint256 constant FIXED_FEE = 10**9;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L191

```solidity
201             int256 constant MAX_PRICE_RATIO = 10**4; // to be comparable with the prices calculated through abdk math
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L201


## [G-06] Use assembly for math (add, sub, mul, div)
Optimize gas consumption in Solidity by using assembly for math operations.
Assembly provides fine-grained control and enables overflow/underflow checks to enhance contract safety while minimizing gas usage.

```solidity
717         int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L717

```solidity
718         int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L718

```solidity
750         int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L750

```solidity
751          int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L751

```solidity
752          int256 f_2 = (f_1 * utility) / MULTIPLIER; 
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L752

```solidity
781         int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L781

```solidity
782          int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L782

```solidity
783          int256 f_2 = (f_1 * utility) / (MULTIPLIER);
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L783

```solidity
838          roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +
                FIXED_FEE;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L838C1-L841C27

```solidity
844          roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L844C10-L847C27


## [G-07] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher.This is because the EVM operates on 32 bytes at a time.Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Note: These are instances missed by the Bot Race.

```solidity
11            int128 py_init;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L11

```solidity
12            int128 px_init;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L12

```solidity
13            int128 py_final;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L13

```solidity
14            int128 px_final;
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L14