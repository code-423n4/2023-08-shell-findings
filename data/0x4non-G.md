# Gas

## `_applyFeeByRounding(int256 amount, bool feeUp)` cand be optimized

### Simplify `negative` variable

Instead of;
```solidity
bool negative = amount < 0 ? true : false;
```
Use this;
```solidity
bool negative = amount < 0;
```

### Sum opcode is cheaper than multiplication opcode;
Instead of;
```solidity
FIXED_FEE * 2
```
Use this;
```solidity
FIXED_FEE + FIXED_FEE
```

### `roundedAbsoluteAmount` can be extracted to remove suplicate code:

Instead of;
```solidity
        uint256 roundedAbsoluteAmount;
        if (feeUp) {
            roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +
                FIXED_FEE;
            require(roundedAbsoluteAmount < INT_MAX);
        } else // @audit missing bracket could difficult readability
            roundedAbsoluteAmount = // @audit this could be simplified
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE;
```
Use this;
```solidity
        uint256 roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE);
        if (feeUp) {
            roundedAbsoluteAmount += FIXED_FEE;
        } else {
            roundedAbsoluteAmount -= FIXED_FEE;
        }
        require(roundedAbsoluteAmount < INT_MAX);
```

## Recommendation
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol
index 85341bb..7cbf103 100644
--- a/src/proteus/EvolvingProteus.sol
+++ b/src/proteus/EvolvingProteus.sol
@@ -826,25 +826,20 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         pure
         returns (int256 roundedAmount)
     {
-        bool negative = amount < 0 ? true : false;
+        bool negative = amount < 0;
         uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);
         // FIXED_FEE * 2 because we will possibly deduct the FIXED_FEE from
         // this amount, and we don't want the final amount to be less than
         // the FIXED_FEE.
-        if (absoluteValue < FIXED_FEE * 2) revert AmountError();
+        if (absoluteValue < FIXED_FEE + FIXED_FEE) revert AmountError();
 
-        uint256 roundedAbsoluteAmount;
+        uint256 roundedAbsoluteAmount = absoluteValue +(absoluteValue / BASE_FEE);
         if (feeUp) {
-            roundedAbsoluteAmount =
-                absoluteValue +
-                (absoluteValue / BASE_FEE) +
-                FIXED_FEE;
+            roundedAbsoluteAmount += FIXED_FEE;
             require(roundedAbsoluteAmount < INT_MAX);
-        } else
-            roundedAbsoluteAmount =
-                absoluteValue -
-                (absoluteValue / BASE_FEE) -
-                FIXED_FEE;
+        } else {
+            roundedAbsoluteAmount -= FIXED_FEE;
+        }
 
         roundedAmount = negative
             ? -int256(roundedAbsoluteAmount)
```
