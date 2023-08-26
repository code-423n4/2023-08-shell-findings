## Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

**Gas Optimization Recommendations**

1- The highest gas expenditure in the project occurs when creating pool instances, if wish the architecture here was design like the singleton architecture in UniswapV4, gas savings will be close to 90%.

2- The most important gas usage of the contract subject to the audit is the `ABDKMath64x64.sol` library that it uses. More gas optimized [prb-math](https://github.com/PaulRBerg/prb-math) may be preferred
detailed gas comparisons are available on the project's github link

### PRBMath

Gas estimations based on the [v2.0.1](https://github.com/PaulRBerg/prb-math/releases/tag/v2.0.1) and the
[v3.0.0](https://github.com/PaulRBerg/prb-math/releases/tag/v3.0.0) releases.

| SD59x18 | Min | Max   | Avg  |     | UD60x18 | Min  | Max   | Avg  |
| ------- | --- | ----- | ---- | --- | ------- | ---- | ----- | ---- |
| abs     | 68  | 72    | 70   |     | n/a     | n/a  | n/a   | n/a  |
| avg     | 95  | 105   | 100  |     | avg     | 57   | 57    | 57   |
| ceil    | 82  | 117   | 101  |     | ceil    | 78   | 78    | 78   |
| div     | 431 | 483   | 451  |     | div     | 205  | 205   | 205  |
| exp     | 38  | 2797  | 2263 |     | exp     | 1874 | 2742  | 2244 |
| exp2    | 63  | 2678  | 2104 |     | exp2    | 1784 | 2652  | 2156 |
| floor   | 82  | 117   | 101  |     | floor   | 43   | 43    | 43   |
| frac    | 23  | 23    | 23   |     | frac    | 23   | 23    | 23   |
| gm      | 26  | 892   | 690  |     | gm      | 26   | 893   | 691  |
| inv     | 40  | 40    | 40   |     | inv     | 40   | 40    | 40   |
| ln      | 463 | 7306  | 4724 |     | ln      | 419  | 6902  | 3814 |
| log10   | 104 | 9074  | 4337 |     | log10   | 503  | 8695  | 4571 |
| log2    | 377 | 7241  | 4243 |     | log2    | 330  | 6825  | 3426 |
| mul     | 455 | 463   | 459  |     | mul     | 219  | 275   | 247  |
| pow     | 64  | 11338 | 8518 |     | pow     | 64   | 10637 | 6635 |
| powu    | 293 | 24745 | 5681 |     | powu    | 83   | 24535 | 5471 |
| sqrt    | 140 | 839   | 716  |     | sqrt    | 114  | 846   | 710  |

### ABDKMath64x64

Gas estimations based on the v3.0 release of ABDKMath. See my [abdk-gas-estimations](https://github.com/PaulRBerg/abdk-gas-estimations) repo.

| Method | Min  | Max  | Avg  |
| ------ | ---- | ---- | ---- |
| abs    | 88   | 92   | 90   |
| avg    | 41   | 41   | 41   |
| div    | 168  | 168  | 168  |
| exp    | 77   | 3780 | 2687 |
| exp2   | 77   | 3600 | 2746 |
| gavg   | 166  | 875  | 719  |
| inv    | 157  | 157  | 157  |
| ln     | 7074 | 7164 | 7126 |
| log2   | 6972 | 7062 | 7024 |
| mul    | 111  | 111  | 111  |
| pow    | 303  | 4740 | 1792 |
| sqrt   | 129  | 809  | 699  |
