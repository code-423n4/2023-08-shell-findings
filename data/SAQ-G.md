## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 5 | - |
| [G-02] | tate variables should be cached in stack variables rather than re-reading them from storage | 2 | - |
| [G-03] | Not using the named return variable when a function returns, wastes deployment gas | 4 | - |
| [G-04] .| Require()/revert() statements that check input arguments should be at the top of the function | 2 | - |
| [G-05] | The result of function calls should be cached rather than re-calling the function | 2 | - |
| [G-06] | Don't initialize variables with default value | 2 | - |
| [G-07] | Duplicated require()/if() checks should be refactored to a modifier or function | 2 | - |
| [G-08] | Use constants instead of type(intx).max | 1 | - |
| [G-09] | Use Modifiers Instead of Functions To Save Gas | 4 | - |
| [G-10] | Do not calculate  constant | 4 | - |
| [G-11] | Use assembly for math (add, sub, mul, div) | 5 | - |
| [G-12] | Use != 0 instead of > 0 for unsigned integer comparison | 3 | - |
| [G-13] | Using Bools For Storage Incurs Overhead | 2 | - |
| [G-14] | State variables which are not modified within functions should be set as constants or immutable for values set at deployment | 1 | - |

## Gas Optimizations  

## [G-01] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
file: /src/proteus/EvolvingProteus.sol

90        return elapsed(self).divu(duration(self));

108        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));


404        int256 result = _lpTokenSpecified(
            depositedToken,
            int256(mintedAmount),
            FEE_UP,
            int256(totalSupply),
            int256(xBalance),
            int256(yBalance)
411        );


709        int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));

719        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L90


## [G-02] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function.
Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
Most of the times this if statement will be true and we will save 100 gas at a small possibility of 3 gas loss 

```solidity
file: /src/proteus/EvolvingProteus.sol

/// @audit the '  FIXED_FEE  ' & '  BASE_FEE  ' is state variables , which accessed second+ within a function.

834       if (absoluteValue < FIXED_FEE * 2) revert AmountError();   ///Found   FIXED_FEE

836        uint256 roundedAbsoluteAmount;

837        if (feeUp) {
            roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +                        ///Found   BASE_FEE
                FIXED_FEE;                                          ///Found   FIXED_FEE
            require(roundedAbsoluteAmount < INT_MAX);
        } else
            roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -                         ///Found   BASE_FEE
847                FIXED_FEE;                                        ///found   FIXED_FEE

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L834-L847


## [G-03] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.

```solidity
file: /src/proteus/EvolvingProteus.sol

/// @audit the ' outputAmount ' variabis is return on line 299

