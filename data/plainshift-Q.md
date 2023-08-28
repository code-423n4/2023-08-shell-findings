### Unreachable code
[The condition of a or b being less than zero](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720) [can never be reached due to how it is being computed](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L115-L125), since [`p_max` and `p_min` are always positive](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251-L256).
Remove the if condition and just assign utility. 

### Incorrect comments

- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L29 - `a` is not `a = 1/sqrt(y_axis_price)` but rather `sqrt(1/y_axis_price)`.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L120 - It should be `Calculates the b variable in the curve eq which is basically a sq. root of the of x instantaneous price`.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L173 - Instead of `10^12` it should be `10^10`.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L295-L296 - The comment should be `Amount needs to be less than 0`.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L450-L451 - The comment should be `Amount needs to be less than 0`.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L459 - It should be written `FEE_DOWN` instead of `FEE_UP`.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L487-L488 - The comment should be `Amount needs to be less than 0`.
- https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L697 - `k` is not used in the code, for `EvolvingProteus` it is equal to 1 and should be removed from comments. 