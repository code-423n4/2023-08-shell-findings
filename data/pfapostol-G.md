##### Gas Optimizations

Gas savings are estimated using the gas report of existing `forge test --gas-report --fuzz-seed 1337` tests (the sum of all deployment costs and the sum of the costs of calling all methods ) and may vary depending on the implementation of the fix.

**NOTE:** I removed `testEdgeSwaps` and `testEdgeSwapsOverT` as they produce unstable gas output but don't contain important logic, only check edge cases (from sponsor's comment in Discord). Also `fuzz-seed` introduced to make gas data more stable.

|       | Issue | Instances | Estimated gas(deployments) | Estimated gas(avg method call) |
| :---: | :---- | :-------: | :------------------------: | :----------------------------: |
|1|Several if-else statements can be merged|2|14 014|177|
|2|Tautological statement wastes gas|1|3 800|154|
|3|Some complex calculations can be turned into constants|4|21 701|5 533|
|4|All `Config` members can be immutable.|1|Loss of some deploy gas (Medium-High amount)|Save of some user gas (Low-Medium amount)|

**Total: 8 instances over 4 issues**

---

## 1. Multiple `if-else` statements can be combined (2 instances)

There are 2 cases where two If statements follow each other with the same condition.

- src\proteus\EvolvingProteus.sol:[547-553](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L547-L553), [640-646](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L640-L646)


### Gas difference

Deployment gas cost reduced by **14 014**

Minimum functions call gas cost reduced by **120**

Average functions call gas cost reduced by **177**

Maximum functions call gas cost reduced by **246**

### Code difference

```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol
index 85341bb..b73ec35 100644
--- a/src/proteus/EvolvingProteus.sol
+++ b/src/proteus/EvolvingProteus.sol
@@ -533,6 +533,8 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
                     utility,
                     _getPointGivenXandUtility
                 );
+                computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
+                _checkBalances(xi + specifiedAmount, yi + computedAmount);
             } else {
                 int256 fixedPoint = yi + roundedSpecifiedAmount;
                 (xf, yf) = _findFinalPoint(
@@ -540,17 +542,10 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
                     utility,
                     _getPointGivenYandUtility
                 );
+                computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
+                _checkBalances(xi + computedAmount, yi + specifiedAmount);
             }
         }
-
-        // balance checks with consideration the computed amount
-        if (specifiedToken == SpecifiedToken.X) {
-            computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
-            _checkBalances(xi + specifiedAmount, yi + computedAmount);
-        } else {
-            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
-            _checkBalances(xi + computedAmount, yi + specifiedAmount);
-        }
     }

     /**
@@ -623,24 +618,20 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         // get final price points
         int256 xf;
         int256 yf;
-        if (specifiedToken == SpecifiedToken.X)
+        if (specifiedToken == SpecifiedToken.X) {
             (xf, yf) = _findFinalPoint(
                 yi,
                 uf,
                 _getPointGivenYandUtility
             );
-        else
+            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
+            _checkBalances(xi + computedAmount, yf);
+        } else {
             (xf, yf) = _findFinalPoint(
                 xi,
                 uf,
                 _getPointGivenXandUtility
             );
-
-        // balance checks with consideration the computed amount
-        if (specifiedToken == SpecifiedToken.X) {
-            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
-            _checkBalances(xi + computedAmount, yf);
-        } else {
             computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
             _checkBalances(xf, yi + computedAmount);
         }
```

---

## 2. Tautological statement wastes gas

The expression essentially takes the form: `if true then true else false`

- src\proteus\EvolvingProteus.sol:[829](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L829)

### Gas difference

Deployment gas cost reduced by **3 800**

Minimum functions call gas cost reduced by **39**

Average functions call gas cost reduced by **154**

Maximum functions call gas cost reduced by **171**

### Code difference

```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol
index 85341bb..2b910ed 100644
--- a/src/proteus/EvolvingProteus.sol
+++ b/src/proteus/EvolvingProteus.sol
@@ -826,7 +826,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         pure
         returns (int256 roundedAmount)
     {
-        bool negative = amount < 0 ? true : false;
+        bool negative = amount < 0;
         uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);
         // FIXED_FEE * 2 because we will possibly deduct the FIXED_FEE from
         // this amount, and we don't want the final amount to be less than
```

---

## 3. Some complex calculations can be turned into constants (4 instances)

