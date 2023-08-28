## Title

The usage of the >= instead of > operator is more appropriate.

### Code snippet 

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L97-L100

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L106-L109

### explanation

if t(self) == 1 then the price calculated will be the same as p.final so there is no need for additional computation, hence the usage of >= will avoid it.

## Title
The usage of the =< instead of < operator is more appropriate.

### Code snippet 

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L830-L835

### explanation

Given the comments, it appears the `    if (absoluteValue < FIXED_FEE * 2) revert AmountError(); ` would benefit from using <= instead of < 