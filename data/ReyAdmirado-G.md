| | issue |
| ----------- | ----------- |
| 1 | [using > 0 costs more gas than != 0 when used on a uint in a require() statement](#) |
| 2 | [avoid an unnecessary sstore by not writing a default value for bools](#) |
| 3 | [public library function should be made private/internal](#) |
| 4 | [operation can be pre determined and given as a constant](#) |
| 5 | [Duplicated require()/revert() checks should be refactored to a modifier or function](#) |

## 1. using > 0 costs more gas than != 0 when used on a uint in a require() statement

This change saves 6 gas per instance. The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

- [EvolvingProteus.sol#L336](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L336)
- [EvolvingProteus.sol#L378](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L378)
- [EvolvingProteus.sol#L414](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L414)


## 2. avoid an unnecessary sstore by not writing a default value for bools

default value of the varible is false when its made, so we dont need to assign false to it

- [EvolvingProteus.sol#L211](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L211)


## 3. public library function should be made private/internal

Changing from public will remove the compiler-introduced checks for msg.value and decrease the contract’s method ID table size

- [EvolvingProteus.sol#L65](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L65)
- [EvolvingProteus.sol#L81](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L81)
- [EvolvingProteus.sol#L89](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L89)
- [EvolvingProteus.sol#L97](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L97)
- [EvolvingProteus.sol#L106](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L106)
- [EvolvingProteus.sol#L115](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L115)
- [EvolvingProteus.sol#L123](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L123)
- [EvolvingProteus.sol#L132](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L132)


## 4. operation can be pre determined and given as a constant

calculate the result of this part of code and save it in a constant because the result is always the same. this way we use less operations resulting in saving gas

`2 * MULTIPLIER`
- [EvolvingProteus.sol#L709](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709)

`MULTIPLIER * MULTIPLIER`
- [EvolvingProteus.sol#L751](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L751)
- [EvolvingProteus.sol#L782](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L782)

`FIXED_FEE * 2`
- [EvolvingProteus.sol#L834](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L834)


## 5. Duplicated require()/revert() checks should be refactored to a modifier or function

after the change mentioned in the bot report G-18 and G-19, there is a bunch of checks being used several times. we can use modifiers instead of them to save deployment gas

`xBalance < INT_MAX`
- [EvolvingProteus.sol#L280](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L280)
- [EvolvingProteus.sol#L320](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L320)
- [EvolvingProteus.sol#L363](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L363)
- [EvolvingProteus.sol#L399](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L399)
- [EvolvingProteus.sol#L436](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L436)
- [EvolvingProteus.sol#L473](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L473)

`yBalance < INT_MAX`
- [EvolvingProteus.sol#L280](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L280)
- [EvolvingProteus.sol#L320](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L320)
- [EvolvingProteus.sol#L364](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L364)
- [EvolvingProteus.sol#L400](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L400)
- [EvolvingProteus.sol#L437](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L437)
- [EvolvingProteus.sol#L474](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L474)

`totalSupply < INT_MAX`
- [EvolvingProteus.sol#L365](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L365)
- [EvolvingProteus.sol#L401](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L401)
- [EvolvingProteus.sol#L438](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L438)
- [EvolvingProteus.sol#L475](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L475)
