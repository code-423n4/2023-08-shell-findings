# Report


## Low Issue Q&N


### [Q&N-1] when fee handling is not done properly malicious users on _swap and and _applyFeeByRounding

malicious user by sending the wrong feeUp value. a malicious user on `_swap` tries to send feeUp = true when making a deposit or withdrawal, so the user making the transaction has to pay a higher fee.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L506-L554
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L824-L852


### [Q&N-2] Loss of precision

Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

see : https://github.com/code-423n4/2023-08-shell/blob/main/bot-report.md#l-06-loss-of-precision-on-division

```solidity
File: c:\Users\user\Desktop\analyzer\4naly3er\contracts\example\EvolvingProteus.sol
830:     uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);
831:   


716:         int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
717:         int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
718:         int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
 
``````


### [Q&N-3] Multiplication on the result of a division
 
 
Dividing an integer by another integer will often result in loss of precision. When the result is multiplied by another number, the loss of precision is magnified, often to material magnitudes. (X / Z) * Y should be re-written as (X * Y) / Z


``` solidity

File: c:\Users\user\Desktop\analyzer\4naly3er\contracts\example\EvolvingProteus.sol
716:    int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));

151:    int256 constant MIN_BALANCE = 10**12;

```


### [Q&N-4] consider SpecifiedToken.X or SpecifiedToken.Y and specifiedAmount
 
whether the specifiedToken provided is one of SpecifiedToken.X or SpecifiedToken.Y. This ensures that only X or Y tokens can be used in this operation, otherwise the transaction will fail

```diff

function _lpTokenSpecified(
    SpecifiedToken specifiedToken,
    int256 specifiedAmount,
    bool feeDirection,
    int256 si,
    int256 xi,
    int256 yi
) internal view returns (int256 computedAmount) {
+    require(specifiedToken == SpecifiedToken.X || specifiedToken == SpecifiedToken.Y, "Invalid specifiedToken");
+    require(specifiedAmount >= 0, "specifiedAmount must be non-negative");

    // get final utility considering the fee
    int256 uf = _getUtilityFinalLp(
        si,
        si + _applyFeeByRounding(specifiedAmount, feeDirection),
        xi,
        yi
    );

    // get final price points
    int256 xf;
    int256 yf;
    if (specifiedToken == SpecifiedToken.X)
        (xf, yf) = _findFinalPoint(
            yi,
            uf,
            _getPointGivenYandUtility
        );
    else
        (xf, yf) = _findFinalPoint(
            xi,
            uf,
            _getPointGivenXandUtility
        );

    // balance checks with consideration the computed amount
    if (specifiedToken == SpecifiedToken.X) {
        computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
        _checkBalances(xi + computedAmount, yf);
    } else {
        computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
        _checkBalances(xf, yi + computedAmount);
    }
}

```




### [NC-1] Consider adding a block/deny-list
 
#### Impact:
Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens`

 ```solidity
 
File: example/EvolvingProteus.sol

137: contract EvolvingProteus is ILiquidityPoolImplementation {

```

### [NC-2] Consider true and false FEE UP AND FEE DOWN
 
#### Impact:
Consider true and false FEE UP AND FEE DOWN, the implementation when receiving a boolean as a parameter. when the logic is wrong or error. an error will occur if (feeUp) {

 ```solidity
 
File: c:\Users\user\Desktop\analyzer\4naly3er\contracts\example\EvolvingProteus.sol
206:     bool constant FEE_UP = true;
207:     /** 
208:       @notice 
209:       flag to indicate decrease of the pool's perceived input or output
210:     */ 

211:     bool constant FEE_DOWN = false;


837:         if (feeUp) {
838:             roundedAbsoluteAmount =
839:                 absoluteValue +
840:                 (absoluteValue / BASE_FEE) +
841:                 FIXED_FEE;
842:             require(roundedAbsoluteAmount < INT_MAX);
843:         } else
844:             roundedAbsoluteAmount =
845:                 absoluteValue -
846:                 (absoluteValue / BASE_FEE) -
847:                 FIXED_FEE;
848: 
849:         roundedAmount = negative
850:             ? -int256(roundedAbsoluteAmount)
851:             : int256(roundedAbsoluteAmount);



```


### [NC-3] Division by zero not prevented `feeDirection`


The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

```solidity
File: c:\Users\user\Desktop\analyzer\4naly3er\contracts\example\EvolvingProteus.sol
506:     function _swap(
507:         bool feeDirection,
508:         int256 specifiedAmount,
509:         int256 xi,
510:         int256 yi,
511:         SpecifiedToken specifiedToken
512:     ) internal view returns (int256 computedAmount) {
513:         int256 roundedSpecifiedAmount;
514:         // calculating the amount considering the fee
515:         {
516:             roundedSpecifiedAmount = _applyFeeByRounding(
517:                 specifiedAmount,
518:                 feeDirection
519:             );
520:         }
521: 
522:         int256 xf;
523:         int256 yf;
524:         // calculate final price points after the swap
525:         {
526: 
527:             int256 utility = _getUtility(xi, yi);
528: 
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
554:     }

```

### [NC-4] Solidity version 0.8.20 may not work on other chains due to PUSH0

```solidity
pragma solidity =0.8.10;
```

