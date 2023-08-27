## Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-01](#L-01) | Dead Code in `_getUtility` function | 1 |
| [L-02](#L-02) | Confusing code comments deviates from function logic | 4 |
| [L-03](#L-03) | No check for `duration` in the constructor | 1 |

## Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Use of extra parenthesis in the code | 2 |

## [L-01] Dead Code in `_getUtility` function

Utility is the pool's internal measure of how much value it holds.
The `_getUtility` function returns the `utility` based in the input x & y parameters. 
After performing the necessary calculations, there are checks to ensure the correct value of `utility` to be returned. 
But the `if` condition will never be executed because the value of `a` & `b` will never be less than 0.
Thus everytime the `else` statement will be executed.

```
720:      if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
```

## [L-02] Confusing code comments deviates from function logic

The mentioned comments above certain checks/functions make it difficult to understand what the code actually wants to implement.
For example in most places it says that the "amount cannot be less than 0" but the `require` statement just below it states the complete opposite.

```solidity

295:          // amount cannot be less than 0
296:          require(result < 0);


450:          // amount cannot be less than 0
451:          require(result < 0);


459:          @dev We use FEE_UP because we want to increase the perceived amount of     //@note - should be FEE_DOWN
             ..........
481:         FEE_DOWN,


487:          // amount cannot be less than 0
488:          require(result < 0);
```

## [L-03] No check for `duration` in the constructor

`duration` is used to set the time for which the curve will evolve. It is used in the `Config` struct to set the `t_final` which is used in calculating various parameters.
Add a valid check to ensure the duration is not set to the default value.

```
        constructor(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) { 
        // price value checks
        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

        // at all times x price should be less than y price
        if (py_init <= px_init) revert InvalidPrice();
        if (py_final <= px_final) revert InvalidPrice();

        // max. price ratio check
        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

        config = LibConfig.newConfig(py_init, px_init, py_final, px_final, duration);
    }
```

## [NC-1] Use of extra parenthesis in the code

Remove extra parenthesis as they serve no purpose & just increase the code size.
```solidity

515:           {
516:               roundedSpecifiedAmount = _applyFeeByRounding(
517:                   specifiedAmount,
518:                   feeDirection
519:               );
520:           }


525:           {
                ....................
544:           }
```