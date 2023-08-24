# G-01 Optimizable Return Statement
## Location
src/proteus/EvolvingProteus.sol: 663
## Description
The returned variable is specified in the function signature, but the return statement is still displayed in the function body, which will increase gas consumption.
## Recommendation
No need to explicitly call the return statement when the returned variable is specified in the function signature.
# G-02 Use Shift Operation Instead of Mul/Div
## Location
src/proteus/EvolvingProteus.sol: 716, 834
## Description
It is recommended to use shift operation instead of direct multiplication and division if possible, because shift operation is more gas-efficient.
## Recommendation
It is recommended to use shift operation instead of multiplication and division when possible.