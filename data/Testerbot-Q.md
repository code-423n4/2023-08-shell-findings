# L-01 There is no check that the discriminant is greater than zero.

The `_getUtility` function is used to compute the `utility` in the curve equation. To solve it for `u`, the function uses the quadratic formula.
It starts by computing the discriminant:

```solidity
int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
``` 

It does not verify that `bQuad**2 - (aQuad.muli(cQuad)*4) >= 0`  before casting to `uint256`. Discriminants smaller than zero exist, meaning that the equation has its solutions in the complex numbers. This should not be the case here if the terms of the curve equation are correctly configured. Anyways, the discriminant should be validated to be greater than zero before casting it to `uint256` to avoid any mistakes.

# L-02 The contract only works for tokens with 18 Decimals.

Some assumptions around the code tell you that the code only works for tokens with 18 decimals. For example, the following constant:

```solidity
uint256 constant FIXED_FEE = 10**9;
```

This constant is a flat fee charged for every swap and liquidity addition or substraction. But, if a token were USDC, this fee means 1000 USDC, which is unreasonable. 

# NC-01 The function `_getUtility` makes an innecessary check:

The following line can be removed:

```solidity
if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
```

`a` and `b` are never going to be smaller than zero.

# NC-01 Incorrect comment in swapGivenInputAmount

```solidity
// amount cannot be less than 0
require(result < 0);
```

The correct comment should be `amount cannot be greater than 0`

# NC-02 Incorrect comment in withdrawGivenInputAmount

```solidity
// * @dev We use FEE_UP because we want to increase the perceived amount of
// *  reserve tokens leaving the pool and to increase the observed amount of
// *  LP tokens being burned.
```

Should be change to something similar as:

```solidity
// We use FEE_DOWN because we want to decrease the perceived amount of 
// LP tokens to burn and to decrease the observed reserve tokens taken out 
```

