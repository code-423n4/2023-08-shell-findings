## 1. better to use the notation of `e` rather than `**` to denote the power of 10
4 instances
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L181
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L191
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L201

## 2. redundant constants declared
All occurences of `MAX_PRICE_VALUE` can be replaced with `MAX_M`. Also all occurences of `MIN_PRICE_VALUE` can be replaced with `MIN_M`
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L157a-L175

## 3. Redundant condition of `a<0`and `b<0`
The values of `a` and `b` has been derived from the function `a()` and `b()` from the library `LibConfig`. These functions uses `sqrt()` function from `ABDKMath64x64 library` which never returns negative values hence never let the values of `a` and `b` to be negative. This makes the condition `a<0 && b<0` redundant.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720