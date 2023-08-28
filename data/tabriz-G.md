## [G-01] Use double if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

``` 

if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
else utility = (r0 > r1) ? r0 : r1;

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720

```

## [G-02] Rearranging order of state variable declarations to pack them will save storage slots and gas
put them in follwing order
    uint64
    bool
    address

```

  function _swap(
        bool feeDirection,
        int256 specifiedAmount,
        int256 xi,
        int256 yi,
        SpecifiedToken specifiedToken
    ) internal view returns (int256 computedAmount) {
        int256 roundedSpecifiedAmount;
        // calculating the amount considering the fee
        {
            roundedSpecifiedAmount = _applyFeeByRounding(
                specifiedAmount,
                feeDirection
            );
        }

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L506C3-L520C10

```
  function _reserveTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {
        
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L563C3-L584C10

```

 function _lpTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L610

```
## [G-03] >= costs less gas than >

```
 if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L259C8-L260C140

```
 require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279C8-L281C11

```

     require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L362

```
so on...

```


