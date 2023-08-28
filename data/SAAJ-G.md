 
# Gas Optimizations Report

This report focuses on Shell Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Shell protocol are categorised into 03 main areas; with further multiple instances in each of the category.

## Summary
[G 01] Consolidating library functions can save gas by preventing external calls (02 Instances)
[G-02] Immutable has more gas efficiency than constant (10 Instances)
[G-03] Use uint256(1)/uint256(2) instead for true and false boolean states (03 Instances)


# [G 01] Consolidating library functions can save gas by preventing external calls (02 Instances)
Libraries of functions commonly used across different contracts come at the increased cost of gas usage at runtime because of the external calls.
Consider moving all the library functions internal to contract or to a single library to save gas from external calls.

Link to the code:
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L6
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L7


# [G-02] Immutable has more gas efficiency than constant (10 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.

Link to the Code:
1.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151
2.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L169
3.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L175
4.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L186
5.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L181
6.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L191
7.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L196
8.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L201
9.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206
10.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L211


# [G-03] Use uint256(1)/uint256(2) instead for true and false boolean states (03 Instances)

Boolean for storage if not used, saves Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Link to the Code:
1.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L206
2.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L211
3.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L829

