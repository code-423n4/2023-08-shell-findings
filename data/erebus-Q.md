# Non-critical

## [N-01] Bad comment
In [EvolvingProteus.sol#L149-L151](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L149-L151) the comment

```
    /** 
     @notice 
     When a token has 18 decimals, this is one microtoken
    */ 
    int256 constant MIN_BALANCE = 10**12;
```

does not translates into the implemented code. The code states the minimum amount of microtokens, that is, `10**18/10**6 = 10**12`, but the comment states the value of a microtoken. Consider changing the comment to

```
    /** 
     @notice 
     A microtoken is 10**6, so for a 18 decimals token the min balance 
     in microtokens is 10**18 / 10**6 = 10**12
    */ 
    int256 constant MIN_BALANCE = 10**12;
```

which is more explicit and coherent with the code. For example, in [EvolvingProteus.sol#L187-L191](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L187C5-L191C40) the comment is coherent as it suggests the base fee is just one nanotoken

```
    /** 
     @notice 
     When a token has 18 decimals, this is 1 nanotoken
    */ 
    uint256 constant FIXED_FEE = 10**9;
```