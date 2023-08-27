### [N-01] Missing require Statement in ``_applyFeeByRounding`` Function

To prevent unintended outcomes, such as values falling below acceptable limits after fee adjustments.

### Recommended Mitigation Steps

``` require(roundedAbsoluteAmount >= FIXED_FEE * 2);```

### [N-02] Inconsistent use of ``Safe Math``  libraries instead of  unchecked operations
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol/#L714-L719
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol/#L749-L754
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol/#L781-L783


### [N-03] Missing bracket in ``MULTIPLIER``

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol/#L751-L52

 adding brackets like in function ``_getPointGivenYandUtility``

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol/#L782-L83

### [N-04] 
The following comment should have  ``X `` instead of ``Y``
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol/#L764-l765
It should be Calculates the b variable in the curve eq which is basically a sq. root of the of x instantaneous price 
https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus#L120
We use FEE_DOWN because we want to decrease the perceived amount of
 reserve tokens leaving the pool and to decrease the observed amount of LP tokens being burned
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L459-L461




