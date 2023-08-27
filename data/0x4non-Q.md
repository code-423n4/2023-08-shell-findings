## NC1 Unnecessary cast

On [EvolvingProteus.sol#L49](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L49) there is an unnecessary cast, you can simple use `1 << 64` 

```solidity
    int128 constant ABDK_ONE = 1 << 64;
```

## NC2 Be specific with the type of the variable, use `uint256` instead of `uint`
`uint` is an alias for `uint256` but it is better to be specific with the type of the variable. Use `uint256` instead of `uint` on;
[EvolvingProteus.sol#L259](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L259)
[EvolvingProteus.sol#L260](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L260)

## NC3 Preffer scientific notation
On:
[EvolvingProteus.sol#L181](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L181)
`uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;`
[EvolvingProteus.sol#L191](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L191)
`uint256 constant FIXED_FEE = 10**9;`
[EvolvingProteus.sol#L201](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L201)
`int256 constant MAX_PRICE_RATIO = 10**4;`

Preferring scientific notation is more readable and less error prone. Use `10e11` instead of `10**11`, `10e9` instead of `10**9` and `10e4` instead of `10**4`

## NC4 Missing bracket could difficult readability
On [EvolvingProteus.sol#L843](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L843) it might be better to add a bracket to make it more readable.

```diff
diff --git a/src/proteus/EvolvingProteus.sol b/src/proteus/EvolvingProteus.sol
index 85341bb..ddbe53f 100644
--- a/src/proteus/EvolvingProteus.sol
+++ b/src/proteus/EvolvingProteus.sol
@@ -840,11 +840,12 @@ contract EvolvingProteus is ILiquidityPoolImplementation {
                 (absoluteValue / BASE_FEE) +
                 FIXED_FEE;
             require(roundedAbsoluteAmount < INT_MAX);
-        } else
+        } else {
             roundedAbsoluteAmount =
                 absoluteValue -
                 (absoluteValue / BASE_FEE) -
                 FIXED_FEE;
+        }
 
         roundedAmount = negative
             ? -int256(roundedAbsoluteAmount)
```
