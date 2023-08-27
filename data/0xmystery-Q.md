## Unreachable if block in function `_getUtility`
In function `_getUtility`: 

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720-L721

```solidity
        if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
        else utility = (r0 > r1) ? r0 : r1;
```
`a` and `b` are the output of a square root function below. Hence they are always greater than or equal to zero, making the if block unreachable at all times.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L111-L125

```solidity
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
```
## Use `uint256` instead of `uint`
In favor of explicitness, consider replacing the following instances of `uint` with `uint256`:

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L259-L260

```solidity
        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
```
## Typo errors
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L736
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L768

```solidity
     @ audit is was
     *  without caring about which particular function is was called.
```
## Skip `r1` if `disc` is zero
When the discriminant is zero, there can only be 1 unique solution. Here's a suggested code refactoring:

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716-L718

```diff
        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
-        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
+        if (disc != 0) int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```

 