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

does not translates into the implemented code. The code states the minimum amount of microtokens, that is, `10**18/10**6 = 10**12`, but the comment is ambiguous. Consider changing the comment to

```
    /** 
     @notice 
     A microtoken is 10**6, so for a 18 decimals token the min balance 
     in microtokens is 10**18 / 10**6 = 10**12
    */ 
    int256 constant MIN_BALANCE = 10**12;
```

which is more explicit and coherent with the code.