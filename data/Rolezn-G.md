## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Activate the optimizer | 1 | - |
| [GAS&#x2011;2](#GAS&#x2011;2) | Avoid repeating computations | 2 | - |
| [GAS&#x2011;3](#GAS&#x2011;3) | Boolean variables are false by default and need not be explicitly assigned | 1 | 224 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 8 | 224 |
| [GAS&#x2011;5](#GAS&#x2011;5) | `internal` functions only called once can be inlined to save gas | 2 | 44 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Using `unchecked` blocks to save gas | 1 | 40 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states | 2 | 10000 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Using XOR (^) and AND (&) bitwise equivalents | 12 | 156 |
| [GAS&#x2011;9](#GAS&#x2011;9) | No need to check same condition twice | 2 | - |

Total: 31 contexts over 9 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Activate the optimizer

Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
	solidity: {
		version: "0.8.19",
		settings: {
			optimizer: {
				enabled: true,
				runs: 1000,
			},
		},
	},
};
```
Kindly visit the following site for further information:

https://docs.soliditylang.org/en/v0.8.19/using-the-compiler.html#using-the-commandline-compiler

Here is one particular example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```

#### <ins>Proof Of Concept</ins>




### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Avoid repeating computations

The following instances show only the first instance of the repeated computations.

#### <ins>Proof Of Concept</ins>


```solidity
File: EvolvingProteus.sol

840: (absoluteValue / BASE_FEE)

846: (absoluteValue / BASE_FEE)

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L840

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L846




### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Boolean variables are false by default and need not be explicitly assigned

This saves 224 gas in deployment cost.

#### <ins>Proof Of Concept</ins>


```solidity
File: EvolvingProteus.sol

211: bool constant FEE_DOWN = false;
```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L211




#### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

```solidity
File: EvolvingProteus.sol

296: require(result < 0);
451: require(result < 0);
488: require(result < 0);
336: require(result > 0);
378: require(result > 0);
414: require(result > 0);
590: require(result < INT_MAX);
661: require(result < INT_MAX);

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L296

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L451

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L488

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L336

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L378

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L414

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L590

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L661








### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> `internal` functions only called once can be inlined to save gas

`_getUtilityFinalLp` is called only once in `_lpTokenSpecified` function.

#### <ins>Proof Of Concept</ins>

```solidity
File: EvolvingProteus.sol

616: int256 uf = _getUtilityFinalLp(

652: function _getUtilityFinalLp

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L616

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L652




### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

```solidity
File: EvolvingProteus.sol

594: computedAmount = _applyFeeByRounding(sf - si, feeDirection);

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L594



### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from `true` to `false` can cost up to ~20000 gas rather than `uint256(2)` to `uint256(1)` that would cost significantly less.

#### <ins>Proof Of Concept</ins>


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






### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: EvolvingProteus.sol

251: if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
252: if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L251-L252

```solidity
File: EvolvingProteus.sol

284: (inputToken == SpecifiedToken.X) ? xBalance : yBalance,
301: (inputToken == SpecifiedToken.X) ? yBalance : xBalance,

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L284

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L301



```solidity
File: EvolvingProteus.sol

323: outputToken == SpecifiedToken.X ? xBalance : yBalance,
341: outputToken == SpecifiedToken.X ? yBalance : xBalance,

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L323

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L341



```solidity
File: EvolvingProteus.sol

529: if (specifiedToken == SpecifiedToken.X) {
529: if (specifiedToken == SpecifiedToken.X) {

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L529

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L529



```solidity
File: EvolvingProteus.sol

577: if (specifiedToken == SpecifiedToken.X) {

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L577

```solidity
File: EvolvingProteus.sol

626: if (specifiedToken == SpecifiedToken.X)
640: if (specifiedToken == SpecifiedToken.X) {

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L626

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L640



```solidity
File: EvolvingProteus.sol

810: if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);

```

https://github.com/code-423n4/2023-08-shell/tree/main/src/proteus/EvolvingProteus.sol#L810



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |



### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> No need to check same condition twice

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