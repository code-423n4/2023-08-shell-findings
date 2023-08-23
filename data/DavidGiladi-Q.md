
### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Complex math in a single line | [Complex math in a single line](#complex-math-in-a-single-line) | 21 |
|[N-2] Do not calculate constants | [Do not calculate constants](#do-not-calculate-constants) | 4 |
|[N-3] Control structures do not follow the Solidity Style Guide | [Control structures do not follow the Solidity Style Guide](#control-structures-do-not-follow-the-solidity-style-guide) | 4 |
|[N-4] Custom error has no error details | [Custom error has no error details](#custom-error-has-no-error-details) | 5 |
|[N-5] Else block not required | [Else block not required](#else-block-not-required) | 1 |
|[N-6] Lack of Descriptive Reason Strings in require/revert Statements | [Lack of Descriptive Reason Strings in require/revert Statements](#lack-of-descriptive-reason-strings-in-requirerevert-statements) | 16 |
|[N-7] Library Should Be Defined in Separate Files From Their Usage | [Library Should Be Defined in Separate Files From Their Usage](#library-should-be-defined-in-separate-files-from-their-usage) | 1 |
|[N-8] Use return statement when there is a named return variable is redundant | [Use return statement when there is a named return variable is redundant](#use-return-statement-when-there-is-a-named-return-variablev) | 1 |
|[N-9] Import declarations should import specific identifiers, rather than the whole file | [Import declarations should import specific identifiers, rather than the whole file](#import-declarations-should-import-specific-identifiers-rather-than-the-whole-file) | 2 |
|[N-10] Whitespace in Expressions | [Whitespace in Expressions](#whitespace-in-expressions) | 1 |

Total: 10 issues

## Complex math in a single line
- Severity: Non-Critical
- Confidence: Medium

### Description
Complex arithmetic calculations in a single line can reduce code readability. It's often beneficial to split these into multiple lines or separate operations.

<details>

<summary>
There are 21 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 251          py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 252          px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L252](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L252)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 279          require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        )
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279-L281](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279-L281)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 319          require(
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        )
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L319-L321](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L319-L321)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 361          require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 397          require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 434          require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 471          require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 716          int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))))
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 717          int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER)
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L717](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L717)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 718          int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER)
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L718](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L718)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 720          a < 0 && b < 0
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 750          int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert)
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L750](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L750)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 751          int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert)
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L751](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L751)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 752          int256 f_2 = (f_1 * utility) / MULTIPLIER
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L752](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L752)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 781          int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L781](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L781)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 782          int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert )
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L782](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L782)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 783          int256 f_2 = (f_1 * utility) / (MULTIPLIER)
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L783](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L783)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 810          x < MIN_BALANCE || y < MIN_BALANCE
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L810](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L810)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 838          roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +
                FIXED_FEE
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L838-L841](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L838-L841)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 844          roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L844-L847](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L844-L847)


</details>

# 


## Control structures do not follow the Solidity Style Guide
- Severity: Non-Critical
- Confidence: High

### Description
Control structures should follow the One True Brace [OTB style](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures). This means: 1. Opening braces should be on the same line as the declaration. 
2. Closing braces should be on their own line, at the same indentation level as the beginning of the declaration. 
3. The opening brace should be preceded by a single space.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 506          function _swap(
        bool feeDirection,
        int256 specifiedAmount,
        int256 xi,
        int256 yi,
        SpecifiedToken specifiedToken
    ) internal view returns (int256 computedAmount) 
```
 These control structures in the function `_swap` should follow the One True Brace (OTB) style:

    - Line: 515:         {
    - Line: 525:         {

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L506-L554](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L506-L554)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 563          function _reserveTokenSpecified(
        SpecifiedToken specifiedToken,
        int256 specifiedAmount,
        bool feeDirection,
        int256 si,
        int256 xi,
        int256 yi
    ) internal view returns (int256 computedAmount) 
```
 These control structures in the function `_reserveTokenSpecified` should follow the One True Brace (OTB) style:

    - Line: 575:         {

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L563-L595](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L563-L595)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 796          function _checkAmountWithBalance(uint256 balance, uint256 amount)
        private
        pure
    
```
 These control structures in the function `_checkAmountWithBalance` should follow the One True Brace (OTB) style:

    - Line: 799:     {

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L796-L801](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L796-L801)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 824          function _applyFeeByRounding(int256 amount, bool feeUp)
        private
        pure
        returns (int256 roundedAmount)
    
```
 These control structures in the function `_applyFeeByRounding` should follow the One True Brace (OTB) style:

    - Line: 828:     {

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L824-L852](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L824-L852)


</details>

