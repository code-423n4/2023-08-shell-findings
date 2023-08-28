## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Constant decimal values | 1 |
| [L-2](#L-2) | Loss of precision on division | 3 |

### [L-1] Constant decimal values
The use of fixed decimal values such as 1e18 or 1e8 in Solidity contracts can lead to inaccuracies, bugs, and vulnerabilities, particularly when interacting with tokens having different decimal configurations. Not all ERC20 tokens follow the standard 18 decimal places, and assumptions about decimal places can lead to miscalculations.Resolution: Always retrieve and use the decimals() function from the token contract itself when performing calculations involving token amounts. This ensures that your contract correctly handles tokens with any number of decimal places, mitigating the risk of numerical errors or under/overflows that could jeopardize contract integrity and user funds.

*Instances (1)*:
```solidity
File: example/1.sol

196:     int256 constant MULTIPLIER = 1e18;

```

### [L-2] Loss of precision on division
Solidity doesnt support fractions, so divisions by large numbers could result in the quotient being zero.To avoid this, its recommended to require a minimum numerator amount to ensure that it is always greater than the denominator.

*Instances (5)*:
```solidity
File: example/1.sol

717:         int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

718:         int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

783:         int256 f_2 = (f_1 * utility) / (MULTIPLIER); 

```

## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Constant redefined elsewhere | 14 |
| [NC-2](#NC-2) | Expressions for constant values should use immutable rather than constant | 14 |
| [NC-3](#NC-3) | Import declarations should import specific identifiers, rather than the whole file | 2 |
| [NC-5](#NC-4) | Lines are too long | 12 |



### [NC-1] Constant redefined elsewhere
Consider defining in only one contract so that values cannot become out of sync when only one location is updated. A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contracts value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers dont get out of sync.

*Instances (14)*:
```solidity
File: example/1.sol

49:     int128 constant ABDK_ONE = int128(int256(1 << 64));

146:     uint256 constant INT_MAX = uint256(type(int256).max);

151:     int256 constant MIN_BALANCE = 10**12;

157:     int128 constant MAX_M = 0x5f5e1000000000000000000;

163:     int128 constant MIN_M = 0x00000000000002af31dc461;

169:     int256 constant MAX_PRICE_VALUE = 1844674407370955161600000000;

175:     int256 constant MIN_PRICE_VALUE = 184467440737;

181:     uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;

186:     uint256 public constant BASE_FEE = 800;

191:     uint256 constant FIXED_FEE = 10**9;

196:     int256 constant MULTIPLIER = 1e18;

201:     int256 constant MAX_PRICE_RATIO = 10**4; // to be comparable with the prices calculated through abdk math

206:     bool constant FEE_UP = true;

211:     bool constant FEE_DOWN = false;

```

### [NC-2] Expressions for constant values should use immutable rather than constant
While it does not save gas for some simple binary expressions because the compiler knows that developers often make this mistake, its still best to use the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

*Instances (14)*:
```solidity
File: example/1.sol

49:     int128 constant ABDK_ONE = int128(int256(1 << 64));

146:     uint256 constant INT_MAX = uint256(type(int256).max);

151:     int256 constant MIN_BALANCE = 10**12;

157:     int128 constant MAX_M = 0x5f5e1000000000000000000;

163:     int128 constant MIN_M = 0x00000000000002af31dc461;

169:     int256 constant MAX_PRICE_VALUE = 1844674407370955161600000000;

175:     int256 constant MIN_PRICE_VALUE = 184467440737;

181:     uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;

186:     uint256 public constant BASE_FEE = 800;

191:     uint256 constant FIXED_FEE = 10**9;

196:     int256 constant MULTIPLIER = 1e18;

201:     int256 constant MAX_PRICE_RATIO = 10**4; // to be comparable with the prices calculated through abdk math

206:     bool constant FEE_UP = true;

211:     bool constant FEE_DOWN = false;

```

### [NC-3] Import declarations should import specific identifiers, rather than the whole file
Using import declarations of the form import {<identifier_name>} from "some/file.sol" avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation (but does not save any gas)

*Instances (2)*:
```solidity
File: example/1.sol

6: import "abdk-libraries-solidity/ABDKMath64x64.sol";

7: import "@openzeppelin/contracts/utils/math/Math.sol";

```

### [NC-4] Lines are too long
Maximum suggested line length is 120 characters according to the documentation.

*Instances (12)*:
```solidity
File: example/1.sol

25:      time: the total duration of the curve's evolution (e.g. the amount of time it should take to evolve from the initial to the final prices)

34:      calculation. config.a() and config.b() are then called whenever a and b are needed, and return the correct value for

35:      a or b and the time t. When the total duration is reached, t remains = 1 and the curve will remain in its final shape. 

37:      Note: To mitigate rounding errors, which if too large could result in liquidity provider losses, we enforce certain constraints on the algorithm.

38:            Min transaction amount: A transaction amount cannot be too small relative to the size of the reserves in the pool. A transaction amount either as an input into the pool or an output from the pool will result in a transaction failure

39:            Max transaction amount: a transaction amount cannot be too large relative to the size of the reserves in the pool. 

40:            Min reserve ratio: The ratio between the two reserves cannot fall below a certain ratio. Any transaction that would result in the pool going above or below this ratio will fail.

41:            Max reserve ratio: the ratio between the two reserves cannot go above a certain ratio. Any transaction that results in the reserves going beyond this ratio will fall.

112:        @notice Calculates the a variable in the curve eq which is basically a sq. root of the inverse of y instantaneous price

120:        @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price

259:         if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

260:         if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

```