There are ABDK math expressions that can actually be constants.

- src\proteus\EvolvingProteus.sol:[259-260](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L259-L260), [709-710](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L709-L710), [751](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L751), [782](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L782)

### Gas difference

Deployment gas cost reduced by **21 701**

Minimum functions call gas cost reduced by **2 831**

Average functions call gas cost reduced by **5 533**

Maximum functions call gas cost reduced by **6 123**

### Code difference

```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol
index 85341bb..f19e3a6 100644
--- a/src/proteus/EvolvingProteus.sol
+++ b/src/proteus/EvolvingProteus.sol
@@ -194,11 +194,16 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
       multiplier for math operations
     */
     int256 constant MULTIPLIER = 1e18;
+    int256 constant MULTIPLIER_POW = 1e36;
     /**
       @notice
       max price ratio
     */
     int256 constant MAX_PRICE_RATIO = 10**4; // to be comparable with the prices calculated through abdk math
+
+    int256 constant MAX_PRICE_RATIO_ABDK = 0x27100000000000000000;
+    int128 constant ABDK_TWO = 0x20000000000000000;
+
     /**
       @notice
       flag to indicate increase of the pool's perceived input or output
@@ -256,8 +261,8 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         if (py_final <= px_final) revert InvalidPrice();

         // max. price ratio check
-        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
-        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
+        if (py_init.div(py_init.sub(px_init)) > MAX_PRICE_RATIO_ABDK) revert MaximumAllowedPriceRatioExceeded();
+        if (py_final.div(py_final.sub(px_final)) > MAX_PRICE_RATIO_ABDK) revert MaximumAllowedPriceRatioExceeded();

         config = LibConfig.newConfig(py_init, px_init, py_final, px_final, duration);
       }
@@ -706,9 +711,9 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         int128 a = config.a(); //these are abdk numbers representing the a and b values
         int128 b = config.b();

-        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
-        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));
-
+        int128 two = ABDK_TWO;
+        int128 one = LibConfig.ABDK_ONE;
+
         int128 aQuad = (a.mul(b).sub(one));
         int256 bQuad = (a.muli(y) + b.muli(x));
         int256 cQuad = x * y;
@@ -748,7 +753,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         x0 = x;

         int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
-        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
+        int256 f_1 = ((MULTIPLIER_POW / f_0) -  b_convert);
         int256 f_2 = (f_1 * utility) / MULTIPLIER;
         y0 = f_2;

@@ -779,7 +784,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         y0 = y;

         int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
-        int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
+        int256 f_1 = ( (MULTIPLIER_POW / f_0) - a_convert );
         int256 f_2 = (f_1 * utility) / (MULTIPLIER);
         x0 = f_2;
```

---

## 4. All `Config` members can be immutable.

Struct members can be made immutable because they are only set once. To do this, we need to turn the "library" into a "contract" with "immutable" variables + `duration` can also be an "immutable" variable or a "pure" function.

- src\proteus\EvolvingProteus.sol:[link](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol)

### Gas difference

Estimating gas in this case is too complicated, but in general ShellProtocol will require to spend more deployment gas, but in some cases it will save gas for the user who interacts with the protocol.

### Code difference

