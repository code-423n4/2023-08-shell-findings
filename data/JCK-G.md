

# GAS SUMMARY

| Number |  Issue  | Instances | Gas Saved |
|--------|---------|-----------|-----------|
|[G-01]| Use uint256(1)/uint256(2) instead for true and false boolean states | 7 | 119700 |
|[G-02]| Use constants instead of type(uintx).max | 1 |  |
|[G-03]| Not using the named return variable when a function returns, wastes deployment gas | 15 |  |
|[G-04]| storage variable can be replaced with calldata variable | 7 |  |
|[G-05]| Arithmatic operations can be unchecked, since there is no underflow or overflow | 17 | 1445 |
|[G-06]| Do not calculate constants | 4 |  |
|[G-07]| Use != 0 instead of > 0 for unsigned integer comparison | 10 | 30 |
|[G-08]| Cache function calls | 7 | 100 |
|[G-09]| Sort Solidity operations using short-circuit mode | 10 | 10000 |
|[G-10]| Optimize names to save gas | 37 | 814 |
|[G-11]| Using bools for storage incurs overhead | 3 | 51300 |
|[G-12]| Inverting the condition of an if-else-statement wastes gas | 2 | 6 |
|[G-13]| Use assembly for math (add, sub, mul, div) | 6 |  |

## [G-01] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.
see source:  
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

```solidity
file:  src/proteus/EvolvingProteus.sol

206    bool constant FEE_UP = true;

211    bool constant FEE_DOWN = false;

507    bool feeDirection,

566    bool feeDirection,

610    bool feeDirection,

824    function _applyFeeByRounding(int256 amount, bool feeUp)

829    bool negative = amount < 0 ? true : false;

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206

## [G-02] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file:  src/proteus/EvolvingProteus.sol

146    uint256 constant INT_MAX = uint256(type(int256).max);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L146

## [G-03] Not using the named return variable when a function returns, wastes deployment gas

```solidity
file:  src/proteus/EvolvingProteus.sol

