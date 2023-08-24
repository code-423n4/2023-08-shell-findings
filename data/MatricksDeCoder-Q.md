### L-1 - License used for ABDKMath64x64 library 

[// SPDX-License-Identifier: BSD-4-Clause](https://github.com/abdk-consulting/abdk-libraries-solidity/blob/5e1e7c11b35f8313d3f7ce11c1b86320d7c0b554/ABDKMath64x64.sol#L1C1-L1C41)

Fixed Point Math library ABDKMath64x64 by ABDK that is used in the contracts makes use of an unusually used license in web3 BSD-4-Clause. The notes in license also state that....
["All advertising materials mentioning features or use of this software must display the following acknowledgement:This product includes software developed by the organization.](https://spdx.org/licenses/BSD-4-Clause.html)

**Impact**-> Not understanding or using this library correctly may result in dispute, legal, reputational, or other unforeseen consequences for the project.  

**Recommendation** -> Although ABDKMath64x64 is one of the best Fixed point and efficient library, it is recommended to fully understand license usage, implement usage requirements, or consider alternative Fixed Point Math libraries with licenses that may be well suited for the project. 

### NC-1 - Underscore function parameters 

All function parameters in the contracts do not have underscore formate e.g function do(uint256 _param)

**Impact**-> It results in contracts that are not following general accepted practises, styles and standards, affects readability, maintainability of the code 

**Recommendation** -> Recommended to make all function parameters have a leading underscore 

### NC-2 - Implicit use of uint

There are instances where uint is used without being explicit if uint without defining if e.g uint8, uint32 uint256 or what specific one was intended 

[EvolvingProteus.sol line 259, ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L259)

[EvolvingProteus.sol line 260, ABDKMath64x64.divu(uint(MAX_PRICE_RATIO),](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L260)

**Impact**-> This impacts code readability, code interpretability, may even introduce errors as there may be misunderstanding if smaller uint were intended. It is best practise to explicitly state uint as uint256 if that is intended desire

**Recommendation** -> Recommended every unit usage explicitly state its size e.g uint256 if default for uint was intended or other size as intended e.g uint8, uint16...etc 


### NC-3 - Top level declarations must be surrounded by 2 blank lines

There are instances where the style guide is not following best practises or recommendations for Soldity Style Guide. Such case is not having 2 blank lines between top level declarations. Examples include the below 
 [lines 135-137](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L135)
```solidity 
    }
} // End of library LibConfig

contract EvolvingProteus is ILiquidityPoolImplementation { //Start contract not separated 2 lines above
```

**Impact**-> This impacts code readability, code interpretability,  It is best to follow style guide

**Recommendation** -> Ensure style guide, best formatting, best layout is followed, especially surrounding top level declarations with 2 blank lines in cases mentioned and any other cases in code not mentioned but that need fixing. Check carefully all code and resolve all issues 

If you use pragma experimental SMTChecker;, then you get additional safety warnings which are obtained by querying an SMT solver. 

### NC-4 - Prefix Custom Errors with Contract Name 

[EvolvingProteus.sol lines 222-229](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L222C5-L229C46)
```solidity 
    error AmountError();
    error BalanceError(int256 x, int256 y);
    error BoundaryError(int256 x, int256 y);
    error CurveError(int256 errorValue); 
    error InvalidPrice();
    error MinimumAllowedPriceExceeded();
    error MaximumAllowedPriceExceeded();
    error MaximumAllowedPriceRatioExceeded();
```

It is best to prefix the events with contract name EvolvingProteus so that it is explicitly clear where the errors are arising from. 

**Impact**-> Without this it can be harder to follow, debug, trace, errors understand where the error is arising from, given that in Solidity events can be passed along a chain and resulting error may be from somewhere else. 

**Recommendation** -> Recommended to prefix as below 
```
    error EvolvingProteus__AmountError();
    error EvolvingProteus__BalanceError(int256 x, int256 y);
    error EvolvingProteus__BoundaryError(int256 x, int256 y);
    error EvolvingProteus__CurveError(int256 errorValue); 
    error EvolvingProteus__InvalidPrice();
    error EvolvingProteus__MinimumAllowedPriceExceeded();
    error EvolvingProteus__MaximumAllowedPriceExceeded();
    error EvolvingProteus__MaximumAllowedPriceRatioExceeded();
```



