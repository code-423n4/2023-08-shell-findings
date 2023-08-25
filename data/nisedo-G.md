### Gas Optimizations

|       | Issue                                                                                        | Instances | Total Gas Saved |
| ----- | :------------------------------------------------------------------------------------------- | :-------- | :-------------: |
| [G01] | State variables should be cached in stack variables rather than re-reading them from storage | 14        |      1400       |
| [G02] | Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states                  | 2         |      34200      |

Total: 16 instances over 2 issues with **35600 gas** saved.

Gas totals use lower bounds of ranges and count two iterations of each for-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.

## G01 - State variables should be cached in stack variables rather than re-reading them from storage:

These are issues not catched by the Bot Race.

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

_Saves 100 gas per instance_

```solidity
File: tmp/bde7fb54-d3f4-4a74-8869-2543988da276/EvolvingProteus.sol

251             if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();


252             if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();


260             if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();


280                 inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX


320                 outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX


365                     totalSupply < INT_MAX


401                     totalSupply < INT_MAX


438                     totalSupply < INT_MAX


475                     totalSupply < INT_MAX


718             int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);


752             int256 f_2 = (f_1 * utility) / MULTIPLIER;


783             int256 f_2 = (f_1 * utility) / (MULTIPLIER);


810             if (x < MIN_BALANCE || y < MIN_BALANCE) revert BalanceError(x,y);


847                     FIXED_FEE;


```


## G02 - Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states:

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from `true` to `false` can cost up to ~20000 gas rather than `uint256(2)` to `uint256(1)` that would cost significantly less.

```solidity
File: tmp/bde7fb54-d3f4-4a74-8869-2543988da276/EvolvingProteus.sol


206         bool constant FEE_UP = true;


211         bool constant FEE_DOWN = false;


```