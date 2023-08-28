# SHELL GAS OPTIMIZATIONS

## INTROUDUCTION
Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."

Emphasizing the significance of our recommendations, we strive to improve the code's efficiency while maintaining its readability. We recognize the importance of code that is easily maintainable and understandable for both developers and auditors.

## TABLE OF FINDINGS


| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Add unchecked blocks for subtractions where the operands cannot underflow | 2 |  |
| G-02| Use v4.9.0 or above OpenZeppelin contracts | 1 |  |
| G-03| Declaring Unnecessary variables | 6 | 18  |
| G-04| Structs can be modified to fit in fewer storage slots | 1 | 2000 |
| G-05 | Change public state variable visibility to private especially when its not called in other contracts | 1 |  |
| G-06| Duplicated require/if checks should be refactored to a modifier or function | 2 |  |
| G-07 | Multiplication by two should use bit shifting  | 1 |  20 |



## [G-01] Add unchecked blocks for subtractions where the operands cannot underflow
There are some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked to have some gas savings.
### 2 Instances
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L82
```solidity
file: src/proteus/EvolvingProteus.sol

81:     function elapsed(Config storage self) public view returns (uint256) {
82:         return block.timestamp - self.t_init;   @audit arithmetic can be unchecked
83:     }
```
From the `elapsed()` function above it is not possible that the `block.timestamp - self.t_init` statement would underflow as the value of `block.timestamp` would always be greater than that of `self.t_init` so we can perform the ` return block.timestamp - self.t_init` statement in an `unchecked {}` block as this would save some gas from the unnecessary internal over/underflow checks. The code could be refcatored as shown in the diff below.
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol        
index 85341bb..b1dbc2a 100644                                                         
--- a/src/proteus/EvolvingProteus.sol                                                 
+++ b/src/proteus/EvolvingProteus.sol                                                 
@@ -79,7 +79,9 @@ library LibConfig {                                                 
        @param self config instance                                                   
     */                                                                               
     function elapsed(Config storage self) public view returns (uint256) {            
-        return block.timestamp - self.t_init;                                        
+        unchecked {                                                                  
+            return block.timestamp - self.t_init;                                    
+        }                                                                            
     }                                                                                
```

- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L133
```solidity
file: src/proteus/EvolvingProteus.sol

132:    function duration(Config storage self) public view returns (uint256) {
133:        return self.t_final - self.t_init;   @audit arithmetic can be unchecked
134:    }
```
From the `duration()` function above it is not possible that the `self.t_final - self.t_init` statement would underflow as the value of `self.t_final` would always be greater than that of `self.t_init` so we can perform the ` return self.t_final - self.t_init` statement in an `unchecked {}` block as this would save some gas from the unnecessary internal over/underflow checks. The code could be recfatored as shown in the diff below.
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol
index 85341bb..69ff653 100644                                                 
--- a/src/proteus/EvolvingProteus.sol                                         
+++ b/src/proteus/EvolvingProteus.sol                                         
@@ -130,7 +130,9 @@ library LibConfig {                                       
        @param self config instance                                           
     */                                                                       
     function duration(Config storage self) public view returns (uint256) {   
-        return self.t_final - self.t_init;                                   
+        unchecked {                                                          
+            return self.t_final - self.t_init;                               
+        }                                                                    
     }                                                                        
 }                                                                            
                                                                              
```

## [G-02] Use v4.9.0 or above OpenZeppelin contracts
From your [package.json file](https://github.com/code-423n4/2023-08-shell/blob/main/package.json#L12) it shows that OpenZeppelin v4.7.3 was used I would recommend you use v4.9.0 or above as it provides many gas optimizations that would help make the protocol gas efficient. https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0



## [G-03] Declaring Unnecessary variables
some variables were defined even though they are used once. Not defining variables can reduce gas cost and contract size.

### 5 Instances
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L659
```solidity
file: src/proteus/EvolvingProteus.sol

652:    function _getUtilityFinalLp(
653:        int256 si,
654:        int256 sf,
655:        int256 xi,
656:        int256 yi
657:    ) internal view returns (int256 uf) {
658:        require(sf >= MIN_BALANCE);
659:        int256 ui = _getUtility(xi, yi);
660:        uint256 result = Math.mulDiv(uint256(ui), uint256(sf), uint256(si));
661:        require(result < INT_MAX);
662:        uf = int256(result);
663:        return uf;
664:    }
```
In the `_getUtilityFinalLp()` function above the variable `ui` was declared on [Line 659](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L659) but this variable was only used once in the function thereby making its declaration unnecessary. We could save about `3` gas units if `_getUtility(xi, yi)` is used directly in the `Math.mulDiv()` function. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol                 
index 85341bb..2870f81 100644                                                                  
--- a/src/proteus/EvolvingProteus.sol                                                          
+++ b/src/proteus/EvolvingProteus.sol                                                          
@@ -656,8 +656,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {                 
         int256 yi                                                                             
     ) internal view returns (int256 uf) {                                                     
         require(sf >= MIN_BALANCE);                                                           
-        int256 ui = _getUtility(xi, yi);                                                      
-        uint256 result = Math.mulDiv(uint256(ui), uint256(sf), uint256(si));                  
+        uint256 result = Math.mulDiv(uint256(_getUtility(xi, yi)), uint256(sf), uint256(si)); 
         require(result < INT_MAX);                                                            
         uf = int256(result);                                                                  
         return uf;                                                                            
```

- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L714
```solidity
file: src/proteus/EvolvingProteus.sol

701:    function _getUtility(
702:       int256 x,
703:       int256 y
704:    ) internal view returns (int256 utility) {
705:
706:        int128 a = config.a(); //these are abdk numbers representing the a and b values
707:        int128 b = config.b();
.
.
. 
713:        int256 bQuad = (a.muli(y) + b.muli(x));
714:        int256 cQuad = x * y;  @audit unnecessary variable declaration
715:
716:        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
717:        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
.
.
.
723:        if (utility < 0) revert CurveError(utility);
724:    }
    
```
In the `_getUtility()` function above the variable `cQuad` was declared on [Line 714](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L714) but this variable was only used once in the function thereby making its declaration unnecessary. We could save about `3` gas units if `x * y` is used directly in the `aQuad.muli()` function. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol                
index 85341bb..2dea618 100644                                                                 
--- a/src/proteus/EvolvingProteus.sol                                                         
+++ b/src/proteus/EvolvingProteus.sol                                                         
@@ -711,9 +711,8 @@ contract EvolvingProteus is ILiquidityPoolImplementation {                
                                                                                              
         int128 aQuad = (a.mul(b).sub(one));                                                  
         int256 bQuad = (a.muli(y) + b.muli(x));                                              
-        int256 cQuad = x * y;                                                                
                                                                                              
-        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));        
+        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(x * y)*4)))));        
         int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER); 
         int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```

- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L746
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L747
```solidity
file: src/proteus/EvolvingProteus.sol

739:    function _getPointGivenXandUtility(
740:        int256 x,
741:        int256 utility
742:    ) internal view returns (int256 x0, int256 y0) {
743:        int128 a = config.a();
744:        int128 b = config.b();
745:
746:        int256 a_convert = a.muli(MULTIPLIER);  @audit unnecessary variable declaration
747:        int256 b_convert = b.muli(MULTIPLIER);  @audit unnecessary variable declaration
748:        x0 = x;
749:        
750:        int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
751:        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
752:        int256 f_2 = (f_1 * utility) / MULTIPLIER; 
753:        y0 = f_2;
754:
755:        if (y0 < 0) revert CurveError(y0);
756:    }
```
In the `_getPointGivenXandUtility()` function above the variables `a_convert` and `b_convert` were declared on [Line 746](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L746) and [Line 747](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L747) respectively but these variables were only used once in the function thereby making their declarations unnecessary. We could save about `6` gas units if we refactor the code as shown in the diff below:
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol   
index 85341bb..954440e 100644                                                    
--- a/src/proteus/EvolvingProteus.sol                                            
+++ b/src/proteus/EvolvingProteus.sol                                            
@@ -743,12 +743,10 @@ contract EvolvingProteus is ILiquidityPoolImplementation { 
         int128 a = config.a();                                                  
         int128 b = config.b();                                                  
                                                                                 
-        int256 a_convert = a.muli(MULTIPLIER);                                  
-        int256 b_convert = b.muli(MULTIPLIER);                                  
         x0 = x;                                                                 
                                                                                 
-        int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);            
-        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);            
+        int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a.muli(MULTIPLIER));   
+        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b.muli(MULTIPLIER));   
         int256 f_2 = (f_1 * utility) / MULTIPLIER;                              
         y0 = f_2;                                                               
