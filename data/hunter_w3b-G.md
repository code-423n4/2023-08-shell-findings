# Gas Optimization

# Summary

| Number | Gas Optimization Details                                                                       | Context |
| :----: | :--------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | State variables should be `cached` in stack variables rather than re-reading them from storage |   47    |
| [G-02] | Stack Variable used as a cheaper cache for a state variable is only used once                  |    1    |
| [G-03] | Use `uint256(1)/uint256(2)` instead for true and false boolean states                          |    2    |
| [G-04] | Use `v4.9.0` OpenZeppelin contracts                                                            |    1    |
| [G-05] | Use assembly for math (add, sub, mul, div)                                                     |   12    |
| [G-06] | Use `!= 0` instead of `> 0` for unsigned integer comparison                                    |    3    |
| [G-07] | Duplicated `if()` checks should be refactored to a modifier or function                        |    4    |
| [G-08] | Don't initialize variables with default value                                                  |    2    |
| [G-09] | The result of function calls should be cached rather than re-calling the function              |    1    |
| [G-10] | Require() statements that check input arguments should be at the top of the function           |    2    |

## [G-01] State variables should be `cached` in stack variables rather than re-reading them from storage

Using stack variables to cache state variables instead of repeatedly reading them from storage is a performance optimization technique that can reduce gas costs.

### 1. `MAX_PRICE_VALUE`, `MIN_PRICE_VALUE`, `MAX_PRICE_RATIO` should be `cached` in stack variables rather than re-reading them from storage

```solidity
        // price value checks
        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

        // at all times x price should be less than y price
        if (py_init <= px_init) revert InvalidPrice();
        if (py_final <= px_final) revert InvalidPrice();

        // max. price ratio check
        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
```

```diff
diff --git a/EvolvingProteus.org.sol b/EvolvingProteus.sol
index f6ae462..1126f9e 100644
--- a/EvolvingProteus.org.sol
+++ b/EvolvingProteus.sol
@@ -1,12 +1,16 @@
     ) {
+        int256 max_price_value = MAX_PRICE_VALUE
+        int256 min_price_value = MIN_PRICE_VALUE
+        int256 max_price_ratio = MAX_PRICE_RATIO

         // price value checks
-        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
-        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

+        if (py_init >= max_price_value || py_final >= max_price_value) revert MaximumAllowedPriceExceeded();
+        if (px_init <= min_price_value || px_final <= min_price_value) revert MinimumAllowedPriceExceeded();

         // at all times x price should be less than y price
         if (py_init <= px_init) revert InvalidPrice();
         if (py_final <= px_final) revert InvalidPrice();

         // max. price ratio check
-        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
-        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
\ No newline at end of file
+        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(max_price_ratio), 1)) revert MaximumAllowedPriceRatioExceeded();
+        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(max_price_ratio), 1)) revert MaximumAllowedPriceRatioExceeded();

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251-L260

### 2. `INT_MAX` should be `cached` in stack variables rather than re-reading them from storage

```solidity
        require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );

