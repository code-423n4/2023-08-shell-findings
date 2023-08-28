### L01- Min price is different than specified in code comment
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L163
The comment reads:
```
The minimum slope (balance of y reserve) / (balance of x reserve)
This limits the pool to having at most 10**8 x for each y.
```
The value of MIN_M = 0x00000000000002af31dc461
Since this is a representation of ABKD64x64 number, the decimal value is 0x00000000000002af31dc461/2**64 = 10^-10.

So the actual min price is 10^-10, which means having at most 10**10 x for each y, not 10**8.