# 


## Custom error has no error details
- Severity: Non-Critical
- Confidence: High

### Description
Consider adding parameters to the error to indicate which user or values caused the failure.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 222          error AmountError();
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L222](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L222)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 226          error InvalidPrice();
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L226](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L226)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 227          error MinimumAllowedPriceExceeded();
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L227](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L227)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 228          error MaximumAllowedPriceExceeded();
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L228](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L228)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 229          error MaximumAllowedPriceRatioExceeded();
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L229](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L229)


</details>

# 

## Do not calculate constants
- Severity: Non-Critical
- Confidence: High

### Description
Due to how constant variables are implemented in Solidity (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas. While in most cases, the compiler will optimize these computations away, it is considered a best practice to write code that does not rely on the compiler optimization.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 151          int256 constant MIN_BALANCE = 10**12
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 181          uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L181](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L181)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 191          uint256 constant FIXED_FEE = 10**9
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L191](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L191)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 201          int256 constant MAX_PRICE_RATIO = 10**4
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L201](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L201)


</details>

# 


## Else block not required
- Severity: Non-Critical
- Confidence: High

### Description
This detector identifies unnecessary else blocks in the code. When an if-statement block ends with a return statement, any subsequent else block becomes superfluous and can be eliminated to reduce code complexity. 

Note that this check also applies to single-line else conditions. For instance, in 'return a > b ? a : b', the else condition is not needed.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 99          return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)))
```
Should not be inside else block.
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L99](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L99)


</details>

# 


## Lack of Descriptive Reason Strings in require/revert Statements
- Severity: Non-Critical
- Confidence: High

### Description
By enforcing the use of descriptive reason strings, you can make it easier for developers and users to identify the specific condition or requirement that caused the failure. This can be particularly helpful in cases where contracts are interacting with each other or when external systems are relying on contract behavior.

<details>

<summary>
There are 16 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 279          require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        )
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279-L281](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L279-L281)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 296          require(result < 0)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L296](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L296)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 319          require(
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        )
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L319-L321](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L319-L321)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 336          require(result > 0)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 361          require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L361-L366)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 378          require(result > 0)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L378](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L378)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 397          require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 414          require(result > 0)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L414](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L414)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 434          require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 451          require(result < 0)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L451](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L451)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 471          require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        )
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L471-L476)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 488          require(result < 0)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L488](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L488)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 590          require(result < INT_MAX)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L590](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L590)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 658          require(sf >= MIN_BALANCE)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L658](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L658)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 661          require(result < INT_MAX)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L661](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L661)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 842          require(roundedAbsoluteAmount < INT_MAX)
```
 add descriptive reason strings 

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L842](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L842)


</details>

# 


## Library Should Be Defined in Separate Files From Their Usage
- Severity: Non-Critical
- Confidence: High

### Description
Library should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 44          library LibConfig 
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L44-L135](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L44-L135)


</details>

# 


## Use return statement when there is a named return variable is redundant
- Severity: Non-Critical
- Confidence: High

### Description
In Solidity, when you define a named return variable for a function, it is not necessary to use a return statement to return a value from the function. The named return variable will automatically be assigned the return value when the function exits.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 663          return uf
```
 redundant return statements
 
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L663](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L663)


</details>

# 


## Import declarations should import specific identifiers, rather than the whole file
- Severity: Non-Critical
- Confidence: High

### Description
Import declarations should import specific identifiers, rather than the whole file, in order to avoid polluting the symbol namespace and to make flattened files smaller, which can speed up compilation.Using import declarations of the form import {< identifier_name >} from 'some/file.sol' can help achieve this by only importing the necessary identifiers from the external file.By following this convention, Solidity developers can improve code readability and reduce the size of compiled contracts, which can improve contract performance and reduce transaction costs on the blockchain

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 6          import "abdk-libraries-solidity/ABDKMath64x64.sol";
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L6](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L6)


- File: src/proteus/EvolvingProteus.sol
```
 
Line: 7          import "@openzeppelin/contracts/utils/math/Math.sol";
```

[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L7](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L7)


</details>

# 


## Whitespace in Expressions
- Severity: Non-Critical
- Confidence: High

### Description
Detects when whitespace usage in expressions does not conform to the Solidity style guide.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/proteus/EvolvingProteus.sol
```
 
Line: 137          contract EvolvingProteus is ILiquidityPoolImplementation 
```

```
// @audit: whitespace inside parenthesis
Line: 600            int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);


// @audit: whitespace inside parenthesis
Line: 620            int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;


// @audit: whitespace inside parenthesis
Line: 621            int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );


```
[https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L137-L854](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L137-L854)


</details>

# 
