## QA Summary<a name="QA Summary">
Low Severity Risks

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Incorrect calculation of `b` variable | 1 |

Total: 1 contexts over 1 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Use a single file for system wide constants | 14 |
| [NC&#x2011;2](#NC&#x2011;2) | Visibility should be set explicitly rather than defaulting to `internal` | 3 |
| [NC&#x2011;3](#NC&#x2011;3) | Library and contracts should be in separate files | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Rename function names to be more comprehensible | 3 |
| [NC&#x2011;5](#NC&#x2011;5) | No need to check same condition twice | 2 |
| [NC&#x2011;6](#NC&#x2011;6) | Incorrect documentation | 3 |

Total: 26 contexts over 6 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Incorrect calculation of `b` variable

The comment for `function b` states that `@notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price`. However, no `inv` is being executed.
Like `function a` it should include `.inv()` according to the documentation.

#### <ins>Proof Of Concept</ins>

```solidity
123: function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L123-L125


This incorrect calculation can potentially effect the `_getUtility`, `_getPointGivenXandUtility`, `_getPointGivenYandUtility` functions that call `config.b()` such as:

```solidity
707: int128 b = config.b(); 
744: int128 b = config.b();
775: int128 b = config.b();
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L707
https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L744
https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L775

Which in turn can affect functions such as the `_swap` function.



## Non Critical Issues


### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Use a single file for system wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example `constants.sol`, use inheritance to access these values)

This will help with readability and easier maintenance for future changes.

`constants.sol` Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution

#### <ins>Proof Of Concept</ins>


```solidity
File: EvolvingProteus.sol

49: int128 constant ABDK_ONE = int128(int256(1 << 64));
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L49

```solidity
File: EvolvingProteus.sol

146: uint256 constant INT_MAX = uint256(type(int256).max);
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L146

```solidity
File: EvolvingProteus.sol

151: int256 constant MIN_BALANCE = 10**12;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L151

```solidity
File: EvolvingProteus.sol

157: int128 constant MAX_M = 0x5f5e1000000000000000000;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L157

```solidity
File: EvolvingProteus.sol

163: int128 constant MIN_M = 0x00000000000002af31dc461;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L163

```solidity
File: EvolvingProteus.sol

169: int256 constant MAX_PRICE_VALUE = 1844674407370955161600000000;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L169

```solidity
File: EvolvingProteus.sol

175: int256 constant MIN_PRICE_VALUE = 184467440737;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L175

```solidity
File: EvolvingProteus.sol

181: uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L181

```solidity
File: EvolvingProteus.sol

186: uint256 public constant BASE_FEE = 800;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L186

```solidity
File: EvolvingProteus.sol

191: uint256 constant FIXED_FEE = 10**9;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L191

```solidity
File: EvolvingProteus.sol

196: int256 constant MULTIPLIER = 1e18;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L196

```solidity
File: EvolvingProteus.sol

201: int256 constant MAX_PRICE_RATIO = 10**4; // to be comparable with the prices calculated through abdk math
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L201

```solidity
File: EvolvingProteus.sol

206: bool constant FEE_UP = true;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L206

```solidity
File: EvolvingProteus.sol

211: bool constant FEE_DOWN = false;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L211




### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Visibility should be set explicitly rather than defaulting to `internal`

#### <ins>Proof Of Concept</ins>


```solidity
File: EvolvingProteus.sol

146: uint256 constant INT_MAX = uint256(type(int256).max);
181: uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;
191: uint256 constant FIXED_FEE = 10**9;

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L146

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L181

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L191




### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Library and contracts should be in separate files

Currently the `EvolvingProteus.sol` contract file contains the `LibConfig` library and the `EvolvingProteus` contract.
It is recommended to separate library and contracts to separate files and inherit/import the library accordingly.


### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Rename function names to be more comprehensible


#### <ins>Proof Of Concept</ins>

```solidity
File: EvolvingProteus.sol

89: function t(Config storage self) public view returns (int128) {

115: function a(Config storage self) public view returns (int128) {

123: function b(Config storage self) public view returns (int128) {

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L89

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L115

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L123


### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> No need to check same condition twice

There is no need to call `if (specifiedToken == SpecifiedToken.X)` again, the calls can be inlined to the initial `if (specifiedToken == SpecifiedToken.X)` block.

#### <ins>Proof Of Concept</ins>

```diff
File: EvolvingProteus.sol

506: function _swap(
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
+++             computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
+++            _checkBalances(xi + specifiedAmount, yi + computedAmount);
            } else {
                int256 fixedPoint = yi + roundedSpecifiedAmount;
                (xf, yf) = _findFinalPoint(
                    fixedPoint,
                    utility,
                    _getPointGivenYandUtility
                );
+++             computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
+++            _checkBalances(xi + computedAmount, yi + specifiedAmount);
            }
        }
        
        // balance checks with consideration the computed amount
---        if (specifiedToken == SpecifiedToken.X) {
---            computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
---            _checkBalances(xi + specifiedAmount, yi + computedAmount);
---        } else {
---            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
---            _checkBalances(xi + computedAmount, yi + specifiedAmount);
---        }
    }

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L506


```diff
File: EvolvingProteus.sol

607: function _lpTokenSpecified(
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
+++         computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
+++         _checkBalances(xi + computedAmount, yf);
        else
            (xf, yf) = _findFinalPoint(
                xi,
                uf,
                _getPointGivenXandUtility
            );
+++         computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
+++         _checkBalances(xf, yi + computedAmount);

        // balance checks with consideration the computed amount
---        if (specifiedToken == SpecifiedToken.X) {
---            computedAmount = _applyFeeByRounding(xf - xi, feeDirection);
---            _checkBalances(xi + computedAmount, yf);
---        } else {
---            computedAmount = _applyFeeByRounding(yf - yi, feeDirection);
---            _checkBalances(xf, yi + computedAmount);
---        }
    }

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L607



### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Incorrect documentation


#### <ins>Proof Of Concept</ins>

```solidity
File: EvolvingProteus.sol
450:    // amount cannot be less than 0
        require(result < 0);   // <= @audit documentation states amount cannot be less than 0, but requires amount to be less than 0

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L450

```solidity
File: EvolvingProteus.sol
455: /**
     * @dev Given an input amount of the LP token, we compute an amount of
     *  a reserve token that must be output to decrease the pool's utility in
     *  proportion to the pool's decrease in total supply of the LP token.
     * @dev We use FEE_UP because we want to increase the perceived amount of
     *  reserve tokens leaving the pool and to increase the observed amount of
     *  LP tokens being burned.
     */
    function withdrawGivenInputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 totalSupply,
        uint256 burnedAmount,
        SpecifiedToken withdrawnToken
    ) external view returns (uint256 withdrawnAmount) {
        // lp amount validations against the current balance
        require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

        int256 result = _lpTokenSpecified(
            withdrawnToken,
            -int256(burnedAmount),
            FEE_DOWN,    // <= @audit documentation states FEE_UP but FEE_DOWN is actually used
            int256(totalSupply),
            int256(xBalance),
            int256(yBalance)
        );

        // amount cannot be less than 0
        require(result < 0);   // <= @audit documentation states amount cannot be less than 0, but requires amount to be less than 0
        withdrawnAmount = uint256(-result);
    }
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L455-L490


```solidity
File: EvolvingProteus.sol
487:    // amount cannot be less than 0
        require(result < 0);   // <= @audit documentation states amount cannot be less than 0, but requires amount to be less than 0
        
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L487