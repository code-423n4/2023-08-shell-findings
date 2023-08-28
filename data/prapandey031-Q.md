**Low Risk Issues**
----------------------------------

**[LOW-1] Swap operations can result in DoS when a malicious attacker would front-run a swap transaction by depositing reserved tokens:**

A malicious actor could DoS the swap operations by inflating xBalance or yBalance.

**Proof of Concept:**
The swap functions: 
```
swapGivenOutputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 outputAmount,
        SpecifiedToken outputToken
    ) external view returns (uint256 inputAmount)
```
and 
```
swapGivenInputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 inputAmount,
        SpecifiedToken inputToken
    ) external view returns (uint256 outputAmount)
```
have a check for ```_checkAmountWithBalance(uint256 balance, uint256 amount)```:

1) https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L283 (LOC-283):
```
function swapGivenInputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 inputAmount,
        SpecifiedToken inputToken
    ) external view returns (uint256 outputAmount) {
        // input amount validations against the current balance
        require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );

        _checkAmountWithBalance(
            (inputToken == SpecifiedToken.X) ? xBalance : yBalance,
            inputAmount
        );

        int256 result = _swap(
            FEE_DOWN,
            int256(inputAmount),
            int256(xBalance),
            int256(yBalance),
            inputToken
        );
        // amount cannot be less than 0
        require(result < 0);

        // output amount validations against the current balance
        outputAmount = uint256(-result);
        _checkAmountWithBalance(
            (inputToken == SpecifiedToken.X) ? yBalance : xBalance,
            outputAmount
        );
    }
```
2) https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L322 (LOC-322):

```
function swapGivenOutputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 outputAmount,
        SpecifiedToken outputToken
    ) external view returns (uint256 inputAmount) {
        // output amount validations against the current balance
        require(
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );
        _checkAmountWithBalance(
            outputToken == SpecifiedToken.X ? xBalance : yBalance,
            outputAmount
        );

        int256 result = _swap(
            FEE_UP,
            -int256(outputAmount),
            int256(xBalance),
            int256(yBalance),
            outputToken
        );

        // amount cannot be less than 0
        require(result > 0);
        inputAmount = uint256(result);

        // input amount validations against the current balance
        _checkAmountWithBalance(
            outputToken == SpecifiedToken.X ? yBalance : xBalance,
            inputAmount
        );
    }
```

Consider a scenario in which the ```yBalance = 10**6 ether``` and ```xBalance = 10**6 ether``` (equality for keeping things simple). To make it simpler, consider tokenY and tokenX to be Ocean wrapped-USDC and Ocean wrapped-USDT. Now, consider a swap transaction that calls 
```
swapGivenInputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 inputAmount,
        SpecifiedToken inputToken
    ) external view returns (uint256 outputAmount)
```
with ```inputAmount = 1.5*(10**-5) ether``` tokens and ```inputToken = tokenY```. Now, the ```_checkAmountWithBalance(uint256 balance, uint256 amount)``` would be (for tokenY):
```
10**6 ether / 1.5(10**-5) ether = 10**11 / 1.5 (< 10**11)
```
Therefore, this transaction would proceed to ```_swap()``` function. However, consider the following attack:

Step-1) A malicious attacker front-runs the swap transaction by increasing ```yBalance``` of the pool to 1.5*(10**6) ether by depositing 500k ether more tokenY (in this case around $500k worth tokens).

Step-2) The swap transaction now gets revert as the call to ```_checkAmountWithBalance(uint256 balance, uint256 amount)``` in LOC-283 would now revert:
```
balance / amount = 1.5*(10**11) ether / 1.5(10**-5) ether = 10**11 (this would revert!)
```

Step-3) The malicious attacker gets back their deposited tokenY (with fees deducted) by withdrawing using their LP tokens.

This way any swap transaction (probably those that have ```balance / amount``` ratio more closer to and slightly less than 10**11) can be DoS.

The liklihood is LOW (as it would be a loss for an attacker since fees are deducted upon withdrawal and they would not get all of their liquidity back upon withdrawal).
The severity is MEDIUM (since swap operations can face DoS).

***Recommended Mitigation Steps:***
The mitigation for this issue can be preventing any depositor from withdrawing their reserve tokens in the same block in which they deposited. Perhaps, allowing withdrawal for a depositor some blocks after their deposit. This would disincentivize the attackers from depositing their liquidity to create issues. 
 

**[LOW-2] Missing check for ```_checkBalances(int256 x, int256 y)``` in ```_reserveTokenSpecified()``` could result in DoS of some critical operations:**

Some critical operations of the protocol can face DoS due to missing check for ratio of reserve token balances in ```_reserveTokenSpecified()``` function.

**Proof of Concept:**
There is an invariant that, upon getting broken up, would result in ```BoundaryError(x,y)```. The check is implemented through the function ```_checkBalances(int256 x, int256 y)```.
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L809:
```
function _checkBalances(int256 x, int256 y) private pure {
        if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);
        int128 finalBalanceRatio = y.divi(x);
        if (finalBalanceRatio < MIN_M) revert BoundaryError(x,y);
        else if (MAX_M <= finalBalanceRatio) revert BoundaryError(x,y);
    }
```

Every function, that would change the xBalance or yBalance of the pool, calls this check except one - ```_reserveTokenSpecified()```.
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L563
```
function _reserveTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {
        int256 xf;
        int256 yf;
        int256 ui;
        int256 uf;
        {
            // calculating the final price points considering the fee
            if (specifiedToken == SpecifiedToken.X) {
                xf = xi + _applyFeeByRounding(specifiedAmount, feeDirection);
                yf = yi;
            } else {
                yf = yi + _applyFeeByRounding(specifiedAmount, feeDirection);
                xf = xi;
            }
        }

        ui = _getUtility(xi, yi);
        uf = _getUtility(xf, yf);

        uint256 result = Math.mulDiv(uint256(uf), uint256(si), uint256(ui));
        require(result < INT_MAX);   
        int256 sf = int256(result);

        // apply fee to the computed amount
        computedAmount = _applyFeeByRounding(sf - si, feeDirection);
    }
```
This would create DoS for some critical functionalities of the protocol such as, depositing reserve tokens, withdrawing them and swap operations. For eg, let us take the scenario of deposit being DoS.

