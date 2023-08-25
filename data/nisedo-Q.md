## Summary

### Low Risk Issues

|        | Issue                                                     | Instances |
| ------ | :-------------------------------------------------------- | :-------: |
| [L001] | Contracts are not using their OZ Upgradeable counterparts |     1     |

Total: 1 instance over 1 issue

### Non-critical Issues

|        | Issue                                                                                     | Instances |
| ------ | :---------------------------------------------------------------------------------------- | :-------: |
| [NC01] | Typos in comments / Wrong comments                                                        |    11     |
| [NC02] | Top level declarations should be separated by two blank lines                             |     4     |
| [NC03] | Adding a return statement when the function defines a named return variable, is redundant |     1     |
| [NC04] | Mixed usage of `int`/`uint` with `int256`/`uint256`                                       |     2     |
| [NC05] | Useless parentheses                                                                       |     1     |

Total: 19 instances over 5 issues

## L001 - Contracts are not using their OZ Upgradeable counterparts:

The non-upgradeable standard version of OpenZeppelin's library is inherited/used by the contracts. It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behavior.

Use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts` where applicable. See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for a list of available upgradeable contracts

```solidity
File: tmp/bde7fb54-d3f4-4a74-8869-2543988da276/EvolvingProteus.sol


7       import "@openzeppelin/contracts/utils/math/Math.sol";


```


## NC01 - Typos in comments / Wrong comments:

Avoid typos, and correct wrong comments.

Wrong comment:

```solidity
    /**
       @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
       @param self config instance
    */
```

Should be: `Calculates the b variable in the curve eq which is basically a sq. root of x instantaneous price`.

Wrong comment:

```solidity
    /**
     @notice
     The minimum price value calculated with abdk library equivalent to 10^12(wei)
    */
```

Should be: `10^10` instead of `10^12` since the min price value is 10^-8.

Wrong comment:

```solidity
      @param px_final The final price at the y axis
```

Should be: `x axis` instead of `y axis`.

Wrong comment:

```solidity
295        // amount cannot be less than 0
296        require(result < 0);
```

Should be: `// amount cannot be greater than or equal to 0`.

```solidity
335        // amount cannot be less than 0
336        require(result > 0);
```

Should be: `// amount cannot be less than or equal to 0`.

```solidity
377        // amount cannot be less than 0
378        require(result > 0);
```

Should be: `// amount cannot be less than or equal to 0`.

```solidity
413        // amount cannot be less than 0
414        require(result > 0);
```

Should be: `// amount cannot be less than or equal to 0`.

Wrong comment:

```solidity
450        // amount cannot be less than 0
451        require(result < 0);
```

Should be: `// amount cannot be greater than or equal to 0`.

Wrong comment:

```solidity
459     * @dev We use FEE_UP because we want to increase the perceived amount of
460     *  reserve tokens leaving the pool and to increase the observed amount of
461     *  LP tokens being burned.
462     */
```

Should be: `We use FEE_DOWN because we want to decrease the perceived amount of reserve tokens leaving the pool and to decrease the observed amount of LP tokens being burned`.

Wrong comment:

```solidity
487        // amount cannot be less than 0
488        require(result < 0);
```

Should be: `// amount cannot be greater than or equal to 0`.

## NC02 - Top level declarations should be separated by two blank lines:

According to the Solidity Style guide, top level declarations should be separated by two blank lines.

> [Surround top level declarations in Solidity source with two blank lines.](https://docs.soliditylang.org/en/v0.8.21/style-guide.html#blank-lines)

```solidity
File: tmp/bde7fb54-d3f4-4a74-8869-2543988da276/EvolvingProteus.sol


6       import "abdk-libraries-solidity/ABDKMath64x64.sol";


10      struct Config {


44      library LibConfig {


137     contract EvolvingProteus is ILiquidityPoolImplementation {
```

## NC03 - Adding a return statement when the function defines a named return variable, is redundant:

If a function defines a named return variable, it is not necessary to explicitly return it. It will automatically be returned at the end of the function.

```solidity
File: tmp/bde7fb54-d3f4-4a74-8869-2543988da276/EvolvingProteus.sol


663             return uf;


```


## NC04 - Mixed usage of `int`/`uint` with `int256`/`uint256`:

`int256`/`uint256` are the preferred type names (they're what are used for function signatures), so they should be used consistently.

```solidity
File: tmp/bde7fb54-d3f4-4a74-8869-2543988da276/EvolvingProteus.sol


259             if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();


260             if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();


```

## NC05 - Useless parentheses:

Parentheses are redundant and should be removed from:

```solidity
782        int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
```

to match the style of:

```solidity
751        int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
```