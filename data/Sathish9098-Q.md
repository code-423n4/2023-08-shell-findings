# QA Report (LOW/NON-CRITICAL)

##

## [L-1] Hardcoded ``BASE_FEE`` and ``FIXED_FEE`` may cause problems in the future

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

## [L-2] Code Logic and Comment Mismatch in ``require(result < 0)`` Check

### Impact
If the comment states that the ``amount cannot be less than 0,`` but the ``require(result < 0)`` is checking for a negative result, then indeed there is an inconsistency between the comment and the condition.

If the intention is to ensure that the ``amoun`` (which seems to be represented by the result variable) should not be less than 0, then the correct condition would be require(result >= 0)

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

## [L-3] Potential Denial-of-Service (DoS) Vulnerability in ``_checkAmountWithBalance``

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




## [L-1] Use Latest version version of solidity

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L4