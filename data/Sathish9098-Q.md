# QA Report (LOW/NON-CRITICAL)

##

## [L-1] Implementing Slippage Protection for Token Swaps to avoid unexpected loss 

### Impact
The ``result`` amount is not checked with ``minExpectedAmount``. The swap function may return less amount than expected. In this scenario the caller lose the tokens may receive less tokens than expected.

``result`` only checked with ``> 0 ``

### POC
```solidity
FILE: Breadcrumbs2023-08-shell/src/proteus/EvolvingProteus.sol

288: int256 result = _swap(
            FEE_DOWN,
            int256(inputAmount),
            int256(xBalance),
            int256(yBalance),
            inputToken
        );

327: int256 result = _swap(
            FEE_UP,
            -int256(outputAmount),
            int256(xBalance),
            int256(yBalance),
            outputToken
        );

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L327-L333

### Recommended Mitigation
The ``slippage tolerance`` should be set based on the ``user's risk tolerance`` and the amount of money they ``are willing to lose``.

```solidity

require(result > minOutAmount, "The amount is too low");

```

##

## [L-2] No expiration Deadline for Token Swaps 

### Impact
Advanced protocols like Automated Market Makers (AMMs) can allow users to specify a deadline parameter that enforces a time limit by which the transaction must be executed. Without a deadline parameter, the transaction may sit in the mempool and be executed at a much later time potentially resulting in a worse price for the user.

Protocols should allow users interacting with AMMs to set expiration deadlines; no expiration deadline may create a potential critical loss of funds vulnerability for any user initiating a swap.

### POC
```solidity
FILE: Breadcrumbs2023-08-shell/src/proteus/EvolvingProteus.sol

288: int256 result = _swap(
            FEE_DOWN,
            int256(inputAmount),
            int256(xBalance),
            int256(yBalance),
            inputToken
        );

327: int256 result = _swap(
            FEE_UP,
            -int256(outputAmount),
            int256(xBalance),
            int256(yBalance),
            outputToken
        );

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L327-L333

### Recommended Mitigation
Add ``deadline`` parameter to ``_swap()`` to avoid unexpected fud lose 

##

## [L-3] Hardcoded ``BASE_FEE`` and ``FIXED_FEE`` may cause problems in the future

### Impact


### POC
```solidity
FILE: 2023-08-shell/src/proteus/EvolvingProteus.sol

  uint256 public constant BASE_FEE = 800;
    /** 
     @notice 
     When a token has 18 decimals, this is 1 nanotoken
    */ 
    uint256 constant FIXED_FEE = 10**9;

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L186

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L169

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L175

### Recommended Mitigation
 Add setter functions to update ``BASE_FEE`` and ``FIXED_FEE``

```diff

+ function setBaseFee(uint256 newBaseFee) external onlyOwner {
+         BASE_FEE = newBaseFee;
+    }

+    function setFixedFee(uint256 newFixedFee) external onlyOwner {
+        FIXED_FEE = newFixedFee;
+    }

```
##

## [L-4] Code Logic and Comment Mismatch in ``require(result < 0)`` Check

### Impact
If the comment states that the ``amount cannot be less than 0,`` but the ``require(result < 0)`` is checking for a negative result, then indeed there is an inconsistency between the comment and the condition.

If the intention is to ensure that the ``amount`` (which seems to be represented by the result variable) should not be less than 0, then the correct condition would be require(result >= 0)

### POC

```solidity
FILE: Breadcrumbs2023-08-shell/src/proteus/EvolvingProteus.sol

295: // amount cannot be less than 0
296: require(result < 0);

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L295-L296

### Recommended Mitigation

```diff

+ 295: // Ensuring that the amount is not less than 0
+ 296: require(result >= 0);

```

##

## [L-5] Potential Denial-of-Service (DoS) Vulnerability in ``_checkAmountWithBalance``

### Impact
Validation of the ratio balance / amount against the predetermined threshold MAX_BALANCE_AMOUNT_RATIO, the function becomes susceptible to abuse. Malicious actors could exploit this vulnerability to trigger unnecessary reverts, resulting in a potential DoS attack.

### POC

```solidity
FILE: 2023-08-shell/src/proteus/EvolvingProteus.sol

283:   _checkAmountWithBalance(
284:            (inputToken == SpecifiedToken.X) ? xBalance : yBalance,
285:            inputAmount
286:        );

 function _checkAmountWithBalance(uint256 balance, uint256 amount)
        private
        pure
    {
        if (balance / amount >= MAX_BALANCE_AMOUNT_RATIO) revert AmountError();
    }

```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L283-L286

### Recommended Mitigation
Validate the input parameters ``balance`` and ``amount`` to ensure they adhere to valid ranges and types before invoking the ``_checkAmountWithBalance`` function.

##

## [L-6] Errors Not efficiently handled for _swap() function

he _swap function is not properly checked for errors. The _swap function could return an error if the input amount is too large or if the pool does not have enough liquidity. This error is not properly checked 

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L288-L294

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L327-L333

##

## [L-7] The functions ``swapGivenInputAmount()``,``swapGivenOutputAmount()``,``depositGivenInputAmount()``,``depositGivenOutputAmount()``,``withdrawGivenOutputAmount()``,``withdrawGivenInputAmount()`` marked as ``external view``, suggesting that it is meant to be a ``read-only operation``

The use of ``require statements``, ``calculations``, and ``potential external function calls`` could result in ``higher gas consumption`` than ``expected`` for a view function.

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L469

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L432

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L395

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L317

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L277


## [L-8] Use Latest version version of solidity

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L4

##

## [L-9] Add events for critical functions 

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L272

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L312

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L353

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L389

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L426

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L463