```

```diff
diff --git a/org.sol b/not.sol
index afdd442..95df1ae 100644
--- a/org.sol
+++ b/not.sol
@@ -5,8 +5,9 @@
         SpecifiedToken inputToken
     ) external view returns (uint256 outputAmount) {
         // input amount validations against the current balance
+        uint256 int_max = INT_MAX;
         require(
-            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
+            inputAmount < int_max && xBalance < int_max && yBalance < int_max
         );

         _checkAmountWithBalance(

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L280

```solidity
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX

```

```diff
diff --git a/org.sol b/not.sol
index ed53793..0a65b89 100644
--- a/org.sol
+++ b/not.sol
@@ -1,7 +1,8 @@
     ) external view returns (uint256 inputAmount) {
         // output amount validations against the current balance
+         uint256 int_max = INT_MAX;
         require(
-            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
+            outputAmount < int_max && xBalance < int_max && yBalance < int_max
         );
         _checkAmountWithBalance(
             outputToken == SpecifiedToken.X ? xBalance : yBalance,

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L320-L321

```solidity
       require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );
```

```diff
diff --git a/org.sol b/not.sol
index 733291d..3de2991 100644
--- a/org.sol
+++ b/not.sol
@@ -1,10 +1,11 @@
     ) external view returns (uint256 mintedAmount) {
         // deposit amount validations against the current balance
+          uint256 int_max = INT_MAX;
         require(
-            depositedAmount < INT_MAX &&
-                xBalance < INT_MAX &&
-                yBalance < INT_MAX &&
-                totalSupply < INT_MAX
+            depositedAmount < int_max &&
+                xBalance < int_max &&
+                yBalance < int_max &&
+                totalSupply < int_max
         );

         int256 result = _reserveTokenSpecified(

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366

```solidity
        require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );
```

```diff
diff --git a/org.sol b/not.sol
index 7b321d0..b5bb0c4 100644
--- a/org.sol
+++ b/not.sol
@@ -1,10 +1,11 @@
     ) external view returns (uint256 burnedAmount) {
         // withdraw amount validations against the current balance
+        uint256 int_max = INT_MAX;
         require(
-            withdrawnAmount < INT_MAX &&
-                xBalance < INT_MAX &&
-                yBalance < INT_MAX &&
-                totalSupply < INT_MAX
+            withdrawnAmount < int_max &&
+                xBalance < int_max &&
+                yBalance < int_max &&
+                totalSupply < int_max
         );

         int256 result = _reserveTokenSpecified(

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439

```solidity
  require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );
```

```diff
diff --git a/org.sol b/not.sol
index 5fc0028..302f35e 100644
--- a/org.sol
+++ b/not.sol
@@ -1,10 +1,11 @@
     ) external view returns (uint256 withdrawnAmount) {
         // lp amount validations against the current balance
+        uint256 int_max = INT_MAX;
         require(
-            burnedAmount < INT_MAX &&
-                xBalance < INT_MAX &&
-                yBalance < INT_MAX &&
-                totalSupply < INT_MAX
+            burnedAmount < int_max &&
+                xBalance < int_max &&
+                yBalance < int_max &&
+                totalSupply < int_max
         );

         int256 result = _lpTokenSpecified(
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476

### 3. `MULTIPLIER` should be `cached` in stack variables rather than re-reading them from storage

```solidity
        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));

        int128 aQuad = (a.mul(b).sub(one));
        int256 bQuad = (a.muli(y) + b.muli(x));
        int256 cQuad = x * y;

        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```

```diff
diff --git a/EvolvingProteus.org.sol b/EvolvingProteus.sol
index 73d41ac..b0d1673 100644
--- a/EvolvingProteus.org.sol
+++ b/EvolvingProteus.sol
@@ -1,17 +1,15 @@
     ) internal view returns (int256 utility) {

+        int256 multiplier = MULTIPLIER;

         int128 a = config.a(); //these are abdk numbers representing the a and b values
         int128 b = config.b();

-        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
-        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));
+        int128 two = ABDKMath64x64.divu(uint256(2 * multiplier), uint256(multiplier));
+        int128 one = ABDKMath64x64.divu(uint256(multiplier), uint256(multiplier));

         int128 aQuad = (a.mul(b).sub(one));
         int256 bQuad = (a.muli(y) + b.muli(x));
         int256 cQuad = x * y;

         int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
-        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
-        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
\ No newline at end of file
+        int256 r0 = (-bQuad*multiplier + disc*multiplier) / aQuad.mul(two).muli(multiplier);
+        int256 r1 = (-bQuad*multiplier - disc*multiplier) / aQuad.mul(two).muli(multiplier);

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709

```solidity
  function _getPointGivenXandUtility(
        int256 x,
        int256 utility
    ) internal view returns (int256 x0, int256 y0) {
        int128 a = config.a();
        int128 b = config.b();

        int256 a_convert = a.muli(MULTIPLIER);
        int256 b_convert = b.muli(MULTIPLIER);
        x0 = x;

        int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
        int256 f_2 = (f_1 * utility) / MULTIPLIER;
        y0 = f_2;

        if (y0 < 0) revert CurveError(y0);
    }
```

```diff
diff --git a/EvolvingProteus.org.sol b/EvolvingProteus.sol
index 6ea9c47..cdf4b14 100644
--- a/EvolvingProteus.org.sol
+++ b/EvolvingProteus.sol
@@ -1,11 +1,13 @@
         int128 a = config.a();
         int128 b = config.b();
+        int256 multiplier = MULTIPLIER;

-        int256 a_convert = a.muli(MULTIPLIER);
-        int256 b_convert = b.muli(MULTIPLIER);
+
+        int256 a_convert = a.muli(multiplier);
+        int256 b_convert = b.muli(multiplier);
         x0 = x;

-        int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
-        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
-        int256 f_2 = (f_1 * utility) / MULTIPLIER;
+        int256 f_0 = ((( x0  * multiplier ) / utility) + a_convert);
+        int256 f_1 = ((multiplier * multiplier / f_0) -  b_convert);
+        int256 f_2 = (f_1 * utility) / multiplier;
         y0 = f_2;
\ No newline at end of file

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L739-L756

```solidity
    function _getPointGivenYandUtility(
        int256 y,
        int256 utility
    ) internal view returns (int256 x0, int256 y0) {
        int128 a = config.a();
        int128 b = config.b();

        int256 a_convert = a.muli(MULTIPLIER);
        int256 b_convert = b.muli(MULTIPLIER);
        y0 = y;

        int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
        int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
        int256 f_2 = (f_1 * utility) / (MULTIPLIER);
        x0 = f_2;
```

```diff
diff --git a/EvolvingProteus.org.sol b/EvolvingProteus.sol
index 483d804..4c0c75f 100644
--- a/EvolvingProteus.org.sol
+++ b/EvolvingProteus.sol
@@ -4,12 +4,14 @@
     ) internal view returns (int256 x0, int256 y0) {
         int128 a = config.a();
         int128 b = config.b();
+        int256 multiplier = MULTIPLIER;

-        int256 a_convert = a.muli(MULTIPLIER);
-        int256 b_convert = b.muli(MULTIPLIER);
+
+        int256 a_convert = a.muli(multiplier);
+        int256 b_convert = b.muli(multiplier);
         y0 = y;

-        int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
-        int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
-        int256 f_2 = (f_1 * utility) / (MULTIPLIER);
+        int256 f_0 = (( y0  * multiplier ) / utility) + b_convert;
+        int256 f_1 = ( ((multiplier)*(multiplier) / f_0) - a_convert );
+        int256 f_2 = (f_1 * utility) / (multiplier);
         x0 = f_2;

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L770C1-L784C18

### 4. `MIN_BALANCE` should be `cached` in stack variables rather than re-reading them from storage

```solidity
        if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);

```

```diff
diff --git a/EvolvingProteus.org.sol b/EvolvingProteus.sol
index 1b2a7d7..d9a2e1c 100644
--- a/EvolvingProteus.org.sol
+++ b/EvolvingProteus.sol
@@ -1,3 +1,4 @@
     function _checkBalances(int256 x, int256 y) private pure {
+        int256 min_balance = MIN_BALANCE;
-         if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);
+         if (x < min_balance || y < min_balance) revert BalanceError(x,y);
         int128 finalBalanceRatio = y.divi(x);

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L810-L810

### 5. `FIXED_FEE`, `BASE_FEE`, should be `cached` in stack variables rather than re-reading them from storage

```solidity
    function _applyFeeByRounding(int256 amount, bool feeUp)
        private
        pure
        returns (int256 roundedAmount)
    {
        bool negative = amount < 0 ? true : false;
        uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);
        // FIXED_FEE * 2 because we will possibly deduct the FIXED_FEE from
        // this amount, and we don't want the final amount to be less than
        // the FIXED_FEE.
        if (absoluteValue < FIXED_FEE * 2) revert AmountError();

        uint256 roundedAbsoluteAmount;
        if (feeUp) {
            roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +
                FIXED_FEE;
            require(roundedAbsoluteAmount < INT_MAX);
        } else
            roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE;

        roundedAmount = negative
            ? -int256(roundedAbsoluteAmount)
            : int256(roundedAbsoluteAmount);
    }
```

```diff
diff --git a/EvolvingProteus.org.sol b/EvolvingProteus.sol
index 4c9c103..932d3e0 100644
--- a/EvolvingProteus.org.sol
+++ b/EvolvingProteus.sol
@@ -3,25 +3,28 @@
         pure
         returns (int256 roundedAmount)
     {
+        uint256 fixed_fee = FIXED_FEE;
+        uint256 base_fee = BASE_FEE;

         bool negative = amount < 0 ? true : false;
         uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);
         // FIXED_FEE * 2 because we will possibly deduct the FIXED_FEE from
         // this amount, and we don't want the final amount to be less than
         // the FIXED_FEE.
-        if (absoluteValue < FIXED_FEE * 2) revert AmountError();
+        if (absoluteValue < fixed_fee * 2) revert AmountError();

         uint256 roundedAbsoluteAmount;
         if (feeUp) {
             roundedAbsoluteAmount =
                 absoluteValue +
-                (absoluteValue / BASE_FEE) +
-                FIXED_FEE;
+                (absoluteValue / base_fee) +
+                fixed_fee;
             require(roundedAbsoluteAmount < INT_MAX);
         } else
             roundedAbsoluteAmount =
                 absoluteValue -
-                (absoluteValue / BASE_FEE) -
-                FIXED_FEE;
+                (absoluteValue / base_fee) -
+                fixed_fee;

         roundedAmount = negative
             ? -int256(roundedAbsoluteAmount)
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L824-L852

## [G-02] Stack Variable used as a cheaper cache for a state variable is only used once

If a state variable is only used, it's generally a good practice to use a stack variable instead of directly reading from storage.
This optimization can help reduce the number of SLOAD operations and save gas costs.

```solidity
    uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
```

This state variable is used in this function if we stack variable instead of state variable it's reduce the SLOAD operations

```soldity
    function _checkAmountWithBalance(uint256 balance, uint256 amount)
        private
        pure
    {
        if (balance / amount >= MAX_BALANCE_AMOUNT_RATIO) revert AmountError();
    }
```

```diff
diff --git a/org.sol b/not.sol
index 97e64f3..61608de 100644
--- a/org.sol
+++ b/not.sol
@@ -2,5 +2,6 @@
         private
         pure
     {
+        uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
         if (balance / amount >= MAX_BALANCE_AMOUNT_RATIO) revert AmountError();
     }

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L181

## [G-03] Use `uint256(1)/uint256(2)` instead for true and false boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Using a bool type for storage variables can be more expensive in terms of gas costs for state changes, especially when changing from true to false. Additionally, using a uint256 to represent boolean values is more efficient in terms of gas

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206C1-L212C9

```solidity

206    bool constant FEE_UP = true;

211    bool constant FEE_DOWN = false;

```

## [G-04] Use `v4.9.0` OpenZeppelin contracts

The upcoming v4.9.0 version of OpenZeppelin provides many small gas optimizations.

https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0

```js
    "@openzeppelin/contracts": "^4.7.3",
```

https://github.com/code-423n4/2023-08-shell/blob/3710196ff36f90e588e05588546aaabb0e941cf4/package.json#L12-L13

## [G-05] Use assembly for math (add, sub, mul, div)

In here is a good examples how to use assembly to save some amount of Gas.
https://medium.com/@bloqarl/solidity-gas-optimization-tips-with-assembly-you-havent-heard-yet-1381c77ff078

### 1. Addition:

```
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```

Gas: 303

```soldity
// addition in assembly
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

### 2. Subtraction:

```solidity

function subTest(uint256 a, uint256 b) public pure {
uint256 c = a - b;
}
```

Gas: 300

```
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

### 3. Multiplication

```solidity
function mulTest(uint256 a, uint256 b) public pure {
uint256 c = a \* b;
}
```

Gas: 325

```solidity
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

### 4. Division

```solidity
function divTest(uint256 a, uint256 b) public pure {
uint256 c = a \* b;
}
```

Gas: 325

```solidity
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

713        int256 bQuad = (a.muli(y) + b.muli(x));

716        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));

