### Low Vulnerabilties

## [L-01] : Unsafe typecasting of `int256` to `uint256` can result in unintended underflow result if int256 hold -ve number. 

In Solidity, int256 is a signed integer that can represent both positive and negative values. When you cast a negative int256 to a uint256, sign extension occurs. This means that the most significant bit (MSB) of the int256 (which indicates the sign) is treated as just another bit in the uint256. 

The range of int256 is from -2^255 to 2^255 - 1, while the range of uint256 is from 0 to 2^256 - 1. Casting a large negative int256 to uint256 can result in a value that's numerically far larger than what the original int256 value represented.

Since Solidity doesn't provide runtime checks for overflow or underflow during typecasting, the conversion may result in data corruption or invalid values, leading to contract vulnerabilities or incorrect results.

### In below code parameter `si` is not checked if it is negative or not before typecasting it to `uint256` at line 589.

```solidity
File : src/proteus/EvolvingProteus.sol

       function _reserveTokenSpecified(
           SpecifiedToken specifiedToken,
           int256 specifiedAmount,
           bool feeDirection,
           int256 si,
           int256 xi,
           int256 yi
       ) internal view returns (int256 computedAmount) {
     
        ...

589:       uint256 result = Math.mulDiv(uint256(uf), uint256(si), uint256(ui));
           require(result < INT_MAX);   
           int256 sf = int256(result);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L589

Recommendation : Check for `si` is non negative Or  if it is negative and still want to typecast to mak it positive, first convert it to +ve then typecast.

```diff
+           require(si >= 0);
589:        uint256 result = Math.mulDiv(uint256(uf), uint256(si), uint256(ui));
            require(result < INT_MAX); 
```