```

- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L777
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L778
```solidity
file: src/proteus/EvolvingProteus.sol

770:    function _getPointGivenYandUtility(
771:        int256 y,
772:        int256 utility
773:    ) internal view returns (int256 x0, int256 y0) {
774:        int128 a = config.a();
775:        int128 b = config.b();
776:
777:        int256 a_convert = a.muli(MULTIPLIER);
778:        int256 b_convert = b.muli(MULTIPLIER);
779:        y0 = y;
780:
.
.
.
786:        if (x0 < 0) revert CurveError(x0);
787:    }
```
In the `_getPointGivenYandUtility()` function above the variables `a_convert` and `b_convert` were declared on [Line 777](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L746) and [Line 778](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L747) respectively but these variables were only used once in the function thereby making their declarations unnecessary. We could save about `6` gas units if we refactor the code as shown in the diff below:
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol           
index 85341bb..25410f0 100644                                                            
--- a/src/proteus/EvolvingProteus.sol                                                    
+++ b/src/proteus/EvolvingProteus.sol                                                    
@@ -774,12 +774,10 @@ contract EvolvingProteus is ILiquidityPoolImplementation {         
         int128 a = config.a();                                                          
         int128 b = config.b();                                                          
                                                                                         
-        int256 a_convert = a.muli(MULTIPLIER);                                          
-        int256 b_convert = b.muli(MULTIPLIER);                                          
         y0 = y;                                                                         
                                                                                         
-        int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;                      
-        int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );                 
+        int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b.muli(MULTIPLIER);             
+        int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a.muli(MULTIPLIER) );        
         int256 f_2 = (f_1 * utility) / (MULTIPLIER);                                    
         x0 = f_2;                                                                       
```
```
Estimated Gas saved: 18
```