Consider the protocol starts with an initial liquidity of xBalance = yBalance = 5 ether.

Since the ```_reserveTokenSpecified()``` function doesn't restrict the amount of tokenX or tokenY deposited (because it does not call ```_checkBalances(int256 x, int256 y)``` anywhere), we can have a situation (which would be rare) in which the xBalance becomes 10**9 ether and yBalance stays at 5 ether (this can happen if users keep depositing only tokenX via ```depositGivenInputAmount()```). 

For now, the ratio:
```
yBalance / xBalance = 5 / 10**9 (< MIN_M)
```

Now, consider a user would want to deposit tokenX through ```depositGivenOutputAmount()``` function.

The call would go to ```_lpTokenSpecified()``` function (at LOC-404). The ```_lpTokenSpecified()``` function is:
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L607
```
function _lpTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {
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
Now:
```
(xi + computedAmount) would be greater than 10**9 ether (since tokenX is deposited)
yf = yi = 5 ether
Therefore, after the deposit, yf / (xi + computedAmount) would still be less than MIN_M.
```
At LOC-642, ```_checkBalances()``` is called. This would now revert (as, yf / (xi + computedAmount) would still be less than MIN_M) (LOC-812).

This way, the user would not be able to deposit tokenX through ```depositGivenOutputAmount()``` function.

The same thing would happen if:
1) A user would call ```withdrawGivenInputAmount()``` to withdraw tokenY.
2) A user swaps tokenX for getting tokenY out of the pool.

This follows a pattern: the DoS happens when ```_checkBalances(int256 x, int256 y)``` is called during the execution and yBalance / xBalance gets below MIN_M (either by withdrawing tokenY or by depositing tokenX).

Another scenario would be to have yBalance = 10**9 ether and xBalance = 5 ether initially. Now, any transaction that would push the balance ratio above MAX_M would revert, thereby, DoS for critical operations.

Likelihood is LOW (since it would be rare that the users keep calling ```depositGivenInputAmount()``` to deposit only one of the reserve token to push the balance ratio out of the boundaries).
The severity is MEDIUM (since DoS of critical functionalities).

***Recommended Mitigation Steps:***
Add ```_checkBalances(int256 x, int256 y)``` at the end of ```_reserveTokenSpecified()``` function.


**[LOW-3] Unsafe cast of discriminant from int256 to uint256:**

The utility is calculated by solving a quadratic equation using [Shridharacharya formula](https://mitacademys.com/quadratic-formula-class-10th/) in the function ```_getUtility()```:
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L701

```
function _getUtility(
        int256 x,
        int256 y
    ) internal view returns (int256 utility) {

        int128 a = config.a(); //these are abdk numbers representing the a and b values
        int128 b = config.b(); 

        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));

        int128 aQuad = (a.mul(b).sub(one));
        int256 bQuad = (a.muli(y) + b.muli(x));
        int256 cQuad = x * y;

        int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

        if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
        else utility = (r0 > r1) ? r0 : r1;
        
        if (utility < 0) revert CurveError(utility);
    }
```
At LOC-716, the discriminant is casted from int256 to uint256 without checking if it is positive:
```
int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
```
A quadratic equation has no solutions when the discriminant is negative. However, here, even if it gets negative, 2**256 would be added to it (due to cast to uint256). This would lead to a wrongly calculated utility in a case when there should be no solutions (that is, no utility). 

Liklihood - LOW
Severity - LOW

***Recommended Mitigation Steps:***
Check for discriminant being positive before casting it to uint256 at LOC-716.


**[LOW-4] No function to change the config / restart the evolution of the pool:**
The deployer sets the ```config``` variable in the constructor:
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L262

```
constructor(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) { 
        // price value checks
        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

        // at all times x price should be less than y price
        if (py_init <= px_init) revert InvalidPrice();
        if (py_final <= px_final) revert InvalidPrice();

        // max. price ratio check
        if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
        if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

        config = LibConfig.newConfig(py_init, px_init, py_final, px_final, duration);
      }
```

However, there is no function to change the config variable (in case the deployer sets a wrong config by mistake during deployment) and restart the evolution period.

Liklihood - LOW
Severity - Depends on what wrong values of the config has been set. 

***Recommended Mitigation Steps:***
It is recommended to add a function that lets the deployer (or, owner) change the config and restart the evolution process.


**Non Critical Issues**
----------------------------------

**[NC-1] Wrong comments:**

1) LOC-50, LOC-487 and LOC-295 should have the comment:
```
// amount must be less than 0
```
2) LOC-40 should have the comment:
```
Min reserve ratio: The ratio between the two reserves cannot fall below a certain ratio. Any transaction that would result in the pool going below this ratio will fail.
```
3) LOC-173 should have the comment:
```
The minimum price value calculated with abdk library equivalent to 10^10(wei)
```
4) LOC-459 should have the comment:
```
We use FEE_DOWN because we want to decrease the perceived amount of reserve tokens leaving the pool and to decrease the observed amount of LP tokens being burned.
```
5) LOC-120 should have the comment:
```
@notice Calculates the b variable in the curve eq which is basically a sq. root of x instantaneous price
```

***Recommended Mitigation Steps:***
It is recommended to correct all the mentioned comments.

 


