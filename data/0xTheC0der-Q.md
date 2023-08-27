## Low: Wrong NatSpec of `MIN_PRICE_VALUE`
The NatSpec of [EvolvingProteus.MIN_PRICE_VALUE](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L171-L175) is misleading.
```solidity
    /** 
     @notice 
     The minimum price value calculated with abdk library equivalent to 10^12(wei)
    */ 
    int256 constant MIN_PRICE_VALUE = 184467440737;
```
Because `MIN_PRICE_VALUE = 184467440737 = 2**64 / 10**8` which is the `abdk library equivalent to 10^10(wei)` and **not** `10^12(wei)` as written in the NatSpec. According to [L127 of the README](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/README.md?plain=1#L127), the minimum price should be `10^-8(ETH) = 10^10(wei)`, therefore we know that the value is correct and just the is NatSpec wrong.  
Anyways, this is a fundamental parameter of the protocol and such a comment might lead to a wrong assumption or subsequent error by anyone who is working with the contract.
