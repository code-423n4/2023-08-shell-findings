## **Gas Report**

1. If the balance check functions **`_checkBalances`** are relatively lightweight and have similar conditions, you could potentially combine the balance checks into a single check after the calculations. This might save some gas by reducing the number of function calls.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L545C6-L553C10

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L640C6-L646C10

```solidity
File: src/proteus/EvolvingProteus.sol

547: if (specifiedToken == SpecifiedToken.X) {
548:            computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
549:            _checkBalances(xi + specifiedAmount, yi + computedAmount);
550:        } else {
551:            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
552:            _checkBalances(xi + computedAmount, yi + specifiedAmount);
553:        }
```

```diff
File: src/proteus/EvolvingProteus.sol

547: if (specifiedToken == SpecifiedToken.X) {
548:            computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
-549:           _checkBalances(xi + computedAmount, yi + specifiedAmount);
550:        } else {
551:            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
-552:            _checkBalances(xi + computedAmount, yi + specifiedAmount);
553:        }
+           require(_checkBalances(xi + computedAmount, yi + specifiedAmount),"Balance check failed"
+           );
```

2. Instead of using **`bool negative = amount < 0 ? true : false;`**, you can directly assign **`negative = amount < 0;`**. It's a more concise and readable way to assign boolean values.

instance : https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L829

3. The boundary checks involve two separate conditions with a shared variable. Consider combining them into a single condition using logical operators, which can potentially save gas.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L809C5-L814C6

```diff
File: src/proteus/EvolvingProteus.sol

809: function _checkBalances(int256 x, int256 y) private pure {
-810:        if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);
811:        int128 finalBalanceRatio = y.divi(x);
-812:        if (finalBalanceRatio < MIN_M) revert BoundaryError(x,y);
-813:        else if (MAX_M <= finalBalanceRatio) revert BoundaryError(x,y);
-    }
+            if (x < MIN_BALANCE || y < MIN_BALANCE ||
+                finalBalanceRatio < MIN_M || MAX_M <= finalBalanceRatio) {
+               revert BalanceBoundaryError(x, y);
+               }
```

4.  If the constants **`aQuad`** and **`bQuad`** don't change throughout the function, you can potentially calculate them once before entering the block of calculations. This might save some gas by avoiding redundant calculations.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L712C9-L714C1

5. Keep constant variables in the default 256 bit size [Here](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L157) we convert these constant variables to uint128 unnecessarily, we can keep them as uint256 to save some gas. uint256 is the default size so it costs extra to down-size, in this instance it does not benefit us to do so.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L157
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L163