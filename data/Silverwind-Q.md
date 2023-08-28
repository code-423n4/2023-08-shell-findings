# Unclear Comments

The same comment: `// amount cannot be less than 0` written in 6 different places ([295](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L295), [335](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L335), [377](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L377), [413](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L413), [450](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L450), [487](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L487)) for the same code: `require(result > or < 0)`. But in each one, the `result` is referring to different outcomes which are **swappableAmount**, **specifiedLpTokenAmount**, and **reservedTokenAmount**. And it leads to confusion when reading the contract.

## Solution

For making the code more understandable to other programmers, comments should be written more specifically.
