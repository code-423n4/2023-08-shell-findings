| Number | Optimization Details                                               | Instances |
| :----: | :----------------------------------------------------------------- | :-------: |
| [G-01] | Function calls should be cached instead of re-calling the function |     2     |
| [G-02] | Using ternary operator instead of if else saves gas                |     2     |

Total 2 issues

## [G-01] Function calls should be cached instead of re-calling the function

The following issues are missed in the bot report

_Total 2 instances - 1 file:_

### Instance#1:

```solidity
File: src/proteus/EvolvingProteus.sol
// @audit ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1) is duplicated on line 259
260:               if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L260

### Instance#2:

```solidity
File: src/proteus/EvolvingProteus.sol
// @audit  _applyFeeByRounding(specifiedAmount, feeDirection) is duplicated on line 578
581: yf = yi + _applyFeeByRounding(specifiedAmount, feeDirection);

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L581C16-L581C78

## [G-02] Using ternary operator instead of if else saves gas

_Total 2 instances - 1 file:_

### Instance#1:

```solidity
File: src/proteus/EvolvingProteus.sol
98:        if (t(self) > ABDK_ONE) return self.px_final;
99:        else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L98C1-L99C93

### Instance#2:

```solidity
File: src/proteus/EvolvingProteus.sol
107: if (t(self) > ABDK_ONE) return self.py_final;
108:        else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L107C8-L108C93
