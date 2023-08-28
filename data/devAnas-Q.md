## Code and comment mismatch

## Vulnerability details

### Impact
The code comment says: “// amount cannot be less than 0” but the code immediately after the comment implements a check “result < 0.” It is unclear if the comment is incorrect or the check is wrong. An incorrect check may have mathematical implications.

There are numerous instances throughout the codebase in different contracts.

### Proof of Concept
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L295
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L450
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L487

### Tools Used

Manual Review


### Recommended Mitigation Steps

Revisit comment and code to sync them by fixing the comment or the code whichever is incorrect.