## [G-04] Structs can be modified to fit in fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

Some member types of the struct can be safely modified, and as result, these struct will fit in fewer storage slots. Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings. We can declare your time variables as "uint40" rather than "uint256", to save gas in structs and contract storage. 2^40 as a unix timestamp is ~35k years into the future. I'm bullish on Ethereum but I think that that should be enough for all contracts deployed today.

See references: [Link 1](https://twitter.com/PaulRBerg/status/1591832937179250693) and [Link2](https://twitter.com/RareSkills_io/status/1632033211609137156)

### 1 Instance

#### Pack `t_init` and `t_final` into one storage slot to save 1 SLOT (~2000 gas)
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L10-#L17

```solidity
file: src/proteus/EvolvingProteus.sol

10: struct Config {
11:     int128 py_init;
12:     int128 px_init;
13:     int128 py_final;
14:     int128 px_final;
15:     uint256 t_init;   @audit refactor to uint40
16:     uint256 t_final;  @audit refactor to uint40
}
```
We can successfully change the types of `t_init` and `t_final` from `uint256` to `uint40` as the max uint40(2^40) timestamp is ~35k years into the future which is a very very long time. 

```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol      
index 85341bb..5e101df 100644                                                       
--- a/src/proteus/EvolvingProteus.sol                                               
+++ b/src/proteus/EvolvingProteus.sol                                               
@@ -12,8 +12,8 @@ struct Config {                                                   
     int128 px_init;                                                                
     int128 py_final;                                                               
     int128 px_final;                                                               
-    uint256 t_init;                                                                
-    uint256 t_final;                                                               
+    uint40 t_init;                                                                 
+    uint40 t_final;                                                                
 }                                                                                  
```
```
Estimated gas saved: 2000
```

## [G-05] Change public state variable visibility to private especially when its not called in other contracts

### 1 Instance

#### State variable `config` could be made private
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L216
```solidity
file: src/proteus/EvolvingProteus.sol

216:    Config public config;
```
During compilation the compiler creates public getter functions for public state variables. The inclusion of this getter function in the generated bytecode would increase the deployment
cost for the contract. If the `config` variable is not to be called by other contract we can make the variable private or internal so as to save on deployment cost.
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol 
index 85341bb..91c1682 100644                                                  
--- a/src/proteus/EvolvingProteus.sol                                          
+++ b/src/proteus/EvolvingProteus.sol                                          
@@ -213,7 +213,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation { 
       @notice                                                                 
       pool config                                                             
     */                                                                        
-    Config public config;                                                     
+    Config private config;                                                    
```

## [G-06] Duplicated require/if checks should be refactored to a modifier or function
This saves deployment gas cost and also enhances code readability.

### 2 Instance

#### We could refactor `require(inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX)`  and `require(outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX)` into a modifier since these require checks are similar.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L319
```solidity
file: src/proteus/EvolvingProteus.sol

272:    function swapGivenInputAmount(
273:        uint256 xBalance,
274:        uint256 yBalance,
275:        uint256 inputAmount,
276:        SpecifiedToken inputToken
277:    ) external view returns (uint256 outputAmount) {
278:        // input amount validations against the current balance
279:        require(
280:            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
281:        );
282:
283:      
.
.
.
304:    }
.
.
.
312:    function swapGivenOutputAmount(
313:        uint256 xBalance,
314:        uint256 yBalance,
315:        uint256 outputAmount,
316:        SpecifiedToken outputToken
317:    ) external view returns (uint256 inputAmount) {
318:        // output amount validations against the current balance
319:        require(
320:            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
321:        );
.
.
.
344:    }
```
In the `EvolvingProteus.sol` contract the require statements `require(inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX)` and `require(outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX)` were used in `swapGivenInputAmount()` and `swapGivenOutputAmount()` functions respectively . Though syntactically the statements are different but the conditions in the require statements are very similar. We could modify the require statement and refactor it into a modifier as this would help reduce deployment cost. The diff below shows how this could be done.
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol                          
index 85341bb..3fc7f53 100644                                                                           
--- a/src/proteus/EvolvingProteus.sol                                                                   
+++ b/src/proteus/EvolvingProteus.sol
@@ -228,6 +228,12 @@ contract EvolvingProteus is ILiquidityPoolImplementation {                         
     error MaximumAllowedPriceExceeded();                                                               
     error MaximumAllowedPriceRatioExceeded();                                                          
                                                                                                        
+    modifier amountVaildations(uint256 xBalance, uint256 yBalance, uint256 amount) {                   
+        require(                                                                                       
+            amount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX                               
+        );                                                                                             
+        _;                                                                                             
+    }                                                                                                  
                                                                                                        
@@ -274,12 +280,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {                         
         uint256 yBalance,                                                                              
         uint256 inputAmount,                                                                           
         SpecifiedToken inputToken                                                                      
-    ) external view returns (uint256 outputAmount) {                                                   
-        // input amount validations against the current balance                                        
-        require(                                                                                       
-            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX                          
-        );                                                                                             
-                                                                                                       
+    ) external view amountVaildations(xBalance, yBalance, inputAmount) returns (uint256 outputAmount) {
         _checkAmountWithBalance(                                                                       
             (inputToken == SpecifiedToken.X) ? xBalance : yBalance,                                    
             inputAmount                                                                                
@@ -314,11 +315,7 @@ contract EvolvingProteus is ILiquidityPoolImplementation {                         
         uint256 yBalance,                                                                              
         uint256 outputAmount,                                                                          
         SpecifiedToken outputToken                                                                     
-    ) external view returns (uint256 inputAmount) {                                                    
-        // output amount validations against the current balance                                       
-        require(                                                                                       
-            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX                         
-        );                                                                                             
+    ) external view amountVaildations(xBalance, yBalance, outputAmount) returns (uint256 inputAmount) {
         _checkAmountWithBalance(                                                                       
             outputToken == SpecifiedToken.X ? xBalance : yBalance,                                     
:                                                                                                       
```

#### We could refactor `require(depositedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)`, `require(mintedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)`, `require(withdrawnAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)` and `require(burnedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)` into a modifier since these require checks are similar.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471
```solidity
file: src/proteus/EvolvingProteus.sol

353:    function depositGivenInputAmount(
354:        uint256 xBalance,
355:        uint256 yBalance,
356:        uint256 totalSupply,
357:        uint256 depositedAmount,
358:        SpecifiedToken depositedToken
359:    ) external view returns (uint256 mintedAmount) {
360:        // deposit amount validations against the current balance
361:        require(
362:            depositedAmount < INT_MAX &&
363:                xBalance < INT_MAX &&
364:                yBalance < INT_MAX &&
365:                totalSupply < INT_MAX
366:        );
.
.
.
380:    }
.
.
.
389:    function depositGivenOutputAmount(
390:        uint256 xBalance,
391:        uint256 yBalance,
392:        uint256 totalSupply,
393:        uint256 mintedAmount,
394:        SpecifiedToken depositedToken
395:    ) external view returns (uint256 depositedAmount) {
396:        // lp amount validations against the current balance
397:        require(
398:            mintedAmount < INT_MAX &&
399:                xBalance < INT_MAX &&
400:                yBalance < INT_MAX &&
401:                totalSupply < INT_MAX
402:        );
.
.
.
416:    }
.
.
.
426:    function withdrawGivenOutputAmount(
427:        uint256 xBalance,
428:        uint256 yBalance,
429:        uint256 totalSupply,
430:        uint256 withdrawnAmount,
431:        SpecifiedToken withdrawnToken
432:    ) external view returns (uint256 burnedAmount) {
433:        // withdraw amount validations against the current balance
434:        require(
435:            withdrawnAmount < INT_MAX &&
436:                xBalance < INT_MAX &&
437:                yBalance < INT_MAX &&
438:                totalSupply < INT_MAX
439:        );
.
.
.
453:    }
.
.
.
463:    function withdrawGivenInputAmount(
464:        uint256 xBalance,
465:        uint256 yBalance,
466:        uint256 totalSupply,
467:        uint256 burnedAmount,
468:        SpecifiedToken withdrawnToken
469:    ) external view returns (uint256 withdrawnAmount) {
470:        // lp amount validations against the current balance
471:        require(
472:            burnedAmount < INT_MAX &&
473:                xBalance < INT_MAX &&
474:                yBalance < INT_MAX &&
475:                totalSupply < INT_MAX
476:        );
.
.
.
490:    }    
```
In the `EvolvingProteus.sol` contract the require statements `require(depositedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)`, `require(mintedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)`, `require(withdrawnAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)` and `require(burnedAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX && totalSupply < INT_MAX)` were used in `depositGivenInputAmount()`,`depositGivenOutputAmount()`,`withdrawGivenOutputAmount()`, and `withdrawGivenInputAmount()` functions respectively . Though syntactically the statements are different but the conditions in the require statements are very similar. We could modify the require statement and refactor it into a modifier as this would help reduce deployment cost. The diff below shows how this could be done.

```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol                          
index 85341bb..3fc7f53 100644                                                                           
--- a/src/proteus/EvolvingProteus.sol                                                                   
+++ b/src/proteus/EvolvingProteus.sol
@@ -226,10 +226,25 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
     error InvalidPrice();
     error MinimumAllowedPriceExceeded();
     error MaximumAllowedPriceExceeded();
     error MaximumAllowedPriceRatioExceeded();
 
+    modifier amountSupplyValidations(
+        uint256 xBalance,
+        uint256 yBalance,
+        uint256 totalSupply,
+        uint256 amount
+    ) {
+        //amount validations against the current balance
+        require(
+            amount < INT_MAX &&
+            xBalance < INT_MAX &&
+            yBalance < INT_MAX &&
+            totalSupply < INT_MAX
+        );
+        _;
+    }
 

@@ -354,18 +369,16 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         uint256 xBalance,
         uint256 yBalance,
         uint256 totalSupply,
         uint256 depositedAmount,
         SpecifiedToken depositedToken
-    ) external view returns (uint256 mintedAmount) {
-        // deposit amount validations against the current balance
-        require(
-            depositedAmount < INT_MAX &&
-                xBalance < INT_MAX &&
-                yBalance < INT_MAX &&
-                totalSupply < INT_MAX
-        );
+    ) external view amountSupplyValidations(
+        xBalance,
+        yBalance,
+        totalSupply,
+        depositedAmount
+    ) returns (uint256 mintedAmount) {
 
         int256 result = _reserveTokenSpecified(
             depositedToken,
             int256(depositedAmount),
             FEE_DOWN,
@@ -390,18 +403,16 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         uint256 xBalance,
         uint256 yBalance,
         uint256 totalSupply,
         uint256 mintedAmount,
         SpecifiedToken depositedToken
-    ) external view returns (uint256 depositedAmount) {
-        // lp amount validations against the current balance
-        require(
-            mintedAmount < INT_MAX &&
-                xBalance < INT_MAX &&
-                yBalance < INT_MAX &&
-                totalSupply < INT_MAX
-        );
+    ) external view amountSupplyValidations(
+        xBalance,
+        yBalance,
+        totalSupply,
+        mintedAmount
+    ) returns (uint256 depositedAmount) {
 
         int256 result = _lpTokenSpecified(
             depositedToken,
             int256(mintedAmount),
             FEE_UP,
@@ -427,19 +438,17 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         uint256 xBalance,
         uint256 yBalance,
         uint256 totalSupply,
         uint256 withdrawnAmount,
         SpecifiedToken withdrawnToken
-    ) external view returns (uint256 burnedAmount) {
-        // withdraw amount validations against the current balance
-        require(
-            withdrawnAmount < INT_MAX &&
-                xBalance < INT_MAX &&
-                yBalance < INT_MAX &&
-                totalSupply < INT_MAX
-        );
-
+    ) external view amountSupplyValidations(
+        xBalance,
+        yBalance,
+        totalSupply,
+        withdrawnAmount
+    ) returns (uint256 burnedAmount) {
+        
         int256 result = _reserveTokenSpecified(
             withdrawnToken,
             -int256(withdrawnAmount),
             FEE_UP,
             int256(totalSupply),
@@ -464,18 +473,16 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
         uint256 xBalance,
         uint256 yBalance,
         uint256 totalSupply,
         uint256 burnedAmount,
         SpecifiedToken withdrawnToken
-    ) external view returns (uint256 withdrawnAmount) {
-        // lp amount validations against the current balance
-        require(
-            burnedAmount < INT_MAX &&
-                xBalance < INT_MAX &&
-                yBalance < INT_MAX &&
-                totalSupply < INT_MAX
-        );
+    ) external view amountSupplyValidations(
+        xBalance,
+        yBalance,
+        totalSupply,
+        burnedAmount
+    ) returns (uint256 withdrawnAmount) {
 
         int256 result = _lpTokenSpecified(
             withdrawnToken,
             -int256(burnedAmount),
             FEE_DOWN,


```

#### Please note this instances were not included in the bot report


## [G-07] Multiplication by two should use bit shifting 
x * 2 is the same as x << 1. While the compiler uses the SHL opcode to accomplish both, the version that uses multiplication incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.

### 1 Instance
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709
```solidity
709:    int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
```
The code should be refactored to:
```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol                          
index 85341bb..3fc7f53 100644                                                                           
--- a/src/proteus/EvolvingProteus.sol                                                                   
+++ b/src/proteus/EvolvingProteus.sol

-    int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
+    int128 two = ABDKMath64x64.divu(uint256(MULTIPLIER << 1), uint256(MULTIPLIER));
```
```
Estimated gas saved: 20 
```
#### Please note this instance was not included in the bot report


## CONCLUSION
As you embark on the journey of incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.