272    function swapGivenInputAmount(
          uint256 xBalance,
          uint256 yBalance,
          uint256 inputAmount,
          SpecifiedToken inputToken
      ) external view returns (uint256 outputAmount) {

312   function swapGivenOutputAmount(
       uint256 xBalance,
       uint256 yBalance,
       uint256 outputAmount,
       SpecifiedToken outputToken
      ) external view returns (uint256 inputAmount) {

353  function depositGivenInputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 totalSupply,
        uint256 depositedAmount,
        SpecifiedToken depositedToken
    ) external view returns (uint256 mintedAmount) {

389  function depositGivenOutputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 totalSupply,
        uint256 mintedAmount,
        SpecifiedToken depositedToken
    ) external view returns (uint256 depositedAmount) {

426   function withdrawGivenOutputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 totalSupply,
        uint256 withdrawnAmount,
        SpecifiedToken withdrawnToken
    ) external view returns (uint256 burnedAmount) {

463   function withdrawGivenInputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 totalSupply,
        uint256 burnedAmount,
        SpecifiedToken withdrawnToken
    ) external view returns (uint256 withdrawnAmount) {

506  function _swap(
        bool feeDirection,
        int256 specifiedAmount,
        int256 xi,
        int256 yi,
        SpecifiedToken specifiedToken
    ) internal view returns (int256 computedAmount) {

563  function _reserveTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {

607 function _lpTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) {
    
652 function _getUtilityFinalLp(
        int256 si,
        int256 sf,
        int256 xi,
        int256 yi
    ) internal view returns (int256 uf) {

681  function _findFinalPoint(
        int256 fixedCoordinate,
        int256 utility,
        function(int256, int256)
            view
            returns (int256, int256) getPoint
    ) internal view returns (int256 xf, int256 yf) {

701 function _getUtility(
        int256 x,
        int256 y
    ) internal view returns (int256 utility) {

739  function _getPointGivenXandUtility(
        int256 x,
        int256 utility
    ) internal view returns (int256 x0, int256 y0) {

770  function _getPointGivenYandUtility(
        int256 y,
        int256 utility
    ) internal view returns (int256 x0, int256 y0) {

824   function _applyFeeByRounding(int256 amount, bool feeUp)
        private
        pure
        returns (int256 roundedAmount)

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L272-L277

## [G-04] storage variable can be replaced with calldata variable

In the EvolvingProteus.elapsed, EvolvingProteus.t, EvolvingProteus.p_min, EvolvingProteus.p_max, EvolvingProteus.a, EvolvingProteus.b and  EvolvingProteus.duration   the Config struct is passed into the function as a storage variable named self. But it is only read from and not written to within the scope of the function implementation. Hence it is recommended to make the self a calldata variable and read from calldata instead, to save on two extra SLOAD operations for the transaction.

```solidity
file:  src/proteus/EvolvingProteus.sol

81   function elapsed(Config storage self) public view returns (uint256) {
        return block.timestamp - self.t_init;
    }

89  function t(Config storage self) public view returns (int128) {
        return elapsed(self).divu(duration(self));
    }

97   function p_min(Config storage self) public view returns (int128) {
        if (t(self) > ABDK_ONE) return self.px_final;
        else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
    }

106   function p_max(Config storage self) public view returns (int128) {
        if (t(self) > ABDK_ONE) return self.py_final;
        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
    }

115  function a(Config storage self) public view returns (int128) {
        return (p_max(self).inv()).sqrt();
    }

123  function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }

132 function duration(Config storage self) public view returns (uint256) {
        return self.t_final - self.t_init;
    }

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L81

## [G-05] Arithmatic operations can be unchecked, since there is no underflow or overflow

```solidity
file:  src/proteus/EvolvingProteus.sol

530    int256 fixedPoint = xi + roundedSpecifiedAmount;

537    int256 fixedPoint = yi + roundedSpecifiedAmount;

548    computedAmount = _applyFeeByRounding(yf - yi, feeDirection);

549    _checkBalances(xi + specifiedAmount, yi + computedAmount);

551    computedAmount = _applyFeeByRounding(xf - xi, feeDirection);

552    _checkBalances(xi + computedAmount, yi + specifiedAmount);

578    xf = xi + _applyFeeByRounding(specifiedAmount, feeDirection);

581    yf = yi + _applyFeeByRounding(specifiedAmount, feeDirection);

618    si + _applyFeeByRounding(specifiedAmount, feeDirection),

641    computedAmount = _applyFeeByRounding(xf - xi, feeDirection);

642    _checkBalances(xi + computedAmount, yf);

644    computedAmount = _applyFeeByRounding(yf - yi, feeDirection);

645    _checkBalances(xf, yi + computedAmount);

713    int256 bQuad = (a.muli(y) + b.muli(x));

717    int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

750    int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);

781    int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L530


## [G-06] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
file:   src/proteus/EvolvingProteus.sol

151    int256 constant MIN_BALANCE = 10**12;

181    uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;

191    uint256 constant FIXED_FEE = 10**9;

201    int256 constant MAX_PRICE_RATIO = 10**4;

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151



## [G-07] Use != 0 instead of > 0 for unsigned integer comparison


```solidity
file:  src/proteus/EvolvingProteus.sol

336   require(result > 0);

278   require(result > 0);

414   require(result > 0);

451   require(result < 0);

488   require(result < 0);

720   if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;

723   if (utility < 0) revert CurveError(utility);

755   if (y0 < 0) revert CurveError(y0);

786   if (x0 < 0) revert CurveError(x0);

829   bool negative = amount < 0 ? true : false;
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336

## [G-08] Cache function calls

External calls are expensive as they are performed via the CALL/STATICCALL opcodes. Calls to internal functions can also be expensive when the internal functions themselves read from storage and/or perform external calls. If a function call such as the ones explained above is performed more than once, it should be cached to avoid incurring the costs multiple times.

Estimated Gas Saved: 240

### Cache _findFinalPoint to save gas

```solidity
file:  src/proteus/EvolvingProteus.sol

506       function _swap(
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

        int256 xf;
        int256 yf;
        // calculate final price points after the swap
        {

            int256 utility = _getUtility(xi, yi);

            if (specifiedToken == SpecifiedToken.X) {
                int256 fixedPoint = xi + roundedSpecifiedAmount;
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenXandUtility
                );
            } else {
                int256 fixedPoint = yi + roundedSpecifiedAmount;
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenYandUtility
                );
            }
        }
        
        // balance checks with consideration the computed amount
        if (specifiedToken == SpecifiedToken.X) {
            computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
            _checkBalances(xi + specifiedAmount, yi + computedAmount);
        } else {
            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
            _checkBalances(xi + computedAmount, yi + specifiedAmount);
        }
    }

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L506-L554

## [G-09] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
```

```solidity
file:   src/proteus/EvolvingProteus.sol

251     if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();

252     if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

720    if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;

810    if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251

```solidity
file:   src/proteus/EvolvingProteus.sol

279    require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );

319    require(
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );

361    require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

397    require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

434   require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

471    require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279-281



## [G‑10] Optimize names to save gas

### Avoid Single-Letter Names: While very short names might save a tiny amount of space, they can quickly become confusing for anyone reading your code, including yourself.

```solidity
file:   src/proteus/EvolvingProteus.sol

89     function t(Config storage self) public view returns (int128) {    /// @audit  function name

115    function a(Config storage self) public view returns (int128) {    /// @audit  function name

123    function b(Config storage self) public view returns (int128) {   /// @audit  function name

702    int256 x,   /// @audit  variable name

703    int256 y    /// @audit  variable name

706    int128 a = config.a();     /// @audit  variable name

707    int128 b = config.b();    /// @audit  variable name

740    int256 x,   /// @audit  variable name
 
771    int256 y,   /// @audit  variable name

774    int128 a = config.a();    /// @audit  variable name
 
775    int128 b = config.b();    /// @audit  variable name


```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L89

### Use Abbreviations Wisely: You can use abbreviations for longer words in your variable and function names, but again, do this judiciously. It's essential that your code remains readable and understandable.

```solidity
file:   src/proteus/EvolvingProteus.sol

509     int256 xi,     /// @audit  variable name

510     int256 yi,     /// @audit  variable name

522     int256 xf;     /// @audit  variable name

523     int256 yf;     /// @audit  variable name

567     int256 si,     /// @audit  variable name

568     int256 xi,     /// @audit  variable name

569     int256 yi      /// @audit  variable name

611     int256 si,     /// @audit  variable name

612     int256 xi,     /// @audit  variable name

613     int256 yi      /// @audit  variable name

624     int256 xf;     /// @audit  variable name

625     int256 yf;     /// @audit  variable name

653     int256 si,     /// @audit  variable name

654     int256 sf,     /// @audit  variable name

655     int256 xi,     /// @audit  variable name

656     int256 yi     /// @audit  variable name

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L509

### Use Shorter Names: Using shorter names for variables and functions can help reduce the size of your contract's bytecode, which in turn can lead to lower gas costs. However, make sure the names are still meaningful and not overly cryptic.

```solidity
file:  src/proteus/EvolvingProteus.sol

227    error MinimumAllowedPriceExceeded();   /// @audit error  name

228    error MaximumAllowedPriceExceeded();   /// @audit error  name

229    error MaximumAllowedPriceRatioExceeded();  /// @audit error  name

353    function depositGivenInputAmount(   /// @audit function name

389    function depositGivenOutputAmount(  /// @audit function name

426    function withdrawGivenOutputAmount(   /// @audit function name

530    int256 fixedPoint = xi + roundedSpecifiedAmount;  /// @audit  variable name

563    function _reserveTokenSpecified(   /// @audit function name

796    function _checkAmountWithBalance(uint256 balance, uint256 amount)   /// @audit function name

811    int128 finalBalanceRatio = y.divi(x);   /// @audit  variable name
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L227



## [G-11] Using bools for storage incurs overhead

Both state variables can potentially be set back and forth from true and false. This would result in a Gsset (20000 gas) everytime the values are set to true from false. We can instead use uint(1) in place of true and uint(2) in place of false and pay the Gsset (20000 gas) once during deployment (to set the slot to uint(1). This would save two Gsset (20000 gas). However, a more efficient mitigation would be to pack the variables into a slot with other variables that would inevitably be written to. Please see this finding for a more efficient solution.


```solidity
file: src/proteus/EvolvingProteus.sol

206   bool constant FEE_UP = true;

211   bool constant FEE_DOWN = false;

830   bool negative = amount < 0 ? true : false;

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206


## [G‑12] Inverting the condition of an if-else-statement wastes gas

```solidity
file:   src/proteus/EvolvingProteus.sol

829     bool negative = amount < 0 ? true : false;

830     uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L829

## [G-13] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.


```solidity
file: src/proteus/EvolvingProteus.sol

530    int256 fixedPoint = xi + roundedSpecifiedAmount;

537    int256 fixedPoint = yi + roundedSpecifiedAmount;

713    int256 bQuad = (a.muli(y) + b.muli(x));

714    int256 cQuad = x * y;

752    int256 f_2 = (f_1 * utility) / MULTIPLIER; 

783    int256 f_2 = (f_1 * utility) / (MULTIPLIER); 

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L530
