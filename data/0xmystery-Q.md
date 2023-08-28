## Unreachable if block in function `_getUtility`
In function `_getUtility`: 

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720-L721

```solidity
        if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
        else utility = (r0 > r1) ? r0 : r1;
```
`a` and `b` are the output of a square root function below. Hence they are always greater than or equal to zero, making the if block unreachable at all times.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L111-L125

```solidity
    /**
       @notice Calculates the a variable in the curve eq which is basically a sq. root of the inverse of y instantaneous price
       @param self config instance
    */
    function a(Config storage self) public view returns (int128) {
        return (p_max(self).inv()).sqrt();
    }

    /**
       @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
       @param self config instance
    */
    function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }
```
## Use `uint256` instead of `uint`
In favor of explicitness, consider replacing the following instances of `uint` with `uint256`:

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L259-L260

```solidity
        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
```
## Typo errors
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L736
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L768

```solidity
     @ audit is was ==> is
     *  without caring about which particular function is was called.
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L39

```solidity
           @ audit a ==> A
           Max transaction amount: a transaction amount cannot be too large relative to the size of the reserves in the pool.
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L41

```solidity
           @ audit the ==> The
           Max reserve ratio: the ratio between the two reserves cannot go above a certain ratio. Any transaction that results in the reserves going beyond this ratio will fall.
```
## Skip `r1` if `disc` is zero
When the discriminant is zero, there can only be 1 unique solution. Here's a suggested code refactoring:

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716-L718

```diff
        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
-        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
+        if (disc != 0) int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
```
## Chain reorganization attack
As denoted in the [Moralis academy article](https://academy.moralis.io/blog/what-is-chain-reorganization):

"... If a node receives a new chain that’s longer than its current active chain of blocks, it will do a chain reorg to adopt the new chain, regardless of how long it is."

Depending on the outcome, if it ended up placing the transaction earlier than anticipated, many of the system function calls could backfire. With slippage protection, this should not affect a swap or an LP token minting. But it could interfere with other function calls involving governance, Ducth auction etc that the protocol might need to be aware of.     

(Note: On Ethereum this is unlikely but this is meant for contracts going to be deployed on any compatible EVM chain many of which like Polygon, Optimism, Arbitrum are frequently reorganized.)

## Use decimal instead of hexadecimal
Where possible, adopt the decimal (in lieu of hexadecimal) number system for readability. 

For instance, the constant assignment below may be refactored like it has been done so on constant [`MAX_PRICE_VALUE`](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L169C1-L169C68): 

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L157

```diff
-    int128 constant MAX_M = 0x5f5e1000000000000000000;
+    int128 constant MAX_M = 1844674407370955161600000000;
```
Similarly, the constant assignment below may be refactored like it has been done so on constant [`MIN_PRICE_VALUE`](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L175) 

```diff
-    int128 constant MIN_M = 0x00000000000002af31dc461;
+    int128 constant MIN_M = 184467440737;
```
## Comment mismatch on a(t) and b(t)
It is [p_max(t)](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L106-L109) and [p_min(t)](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L97-L100) (in stead of a(t) and b(t)), and [py_init, py_final, px_init and px_final](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L244-L247) (in stead of a_init, a_final, b_init and b_final) pertaining to the comment below.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L29-L35

```solidity
     The parameters "a" and "b" are calculated from the price. a = 1/sqrt(y_axis_price) and b = sqrt(x_axis_price). 
     We calculate a(t) and b(t) by taking the time-dependant linear interpolate between the initial and final values. 
     In other words, a(t) = (a_init * (1-t)) + (a_final * (t)) and b(t) = (b_init * (1-t)) + (b_final * (t)), where "t"
     is the percentage of time elapsed relative to the total specified duration. Since 
     a_init, a_final, b_init and b_final can be easily calculated from the input parameters (prices), this is a trivial
     calculation. config.a() and config.b() are then called whenever a and b are needed, and return the correct value for
     a or b and the time t. When the total duration is reached, t remains = 1 and the curve will remain in its final shape. 
```
## Comment mismatch on min reserve ratio
It does not make good sense where any transaction that would result in the pool going above this ratio will fail, as far as the min reserve ratio is concerned. It should be corrected as follows:

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L40

```diff
-           Min reserve ratio: The ratio between the two reserves cannot fall below a certain ratio. Any transaction that would result in the pool going above or below this ratio will fail.
+           Min reserve ratio: The ratio between the two reserves cannot fall below a certain ratio. Any transaction that would result in the pool going below this ratio will fail.
```
## Comment mismatch on BASE_FEE
It is accurate by saying that the fee is applied twice. 

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L182-L186

```solidity
    /** 
     @notice 
     Equivalent to roughly twenty-five basis points since fee is applied twice.
    */
    uint256 public constant BASE_FEE = 800;
```
This is because `BASE_FEE` is applied either in the if or the else block as is evidenced in the code logic below. Hence, the fee should be equivalent to roughly one half of twenty-five basis point.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L837-L847

```solidity
        if (feeUp) {
            roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +
                FIXED_FEE;
            require(roundedAbsoluteAmount < INT_MAX);
        } else
            roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE;
```
## Bit shifting should go beyond multiplication/division by two
It is known in the bot report that bit shifting is recommended for multiplication/division by two. However, this recommendation should go beyond all multiplication/division by a number comprising of n number of the factor two.

For instance, `*4` in the code line below may be replaced by `<<2`: 

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716

```diff
-        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
+        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)<<2)))));
```