277    ) external view returns (uint256 outputAmount) {


/// @audit the ' inputAmount ' variabis is return on line 337

317    ) external view returns (uint256 inputAmount) {


/// @audit the ' roundedAmount ' variabis is return on line 849

827        returns (int256 roundedAmount)

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L277


```solidity
file: /src/proteus/EvolvingProteus.sol

/// @audit the ' uf ' variabis is return on line 663


657   ) internal view returns (int256 uf) {
        require(sf >= MIN_BALANCE);
        int256 ui = _getUtility(xi, yi);
        uint256 result = Math.mulDiv(uint256(ui), uint256(sf), uint256(si));
        require(result < INT_MAX);
        uf = int256(result);
        return uf;
664     }
```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L657-L664

## [G-04] Require()/revert() statements that check input arguments should be at the top of the function  

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

```solidity
file: /src/proteus/EvolvingProteus.sol

590        require(result < INT_MAX);   

842        require(roundedAbsoluteAmount < INT_MAX);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L590


## [G-05] The result of function calls should be cached rather than re-calling the function

The instances below point to the second+ call of the function within a single function

```solidity
file: /src/proteus/EvolvingProteus.sol

/// @audit the '  _findFinalPoint  ' function is second+ called within a single function on line 627, 633  and 531, 538

681    function _findFinalPoint(


/// @audit the '  _getUtilityFinalLp  ' function is second+ called within a single function on line 586, 587

652       function _getUtilityFinalLp(

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L681


## [G-06] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with itâ€™s default value costs unnecesary gas.

```solidity
file: /src/proteus/EvolvingProteus.sol

206    bool constant FEE_UP = true;

211    bool constant FEE_DOWN = false;

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206


## [G-07] Duplicated require()/if() checks should be refactored to a modifier or function   

To reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /src/proteus/EvolvingProteus.sol

529        if (specifiedToken == SpecifiedToken.X) {
547        if (specifiedToken == SpecifiedToken.X) {    


626        if (specifiedToken == SpecifiedToken.X)]
640        if (specifiedToken == SpecifiedToken.X)


```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L529


## [G-08] Use constants instead of type(intx).max

type(int120).max or type(int128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /src/proteus/EvolvingProteus.sol

146    uint256 constant INT_MAX = uint256(type(int256).max);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L146


## [G-09] Use Modifiers Instead of Functions To Save Gas

Gas savings can be significant, especially when you have complex conditions or multiple functions that require the same access control checks. However, it's important to note that using modifiers does incur a small additional gas cost for the extra function call overhead.
Overall, using modifiers instead of duplicating code in multiple functions can be an efficient way to save gas and improve the overall efficiency of Solidity smart contracts.

This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. 

Example:

```solitity
/// incorrect

function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
}
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }

/// correct

modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }

```

Instances:

```solidity
file: /src/proteus/EvolvingProteus.sol

272    function swapGivenInputAmount(
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
304    }


312    function swapGivenOutputAmount(
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
344    }


309    function depositGivenOutputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 totalSupply,
        uint256 mintedAmount,
        SpecifiedToken depositedToken
    ) external view returns (uint256 depositedAmount) {
        // lp amount validations against the current balance
        require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

        int256 result = _lpTokenSpecified(
            depositedToken,
            int256(mintedAmount),
            FEE_UP,
            int256(totalSupply),
            int256(xBalance),
            int256(yBalance)
        );

        // amount cannot be less than 0
        require(result > 0);
        depositedAmount = uint256(result);
416    }


652    function _getUtilityFinalLp(
        int256 si,
        int256 sf,
        int256 xi,
        int256 yi
    ) internal view returns (int256 uf) {
        require(sf >= MIN_BALANCE);
        int256 ui = _getUtility(xi, yi);
        uint256 result = Math.mulDiv(uint256(ui), uint256(sf), uint256(si));
        require(result < INT_MAX);
        uf = int256(result);
        return uf;
664    }


```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L272-L304


## [G-10] Do not calculate  constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity
file: /src/proteus/EvolvingProteus.sol

151    int256 constant MIN_BALANCE = 10**12;

181    uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;

191    uint256 constant FIXED_FEE = 10**9;

201    int256 constant MAX_PRICE_RATIO = 10**4; // to be comparable with the prices calculated through abdk math

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151


## [G-11] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /src/proteus/EvolvingProteus.sol

713        int256 bQuad = (a.muli(y) + b.muli(x));

714        int256 cQuad = x * y;

750       int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);

751        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);

752        int256 f_2 = (f_1 * utility) / MULTIPLIER;


```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L713-L714


## [G-12] Use != 0 instead of > 0 for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:
``` solidity
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```


```solidity
file: /src/proteus/EvolvingProteus.sol

336        require(result > 0);

378        require(result > 0);

414        require(result > 0);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336


## [G-13] Using Bools For Storage Incurs Overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

```solidity 
file: /src/proteus/EvolvingProteus.sol
 
206    bool constant FEE_UP = true;

211    bool constant FEE_DOWN = false;

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206


## [G-14] State variables which are not modified within functions should be set as constants or immutable for values set at deployment

Set state variable below as constant or immutable for values set at deployment. Ensure it is safe to do so

```solidity
file: /src/proteus/EvolvingProteus.sol

216    Config public config;

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L216