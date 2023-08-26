## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Failed assertion inside `divuu()` function affecting the `divu()` functionality in `EvolvingProteus.sol` | 3 |
| [L&#x2011;02] | Misleading comment on function `b()` calculation | 1 |
| [L&#x2011;03] | Use of outdated version of abdk-library | 1 |
| [L&#x2011;04] | Misleading comment on `withdrawGivenInputAmount()` as it uses `FEE_DOWN` | 1 |
| [L&#x2011;05] | Misleading comment on `MIN_PRICE_VALUE ` calculation | 1 |

### [L&#x2011;01]  Failed assertion inside `divuu()` function affecting the `divu()` functionality in `EvolvingProteus.sol`
`EvolvingProteus.sol` has used `ABDKMath64x64.sol` which has a potential issue with assert condition. This assert condition is present in `divuu()` which is used in `divu()`. It is to be noted here `divu()` has been extensively used in contract at below locations,

Instance [1](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L90)
```Solidity

    function t(Config storage self) public view returns (int128) {
>>      return elapsed(self).divu(duration(self));
    }
```

Instance [2](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L259-L260)

```Solidity

    constructor(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) { 


     // some code


>>        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
>>        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

}
```

Instance [3](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L709-L710)

```Solidity

    function _getUtility(
        int256 x,
        int256 y
    ) internal view returns (int256 utility) {


     // some code

>>        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
>>        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));

     // some code

}
```

If you see inside the `divu()` in `ABDKMath64x64.sol`, `divuu()` has been used here which is a major issue 

```Solidity

  function divu (uint256 x, uint256 y) internal pure returns (int128) {
    unchecked {
      require (y != 0);
>>      uint128 result = divuu (x, y);
      require (result <= uint128 (MAX_64x64));
      return int128 (result);
    }
  }
```

The issue here with `divuu()` is with this assert condition which fails in `divuu()` function.

```Solidity

  function divuu (uint256 x, uint256 y) private pure returns (uint128) {

     // some code

>>        assert (xh == hi >> 128);

     // some code

```

This issue has been accepted by abdk libraries and seems to be fixed in latest version.
Reference link: [check here](https://github.com/abdk-consulting/abdk-libraries-solidity/commit/5e1e7c11b35f8313d3f7ce11c1b86320d7c0b554)

### Recommended Mitigation steps
abdk libraries has fixed this issue with following changes in `ABDKMath64x64.sol`

```diff

  function divuu (uint256 x, uint256 y) private pure returns (uint128) {

-        assert (xh == hi >> 128);

-        result += xl / y;
+        result += xh == hi >> 128 ? xl / y : 1;
```

Alternatively, The library can be upgraded to latest version v3.2 to fix this issue.

### [L&#x2011;02]  Misleading comment on function `b()` calculation
There is a misleading and incorrect comment on calculation of b varibale on curve equation.

```Solidity

    /**
>>       @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
       @param self config instance
    */
    function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }
```

Per contest readme, `b` variable is calculated as `b = sqrt(p_min_current)`, here it is directly taken as square root of p_min_current and does not take the inverse of p_min_current. Therefore the Natspec is incorrect.

### Recommended Mitigation Steps

```diff

    /**
-       @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
+       @notice Calculates the b variable in the curve eq which is basically a sq. root of x instantaneous price
       @param self config instance
    */
    function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }
```

### [L&#x2011;03]  Use of outdated version of abdk-library
[EvolvingProteus.sol](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L6) has used abdk-library ABDKMath64x64.sol in contract. It has used an old version which is confirmed from [package.json](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/package.json#L13) and found to be v3.0.0 which is an 2 year old version. When using external libraries in contracts, security is utmost important and use of latest version is recommended.

### Recommended Mitigation steps
Update the abdk-libraries version to v3.2

### [L&#x2011;04]  Misleading comment on `withdrawGivenInputAmount()` as it uses `FEE_DOWN`
In `EvolvingProteus.sol`, `withdrawGivenInputAmount()` has used `FEE_DOWN` in function which can be checked [here](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L481), however the Natspec incorrectly mentions the use of `FEE_UP` which can be checked [here](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L459).

### Recommended Mitigation steps

```diff

    /**
     * @dev Given an input amount of the LP token, we compute an amount of
     *  a reserve token that must be output to decrease the pool's utility in
     *  proportion to the pool's decrease in total supply of the LP token.
-    * @dev We use FEE_UP because we want to increase the perceived amount of
-    *  reserve tokens leaving the pool and to increase the observed amount of
-    *  LP tokens being burned.
+    * @dev We use FEE_DOWN because we want to decrease the perceived amount of
+    *  reserve tokens leaving the pool and to decrease the observed amount of
+    *  LP tokens being burned.
     */
    function withdrawGivenInputAmount(
```

### [L&#x2011;05]  Misleading comment on `MIN_PRICE_VALUE ` calculation
In EvolvingProteus.sol, There is misleading [comment](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L173) on `MIN_PRICE_VALUE` which should be corrected per recommendation.

### Recommended Mitigation steps

```diff

     @notice 
-    The minimum price value calculated with abdk library equivalent to 10^12(wei)
+    The minimum price value calculated with abdk library equivalent to 10^10(wei)
    */ 
    int256 constant MIN_PRICE_VALUE = 184467440737;
```