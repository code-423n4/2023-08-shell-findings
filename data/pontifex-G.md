### GAS-1 Use caching variable for `t(self)` in `_addShares` function
Use caching variables for `t(self)` storage variables in the `p_min` and  `p_max` functions to prevent multiple storage reading.
```solidity
98:        if (t(self) > ABDK_ONE) return self.px_final;
99:        else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L97-L100
```solidity
107:        if (t(self) > ABDK_ONE) return self.py_final;
108:        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L106-L109
