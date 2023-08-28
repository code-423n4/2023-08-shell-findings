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

With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

Recommendation Code:

contracts\library\UtilLib.sol:

- 168:         (bool success, ) = payable(_receiver).call{value: _amount}('');
+              address addr = payable(_receiver)
+              bool success;
+              assembly {
+              sent := call(gas(), _receiver, _amount, 0, 0, 0, 0)
+              }
  169:         if (!success) {
  170:             revert TransferFailed();
  171:         }

if () / require() statements that check input arguments should be at the top of the function
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

Reduce Gas Costs by Emitting Events Outside of For Loops
Placing events inside loops results in the event being emitted at each iteration, which can significantly increase gas consumption. By moving events outside the loop and emitting them globally, unnecessary gas expenses can be minimized, leading to more efficient and cost-effective contract execution.

Use nested if and, avoid multiple check combinations
Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

Do not calculate constants variables
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

State variables can be cached 

Don't cache if used once

Result of function call should be cached rather than call the function more than once

Reducing the number of state variable updates can save gas. In Ethereum, every storage operation costs a significant amount of gas. Therefore, optimizing the contract to minimize storage operations can lead to substantial gas savings.

Don't emit state variable 

Calldata instead of memory 



