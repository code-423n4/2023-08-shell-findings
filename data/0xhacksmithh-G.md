### [Gas-01] `struct` can be tightly packed to save deployment gas
```solidity
struct Config {
    int128 py_init;
    int128 px_init;
    int128 py_final;
    int128 px_final;
-   uint256 t_init;
-   uint256 t_final;

+   uint128 t_init;
+   uint128 t_final;
}
```
*Instances(01)*
```
File: EvolvingProteus.sol
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L15-L16
```

### [Gas-02] `constant` should be value not expressions
```solidity
-      int128 constant ABDK_ONE = int128(int256(1 << 64));

+      
+      int128 constant ABDK_ONE = {particular value here}  // int128(int256(1 << 64))
```
*Instances(01)*
```
File: EvolvingProteus.sol
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L49
```

### [Gas-03] Using `bools` for storage incurs overhead
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back. This is the compiler’s defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false instead

```solidity
-   bool constant FEE_UP = true;
    /** 
      @notice 
      flag to indicate decrease of the pool's perceived input or output
    */ 
-   bool constant FEE_DOWN = false;

+   uint256 constant FEE_UP = 1;
+   uint256 constant FEE_UP = 2;
```
*Instances(02)*
```
File: EvolvingProteus.sol
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206-L211
```

### [Gas-04] While validating input parameters inside constructor by some reorgaization we can save gas on failure
Before going to check `constant` state variables, we can simply check inputed values, and if they not match as per our condition we can revert save those extra reads.

```solidity
    constructor(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) { 
+       // at all times x price should be less than y price
+       if (py_init <= px_init) revert InvalidPrice(); 
+       if (py_final <= px_final) revert InvalidPrice();

        // price value checks
        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

-       // at all times x price should be less than y price
-       if (py_init <= px_init) revert InvalidPrice(); 
-       if (py_final <= px_final) revert InvalidPrice();

        // max. price ratio check
        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert 
MaximumAllowedPriceRatioExceeded();
        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

        config = LibConfig.newConfig(py_init, px_init, py_final, px_final, duration);
      }
```
*Instances(01)*
```
File: 
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L255-L256
```

### [Gas-05] Instead of creating `fixedPoint` memory variable 2 times gas can saved by creating it single time and overriding that in subsequent conditions.
```solidity
{

            int256 utility = _getUtility(xi, yi);

+           int256 fixedPoint;

            if (specifiedToken == SpecifiedToken.X) {
-               int256 fixedPoint = xi + roundedSpecifiedAmount;
+               fixedPoint = xi + roundedSpecifiedAmount;

                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenXandUtility
                );
            } else {
-               int256 fixedPoint = yi + roundedSpecifiedAmount; 
+               fixedPoint = yi + roundedSpecifiedAmount;

                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenYandUtility
                );
            }
        }
```
*Instances(2)*
```
File: EvolvingProteus.sol
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L530
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L537
```

### [Gas-06] Unnecessary `If-else` condition, when below code can merge with above code block
Instead of checking `if (specifiedToken == SpecifiedToken.X) {` 2 times and making separate code we can end lines code to upper `If-else` clause. 

```solidity
        int256 xf;
        int256 yf;
        // calculate final price points after the swap
        {

            int256 utility = _getUtility(xi, yi);

            if (specifiedToken == SpecifiedToken.X) {
                int256 fixedPoint = xi + roundedSpecifiedAmount;
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenXandUtility
                );
            } else {
                int256 fixedPoint = yi + roundedSpecifiedAmount; 
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenYandUtility
                );
            }
        }
        
        // balance checks with consideration the computed amount
        if (specifiedToken == SpecifiedToken.X) { 
            computedAmount = _applyFeeByRounding(yf - yi, feeDirection); 
            _checkBalances(xi + specifiedAmount, yi + computedAmount);
        } else {
            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
            _checkBalances(xi + computedAmount, yi + specifiedAmount); 
        }
```


```solidity
        int256 xf;
        int256 yf;
        // calculate final price points after the swap
        {

            int256 utility = _getUtility(xi, yi);

            if (specifiedToken == SpecifiedToken.X) {
                int256 fixedPoint = xi + roundedSpecifiedAmount;
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenXandUtility
                );
+           computedAmount = _applyFeeByRounding(yf - yi, feeDirection); 
+           _checkBalances(xi + specifiedAmount, yi + computedAmount);

            } else {
                int256 fixedPoint = yi + roundedSpecifiedAmount; 
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenYandUtility
                );
+           computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
+           _checkBalances(xi + computedAmount, yi + specifiedAmount);
            }
        }
        
        
```
*Instances(1)*
```
File: EvolvingProteus.sol
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L547-L553 
```

