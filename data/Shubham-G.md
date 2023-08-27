## Gas Optimization

| |Issue|Instances|
|-|:-|:-:|
| [G-01](#G-01) | Unnecessary computation of Dead Code in `_getUtility` function wastes gas | 1 |

## [G-01] Unnecessary computation of Dead Code in `_getUtility` function

The `_getUtility` function returns the `utility` based in the input x & y parameters. 
After performing the necessary calculations, there are checks to ensure the correct value of `utility` to be returned. 
But the `if` condition will never be executed because the value of `a` & `b` will never be less than 0.
Thus everytime the `else` statement will be executed.
As the function '_getUtility' is called on many times, unnecessary computation will cause wastage of gas.

```solidity

720:      if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;

```

