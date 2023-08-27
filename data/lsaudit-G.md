# [G-01] Do not calculate constants


```
146: uint256 constant INT_MAX = uint256(type(int256).max);
151: int256 constant MIN_BALANCE = 10**12;
181: uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
191: uint256 constant FIXED_FEE = 10**9;
196: int256 constant MULTIPLIER = 1e18;
201: int256 constant MAX_PRICE_RATIO = 10**4
```

# [G-02] Do not initialize variables with default values

```
211: bool constant FEE_DOWN = false;
```

# [G-03] The result of function calls should be cached rather than re-calling the function

Function `t(self)` is being called 3 times.
Firstly, in the `if`-condition (line 98), then - when condition fails: `ABDK_ONE.sub(t(self))` and `self.px_final.mul(t(self))`

```
File: EvolvingProteus.sol

097:     function p_min(Config storage self) public view returns (int128) {
098:         if (t(self) > ABDK_ONE) return self.px_final;
099:         else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
100:     }
```

The same issue occurs for `p_max`

```
File: EvolvingProteus.sol

106:     function p_max(Config storage self) public view returns (int128) {
107:         if (t(self) > ABDK_ONE) return self.py_final;
108:         else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
109:     }
```

The result of `t(self)` should be cached in a variable instead of being executed 3 times.



# [G-04] More-likely to revert operations should be executed first

```
File: EvolvingProteus.sol

250:         // price value checks
251:         if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
252:         if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();
253: 
254:         // at all times x price should be less than y price
255:         if (py_init <= px_init) revert InvalidPrice();
256:         if (py_final <= px_final) revert InvalidPrice();

```

Since `py_init`, `px_init`, `py_final`, `px_final` are `int128`, there are 2**128 values they can be assigned to.

This implies, that condition `if (py_init <= px_init)` will revert for `(2**128) * (2**128 - 1) / 2 + 2**128` cases, while
condition `if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE)`  will revert for `(2**128 - MAX_PRICE_VALUE) * 2**128 + (2**128 - MAX_PRICE_VALUE) * 2**128`

According to that, reverts for `py_init <= px_init` and `py_final <= px_final` are more likely than reverts for `py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE` and `px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE`, thus ordering of lines 251-252 and 255-256 should be changed:

```
if (py_init <= px_init) revert InvalidPrice();
if (py_final <= px_final) revert InvalidPrice();

if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE)
if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE)
```


# [G-05] OR in `if`-condition can be rewritten to two single `if` conditions

```
File: EvolvingProteus.sol
251:         if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
252:         if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();
```

Refactoring the `if`-condition in a way it won't be containing `||` operator will save more gas.


This is a simple forge test, which demonstrates gas usage for both versions:

```
     function testIfWithOR(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) public returns (uint) { 
        
        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) return 1;
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) return 1;
    }
   function testIfWithoutOr(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) public returns (uint) { 
        
        if (py_init >= MAX_PRICE_VALUE) return 1;
        if (py_final >= MAX_PRICE_VALUE) return 1;

        if (px_init <= MIN_PRICE_VALUE) return 1;
        if (px_final <= MIN_PRICE_VALUE) return 1;
    }


```

```
Gas:testIfWithOR(int128,int128,int128,int128,uint256):(uint256) (runs: 256, μ: 914, ~: 926)
Gas:testIfWithoutOr(int128,int128,int128,int128,uint256):(uint256) (runs: 256, μ: 844, ~: 855)
```

This suggests, that conditions should be rewritten to:

```
if (py_init >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
if (py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
if (px_init <= MIN_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
if (px_final <= MIN_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
```


# [G-06] Incorrect usage of `ABDKMath64x64.divu` in constructor

```
259:        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
260:        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

```

`MAX_PRICE_RATIO` is declared as `int256 constant MAX_PRICE_RATIO = 10**4;` and it is only used in the `constructor`.
Having this constant declared as `int256` and then being casted to `uint` is an unreasonable waste of gas.

Declare this constant as uint and do not cast it later:

