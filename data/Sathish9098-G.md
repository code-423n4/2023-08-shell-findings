# GAS OPTIMIZATION

##

## [G-1] State variables can be packed with less slots 

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas)

### ``t_init,t_final`` can be used uin128 instead of uint256 : Saves ``2000 GAS , 1 SLOT``

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L10-L17

#### Explanations

The ``block.timestamp`` in Solidity represents time in seconds since the ``Unix epoch (January 1, 1970)``. If we convert the maximum value that can be stored in a ``uint128 (2^128 - 1)`` to years, we can get an estimate of the maximum representable time span.

the current estimated age of the universe is about ``13.8 billion years``. The maximum uint128 timestamp value is astronomically larger than 13.8 billion years any practical application scenario you're likely to encounter.

```diff
FILE: 2023-08-shell/src/proteus/EvolvingProteus.sol

10: struct Config {
11:    int128 py_init;
12:    int128 px_init;
13:    int128 py_final;
14:    int128 px_final;
- 15:    uint256 t_init;
- 16:    uint256 t_final;
+ 15:    uint128 t_init;
+ 16:    uint128 t_final;
17: }

```