```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol
index 85341bb..3208ca0 100644
--- a/src/proteus/EvolvingProteus.sol
+++ b/src/proteus/EvolvingProteus.sol
@@ -7,15 +7,6 @@ import "abdk-libraries-solidity/ABDKMath64x64.sol";
 import "@openzeppelin/contracts/utils/math/Math.sol";
 import {ILiquidityPoolImplementation, SpecifiedToken} from "./ILiquidityPoolImplementation.sol";

-struct Config {
-    int128 py_init;
-    int128 px_init;
-    int128 py_final;
-    int128 px_final;
-    uint256 t_init;
-    uint256 t_final;
-}
-
 /**
      * @dev The contract is called with the following parameters:
      y_init: the initial price at the y axis
@@ -41,104 +32,90 @@ struct Config {
            Max reserve ratio: the ratio between the two reserves cannot go above a certain ratio. Any transaction that results in the reserves going beyond this ratio will fall.
 */

-library LibConfig {
+contract Config {
     using ABDKMath64x64 for uint256;
     using ABDKMath64x64 for int256;
     using ABDKMath64x64 for int128;

     int128 constant ABDK_ONE = int128(int256(1 << 64));

-    /**
-       @notice Calculates the equation parameters "a" & "b" described above & returns the config instance
-       @param py_init The initial price at the y axis
-       @param px_init The initial price at the x axis
-       @param py_final The final price at the y axis
-       @param px_final The final price at the x axis
-       @param _duration duration over which the curve will evolve
-     */
-    function newConfig(
-        int128 py_init,
-        int128 px_init,
-        int128 py_final,
-        int128 px_final,
+    int128 immutable public py_init;
+    int128 immutable public px_init;
+    int128 immutable public py_final;
+    int128 immutable public px_final;
+    uint256 immutable public t_init;
+    uint256 immutable public t_final;
+    uint256 immutable public duration;
+
+    constructor(
+        int128 _py_init,
+        int128 _px_init,
+        int128 _py_final,
+        int128 _px_final,
         uint256 _duration
-    ) public view returns (Config memory) {
-
-        return Config(
-            py_init,
-            px_init,
-            py_final,
-            px_final,
-            block.timestamp,
-            block.timestamp + _duration
-        );
+    ) {
+        py_init = _py_init;
+        px_init = _px_init;
+        py_final = _py_final;
+        px_final = _px_final;
+        t_init = block.timestamp;
+        t_final = block.timestamp + _duration;
+        duration = _duration;
+    }
+
+    function data() public view returns (int128,int128,int128,int128,uint256,uint256) {
+        return (py_init, px_init, py_final, px_final, t_init, t_final);
     }

     /**
        @notice Calculates the time that has passed since deployment
-       @param self config instance
     */
-    function elapsed(Config storage self) public view returns (uint256) {
-        return block.timestamp - self.t_init;
+    function elapsed() public view returns (uint256) {
+        return block.timestamp - t_init;
     }

     /**
        @notice Calculates the time as a percent of total duration
-       @param self config instance
     */
-    function t(Config storage self) public view returns (int128) {
-        return elapsed(self).divu(duration(self));
+    function t() public view returns (int128) {
+        return elapsed().divu(duration);
     }

     /**
        @notice The minimum price (at the x asymptote) at the current block
-       @param self config instance
     */
-    function p_min(Config storage self) public view returns (int128) {
-        if (t(self) > ABDK_ONE) return self.px_final;
-        else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
+    function p_min() public view returns (int128) {
+        if (t() > ABDK_ONE) return px_final;
+        else return px_init.mul(ABDK_ONE.sub(t())).add(px_final.mul(t()));
     }

     /**
        @notice The maximum price (at the y asymptote) at the current block
-       @param self config instance
     */
-    function p_max(Config storage self) public view returns (int128) {
-        if (t(self) > ABDK_ONE) return self.py_final;
-        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
+    function p_max() public view returns (int128) {
+        if (t() > ABDK_ONE) return py_final;
+        else return py_init.mul(ABDK_ONE.sub(t())).add(py_final.mul(t()));
     }

     /**
        @notice Calculates the a variable in the curve eq which is basically a sq. root of the inverse of y instantaneous price
-       @param self config instance
     */
-    function a(Config storage self) public view returns (int128) {
-        return (p_max(self).inv()).sqrt();
+    function a() public view returns (int128) {
+        return (p_max().inv()).sqrt();
     }

     /**
        @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
-       @param self config instance
     */
-    function b(Config storage self) public view returns (int128) {
-        return p_min(self).sqrt();
-    }
-
-
-    /**
-       @notice Calculates the duration of the curve evolution
-       @param self config instance
-    */
-    function duration(Config storage self) public view returns (uint256) {
-        return self.t_final - self.t_init;
+    function b() public view returns (int128) {
+        return p_min().sqrt();
     }
 }

 contract EvolvingProteus is ILiquidityPoolImplementation {
     using ABDKMath64x64 for int128;
     using ABDKMath64x64 for int256;
-    using LibConfig for Config;
-
+
     /**
      @notice
      max threshold for amounts deposited, withdrawn & swapped
@@ -213,7 +190,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
       @notice
       pool config
     */
-    Config public config;
+    Config public immutable config;


     //*********************************************************************//
@@ -259,7 +236,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
         if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

-        config = LibConfig.newConfig(py_init, px_init, py_final, px_final, duration);
+        config = new Config(py_init, px_init, py_final, px_final, duration);
       }
```