```
201:        uint constant MAX_PRICE_RATIO = 10**4

(...)

259:        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(MAX_PRICE_RATIO, 1)) revert MaximumAllowedPriceRatioExceeded();
260:        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(MAX_PRICE_RATIO, 1)) revert MaximumAllowedPriceRatioExceeded();
```

# [G-07] Unnecessary double `if` condition in `_swap` and `_lpTokenSpecified`

```
File: EvolvingProteus.sol

529:             if (specifiedToken == SpecifiedToken.X) {
530:                 int256 fixedPoint = xi + roundedSpecifiedAmount;
531:                 (xf, yf) = _findFinalPoint(
532:                     fixedPoint,
533:                     utility,
534:                     _getPointGivenXandUtility
535:                 );
536:             } else {
537:                 int256 fixedPoint = yi + roundedSpecifiedAmount;
538:                 (xf, yf) = _findFinalPoint(
539:                     fixedPoint,
540:                     utility,
541:                     _getPointGivenYandUtility
542:                 );
543:             }
544:         }
545:         
546:         // balance checks with consideration the computed amount
547:         if (specifiedToken == SpecifiedToken.X) {
548:             computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
549:             _checkBalances(xi + specifiedAmount, yi + computedAmount);
550:         } else {
551:             computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
552:             _checkBalances(xi + computedAmount, yi + specifiedAmount);
553:         }
```

Function `_swap` checks `specifiedToken == SpecifiedToken.X` condition twice: firstly in line 529, then in line 547. There are no additional operations within those two `if` blocks, thus the result of `specifiedToken == SpecifiedToken.X` will be always the same. This implies, that the 2nd `if` condition can be removed and lines 548-549 can be included in the first `if` block:

```
            int256 utility = _getUtility(xi, yi);

            if (specifiedToken == SpecifiedToken.X) {
                int256 fixedPoint = xi + roundedSpecifiedAmount;
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenXandUtility
                );

                 computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
                _checkBalances(xi + specifiedAmount, yi + computedAmount);
            } else {
                int256 fixedPoint = yi + roundedSpecifiedAmount;
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenYandUtility
                );
                 computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
                _checkBalances(xi + computedAmount, yi + specifiedAmount);
            }

        
```

The same issue occurs for `_lpTokenSpecified`:

```
File: EvolvingProteus.sol

626:         if (specifiedToken == SpecifiedToken.X)
627:             (xf, yf) = _findFinalPoint(
628:                 yi,
629:                 uf,
630:                 _getPointGivenYandUtility
631:             );
632:         else
633:             (xf, yf) = _findFinalPoint(
634:                 xi,
635:                 uf,
636:                 _getPointGivenXandUtility
637:             );
638: 
639:         // balance checks with consideration the computed amount
640:         if (specifiedToken == SpecifiedToken.X) {
641:             computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
642:             _checkBalances(xi + computedAmount, yf);
643:         } else {
644:             computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
645:             _checkBalances(xf, yi + computedAmount);
646:         }

```

# [G-08] Unnecessary variable declaration in `_reserveTokenSpecified`

```
File: EvolvingProteus.sol

589:         uint256 result = Math.mulDiv(uint256(uf), uint256(si), uint256(ui));
590:         require(result < INT_MAX);   
591:         int256 sf = int256(result);
592: 
593:         // apply fee to the computed amount
594:         computedAmount = _applyFeeByRounding(sf - si, feeDirection);

```

Declaring `int256 sf` variable is redundant. Since it's only being used once (line 594) - `int256(result)` can be used directly: `computedAmount = _applyFeeByRounding(int256(result) - si, feeDirection);`.


# [G-09] Calculations which won't underflow can be `unchecked`

