### [L-01] Duration check is missing in the constructor

Lines of Affected Codes:
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L243-L263

I would suggest adding a duration check if it is greater than zero

### [L-02] Amount check is missing for swap, deposit and withdraw functions

It would be ideal to generate AmountError if `inputAmount` or `outputAmount` is zero.