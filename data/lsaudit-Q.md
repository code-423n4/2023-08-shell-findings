# [Q-01] Incorrect comment related to `MIN_PRICE_VALUE`

```
File: EvolvingProteus.sol
171:     /** 
172:      @notice 
173:      The minimum price value calculated with abdk library equivalent to 10^12(wei)
174:     */ 
175:     int256 constant MIN_PRICE_VALUE = 184467440737;
```

Since the minimum price value is `184467440737`, it is equivalent to 10^10, instead of 10^12
The correct comment should be: `The minimum price value calculated with abdk library equivalent to 10^10(wei)`

# [Q-02] Incorrect `InvalidPrice()` definition in protocol's description

[File: README.md](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/README.md?plain=1#L126)
```
* `InvalidPrice` - The x_init/final price cannot be less than y_init/final
```
Protocol's README.md states that: `The x_init/final price cannot be less than y_init/final`. However, according to protocol's logic, it should be: `The y_init/final price cannot be less than x_init/final`.

Recommendation: Change `The x_init/final price cannot be less than y_init/final` to `The y_init/final price cannot be less than x_init/final` in protocol's documentation



# [Q-03] Incorrect `p_min_current` and `p_max_current` description in protocol's documentation

[File: README.md](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/README.md?plain=1#L100-L101)
```
1. `int128 p_min_current` - the maximum price (at the y asymptote) at the current block
2. `int128 p_max_current` - the minimum price (at the x asymptote) at the current block
```

Actually, as name suggests, `p_min_current` is the minimum price, while `p_max_current` is the maximum price.


# [Q-04] Incorrect comment related to `function b` 

```
File: EvolvingProteus.sol
119:     /**
120:        @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
121:        @param self config instance
122:     */
123:     function b(Config storage self) public view returns (int128) {
124:         return p_min(self).sqrt();
125:     }

```

`function b()` does not calculate the inverse of `x`, and it shouldn't calculate its inverse. The current implementation of the function is correct, however, the comment describing the function is not.
The comment should state that:  `Calculates the b variable in the curve eq which is basically a sq. root of x instantaneous price`. The inverse is being calculated in `function a()`.


# [Q-05] Incorrect comment related to `withdrawGivenOutputAmount`

```
File: EvolvingProteus.sol
455:     /**
456:      * @dev Given an input amount of the LP token, we compute an amount of
457:      *  a reserve token that must be output to decrease the pool's utility in
458:      *  proportion to the pool's decrease in total supply of the LP token.
459:      * @dev We use FEE_UP because we want to increase the perceived amount of
460:      *  reserve tokens leaving the pool and to increase the observed amount of
461:      *  LP tokens being burned.
462:      */
463:     function withdrawGivenInputAmount(
```

Actually, for `withdrawGivenInputAmount` we should use `FEE_DOWN` to decrease the perceived amount of reserve tokens. The current implementation of the function is correct (it uses `FEE_DOWN`), however, the comment describing the function is not.

Line 459 should be changed to: `We use FEE_DOWN because we want to decrease the perceived amount of reserve tokens leaving the pool and to decrease the observed amount of LP tokens being burned`


# [Q-06] Rolling own `abs` function instead of using `ABDKMath64x64.abs`

```
File: EvolvingProteus.sol
829:         bool negative = amount < 0 ? true : false;
830:         uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);

```

Function `_applyFeeByRounding` implements its own calculations for absolute value, instead of using the one from `ABDKMath64x64`:

[File: abdk-libraries-solidity/ABDKMath64x64.so](https://github.com/abdk-consulting/abdk-libraries-solidity/blob/master/ABDKMath64x64.sol#L304-L309)
```
  function abs (int128 x) internal pure returns (int128) {
    unchecked {
      require (x != MIN_64x64);
      return x < 0 ? -x : x;
    }
  }
```

It's important to notice, that the current implementation misses the important check: `require (x != MIN_64x64);`.
Without this check, whenever `amount` is set to `-0x80000000000000000000000000000000` - function `_applyFeeByRounding` will revert:

```
- ( - 0x80000000000000000000000000000000) = 0x80000000000000000000000000000000

``` 
however, `0x80000000000000000000000000000000` does not fit in `int256`, thus revert will occur.


# [Q-07] Case, when `amount` is equal to 0 is omitted

```
295:        // amount cannot be less than 0
296:        require(result < 0);
297:
298:        // output amount validations against the current balance
299:        outputAmount = uint256(-result);
(...)

335:        // amount cannot be less than 0
336:        require(result > 0);
337:        inputAmount = uint256(result);
(...)

377:        // amount cannot be less than 0
378:        require(result > 0);
379:        mintedAmount = uint256(result);

(...)

413:        // amount cannot be less than 0
414:        require(result > 0);
415:        depositedAmount = uint256(result)
(...)

450:        // amount cannot be less than 0
451:        require(result < 0);
452:        burnedAmount = uint256(-result)

(...)

487:        // amount cannot be less than 0
488:        require(result < 0);
489        withdrawnAmount = uint256(-result);
```

According to the comment: `amount cannot be less than 0`. This implies, that amount has to be greater or equal to 0. The require statement does not take into a consideration a case, when `amount` will be 0.
Function will revert every time when `result = 0`, even thou - according to comment - it should not revert. When `result = 0`, then `amount = 0`. Condition that `amount` cannot be less than 0 is true for `amount = 0`.


# [N-01] Stick with one way of representing large numbers in constants

```
    int256 constant MIN_BALANCE = 10**12;
    uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
    uint256 constant FIXED_FEE = 10**9;
    int256 constant MULTIPLIER = 1e18;
    int256 constant MAX_PRICE_RATIO = 10**4; 
```

Constants are declared either as scientific notation or as power of 10. For the code clarity - stick with one way of representing numbers.

# [N-02] `FEE_DOWN`/`FEE_UP` usage may lead to confusion

`FEE_DOWN` and `FEE_UP` constants are used to decide if values passed to the function should be either true or false. This convention may, however, lead to some confusion.
It's not explicitly clear, if `FEE_DOWN = false` means, that `FEE_DOWN` is not being set (since it has `false` value). 
The cleared convention would be creating `FEE` enum, whose values `DOWN` and `UP` will be represented by `false` and `true`.


# [N-03] Grammar errors in comments

```
57:        @param _duration duration over which the curve will evolve
(...)
241:       @param duration duration for which the curve will evolve
```

`duration for which the curve will be evolving` is more grammatically  correct.


# [N-04] Do not use `&` as abbreviation for `and`

```
52:       @notice Calculates the equation parameters "a" & "b" described above & returns the config instance
```

Since `&` is being used as AND operator, using it in comment description (especially, near variables) may be confusing. 

Recommendation: Rewrite comment to: `@notice Calculates the equation parameters "a" and "b" described above and returns the config instance`