```
834:        if (absoluteValue < FIXED_FEE * 2) revert AmountError();
835:
836:        uint256 roundedAbsoluteAmount;
837:        if (feeUp) {
838:            roundedAbsoluteAmount =
839:                absoluteValue +
840:                (absoluteValue / BASE_FEE) +
841:                FIXED_FEE;
842:            require(roundedAbsoluteAmount < INT_MAX);
843:        } else
844:            roundedAbsoluteAmount =
845:                absoluteValue -
846:                (absoluteValue / BASE_FEE) -
847:                FIXED_FEE;
```

According to line 834, `absoluteValue` is at least `>= 2 * FIXED_FEE` (otherwise, function will revert with `AmountError()`).
This implies, that lines 844-847 will never underflow.
In the worst case scenario, `absoluteValue = 2 * FIXED_FEE` (because `if (absoluteValue < FIXED_FEE * 2)` - then function will revert).
Moreover, we know that `BASE_FEE = 800` (line 186).

```
roundedAbsoluteAmount = absoluteValue - (absoluteValue / BASE_FEE) - FIXED_FEE;

for BASE_FEE = 800

roundedAbsoluteAmount = absoluteValue - (absoluteValue / 800) - FIXED_FEE;

for absoluteValue = 2 * FIXED_FEE

2 * FIXED_FEE - (2 * FIXED_FEE  / 800) - FIXED_FEE = FIXED_FEE - (2 * FIXED_FEE  / 800)

FIXED_FEE - (2 * FIXED_FEE  / 800) >= 0
800 * FIXED_FEE - 2 * FIXED_FEE >= 0
798 * FIXED_FEE >= 0
```

This implies, that calculations won't underflow and can be inserted into `unchecked` block:

```
 unchecked{roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE;}
```

# [G-10] Pre-calculate equations which contain only constant values in `constructor`

```
259:        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
260:        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
```

Since `MAX_PRICE_RATIO` is a constant value, the value of `ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1))` can be calculated before compiling the contract.
Calling `ABDKMath64x64.divu` on a known, constant value is a waste of gas.

# [G-11] Pre-calculate equations which contains only constant values in `_getUtility`

```
709:        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
710:        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));
```

Since `MULTIPLIER` is a constant value, the values of `ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER))` and `ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER))` can be calculated before compiling the contract.
Calling `ABDKMath64x64.divu` on a known, constant value is a waste of gas.

# [G-12] Some equations can be inlined to reduce number of variables declarations

* `_getUtility`

```
int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));

int128 aQuad = (a.mul(b).sub(one));
int256 bQuad = (a.muli(y) + b.muli(x));
int256 cQuad = x * y;

int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```

can be changed to:

```
int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));

int128 aQuad = (a.mul(b).sub(ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER))));
int256 bQuad = (a.muli(y) + b.muli(x));

int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(x * y)*4)))));
int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```

* `_getPointGivenXandUtility`

```
int256 a_convert = a.muli(MULTIPLIER);
int256 b_convert = b.muli(MULTIPLIER);
x0 = x;

int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
int256 f_2 = (f_1 * utility) / MULTIPLIER; 
y0 = f_2;
```

can be changed to:

```
x0 = x;
y0 = (((MULTIPLIER * MULTIPLIER / ((( x0  * MULTIPLIER ) / utility) + a.muli(MULTIPLIER))) -  b.muli(MULTIPLIER)) * utility) / MULTIPLIER;
```

* `_getPointGivenYandUtility`

```
int256 a_convert = a.muli(MULTIPLIER);
int256 b_convert = b.muli(MULTIPLIER);
y0 = y;

int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
int256 f_2 = (f_1 * utility) / (MULTIPLIER); 
x0 = f_2;
```

can be changed to:

```
y0 = y;
x0 = (( ((MULTIPLIER)*(MULTIPLIER) / (( y0  * MULTIPLIER ) / utility) + b.muli(MULTIPLIER)) - a.muli(MULTIPLIER) ) * utility) / (MULTIPLIER);
```


# [G-13] `_getUtility` calculates the same value twice

```
717:        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
718:        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```

Cache results of `-bQuad*MULTIPLIER`, `disc*MULTIPLIER` and `aQuad.mul(two).muli(MULTIPLIER)` to not calculate it twice.