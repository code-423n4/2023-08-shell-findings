# Low Risk and Non-Critical Issues

# Summary
[L-01] Use a modifier for access control (09 Instances)
[L 02] Missing initializer modifier on constructor (01 Instances)
[L-03] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values (14 Instances)
[N-01] Function writing that does not comply with the Solidity Style Guide


# [L-01] Use a modifier for access control (09 Instances)

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

Link to the code:
1.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L98
2.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L107
3.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L529
4.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L547
5.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L577
6.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L626
7.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L640
8.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L810
9.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L812


# [L 02] Missing initializer modifier on constructor (01 Instances)
OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. Ensure that the modifier is applied to the implementation contract. 
If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

Link to the code:
1.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L243


# [L-03] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values (14 Instances)

Link to the code:
1.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L146
2.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L259
3.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L260
4.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L337
5.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L379
6.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L415
7.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L452
8.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L489
9.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L589
10.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L660
11.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709
12.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L710
13.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716
14.	https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L830



# [N-01] Function writing that does not comply with the Solidity Style Guide

All Contracts
Order of functions should help readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered i.e.;
constructor
external
public
internal
private
within a grouping, place the view and pure functions last
