

1. Pack timestamp struct variables
[Here](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L15C2-L16C21 ) as `t_init` and `t_final` are both timestamp variables we can pack them into one storage slot by setting them as `uint128` this will save SLOAD calls.

2. Revert as early as possible to save gas

For example [here](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L340) we can move this beneath the first one in the function call on line 326 as `_swap` does not create any state changes the result of the checks should be the same, so we can call them at the start of the function call

3. Unnecessary function jumps can be removed to save gas
[This](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L796C1-L801C6) function can be removed as all it does is check for a revert scenario, instead we can replace it with just the if statement as such:
```solidity
if (balance / amount >= MAX_BALANCE_AMOUNT_RATIO) revert AmountError();
```

This will save gas as its not worth the extra opcodes when we can just replace it with the if statement

4. Use unchecked math when we have require statements that also check for over/underflow

[Here](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L441C1-L448C11) for example we have a require statement that checks that `result > 0` meaning in the case of an overflow we will revert, and inside `_reserveTokenSpecified` we have a check to make sure `result < INT_MAX` [here](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L590C7-L590C7) in the case of an underflow, so we can safely wrap all of this function call in an unchecked box to save gas.

The same optimization can be done for functions calling `_swap` and `_lpTokenSpecified` due to the internal calls of `_checkBalances` being made