717        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

718        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);


750        int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);

751        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);

752        int256 f_2 = (f_1 * utility) / MULTIPLIER;

781        int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;

782        int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );

783        int256 f_2 = (f_1 * utility) / (MULTIPLIER);

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L530

## [G-06] Use `!= 0` instead of `> 0` for unsigned integer comparison

The Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation.

As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```solidity
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

## [G-07] Duplicated `if()` checks should be refactored to a modifier or function

```solidity
file: /src/proteus/EvolvingProteus.sol

529        if (specifiedToken == SpecifiedToken.X) {
547        if (specifiedToken == SpecifiedToken.X) {


626        if (specifiedToken == SpecifiedToken.X)]
640        if (specifiedToken == SpecifiedToken.X)


```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol

## [G-08] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with itâ€™s default value costs unnecesary gas.

```solidity
file: /src/proteus/EvolvingProteus.sol

206    bool constant FEE_UP = true;

211    bool constant FEE_DOWN = false;

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206

## [G-9] The result of function calls should be cached rather than re-calling the function

The instances below point to the second+ call of the function within a single function

```solidity

// @audit  cache the value of _applyFeeByRounding
616        int256 uf = _getUtilityFinalLp(
617            si,
618            si + _applyFeeByRounding(specifiedAmount, feeDirection),
619            xi,
620            yi
620        );


```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L618

## [G-10] Require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas\*) in a function that may ultimately revert in the unhappy case.

```solidity

590        require(result < INT_MAX);

842        require(roundedAbsoluteAmount < INT_MAX);

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L590
