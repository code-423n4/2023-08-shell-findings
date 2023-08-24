## Non Critical Issues

|      | Issue                                                                      |
| ---- | :------------------------------------------------------------------------- |
| NC-1 | ADD TO BLACKLIST FUNCTION                                                  |
| NC-2 | Be explicit declaring types                                                |
| NC-3 | GENERATE PERFECT CODE HEADERS EVERY TIME                                   |
| NC-4 | FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE                          |
| NC-5 | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES                    |
| NC-6 | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION              |
| NC-7 | USE A MORE RECENT VERSION OF SOLIDITY                                      |
| NC-8 | `require()` / `revert()` statements should have descriptive reason strings |
| NC-9 | USE UNDERSCORES FOR NUMBER LITERALS                                        |

### [NC-1] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

4: pragma solidity =0.8.10;

```

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-2] Be explicit declaring types

#### Description:

Instead of uint use uint256

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

259:         if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

260:         if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

#### Description:

Use camel case for all functions, parameters and variables and snake case for constants.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

11:     int128 py_init;

12:     int128 px_init;

13:     int128 py_final;

14:     int128 px_final;

15:     uint256 t_init;

16:     uint256 t_final;

97:     function p_min(Config storage self) public view returns (int128) {

106:     function p_max(Config storage self) public view returns (int128) {

746:         int256 a_convert = a.muli(MULTIPLIER);

747:         int256 b_convert = b.muli(MULTIPLIER);

750:         int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);

751:         int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);

752:         int256 f_2 = (f_1 * utility) / MULTIPLIER;

777:         int256 a_convert = a.muli(MULTIPLIER);

778:         int256 b_convert = b.muli(MULTIPLIER);

781:         int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;

782:         int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );

783:         int256 f_2 = (f_1 * utility) / (MULTIPLIER);

```

### [NC-5] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

6: import "abdk-libraries-solidity/ABDKMath64x64.sol";

7: import "@openzeppelin/contracts/utils/math/Math.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-6] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

4: pragma solidity =0.8.10;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-7] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

4: pragma solidity =0.8.10;

```

### [NC-8] `require()` / `revert()` statements should have descriptive reason strings

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

296:         require(result < 0);

336:         require(result > 0);

378:         require(result > 0);

414:         require(result > 0);

451:         require(result < 0);

488:         require(result < 0);

590:         require(result < INT_MAX);

658:         require(sf >= MIN_BALANCE);

661:         require(result < INT_MAX);

842:             require(roundedAbsoluteAmount < INT_MAX);

```

### [NC-9] USE UNDERSCORES FOR NUMBER LITERALS

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

175:     int256 constant MIN_PRICE_VALUE = 184467440737;

```

## Low Issues

|     | Issue                                                          |
| --- | :------------------------------------------------------------- |
| L-1 | LACK OF INPUT VALIDATION                                       |
| L-2 | LOSS OF PRECISION DUE TO ROUNDING                              |
| L-3 | UNIFY RETURN CRITERIA                                          |
| L-4 | REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR                     |
| L-5 | Timestamp Dependence                                           |
| L-6 | Void constructor                                               |
| L-7 | NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES |

### [L-1] LACK OF INPUT VALIDATION

#### Description:

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

796:     function _checkAmountWithBalance(uint256 balance, uint256 amount)

```

### [L-2] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

717:         int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

718:         int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

750:         int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);

751:         int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);

752:         int256 f_2 = (f_1 * utility) / MULTIPLIER;

781:         int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;

782:         int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );

783:         int256 f_2 = (f_1 * utility) / (MULTIPLIER);

```

### [L-3] UNIFY RETURN CRITERIA

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

52:        @notice Calculates the equation parameters "a" & "b" described above & returns the config instance

65:     ) public view returns (Config memory) {

81:     function elapsed(Config storage self) public view returns (uint256) {

89:     function t(Config storage self) public view returns (int128) {

97:     function p_min(Config storage self) public view returns (int128) {

106:     function p_max(Config storage self) public view returns (int128) {

115:     function a(Config storage self) public view returns (int128) {

123:     function b(Config storage self) public view returns (int128) {

132:     function duration(Config storage self) public view returns (uint256) {

277:     ) external view returns (uint256 outputAmount) {

317:     ) external view returns (uint256 inputAmount) {

359:     ) external view returns (uint256 mintedAmount) {

395:     ) external view returns (uint256 depositedAmount) {

432:     ) external view returns (uint256 burnedAmount) {

469:     ) external view returns (uint256 withdrawnAmount) {

512:     ) internal view returns (int256 computedAmount) {

570:     ) internal view returns (int256 computedAmount) {

614:     ) internal view returns (int256 computedAmount) {

657:     ) internal view returns (int256 uf) {

686:             returns (int256, int256) getPoint

687:     ) internal view returns (int256 xf, int256 yf) {

704:     ) internal view returns (int256 utility) {

742:     ) internal view returns (int256 x0, int256 y0) {

773:     ) internal view returns (int256 x0, int256 y0) {

827:         returns (int256 roundedAmount)

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-4] REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR

#### Description:

The correct and clear error description explains to the user why the function reverts, but the error descriptions below in the project are not self-explanatory. These error descriptions are very important in the debug features of DApps like Tenderly. Error definitions should be added to the require block, not exceeding 32 bytes.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol
296:         require(result < 0);

336:         require(result > 0);

378:         require(result > 0);

414:         require(result > 0);

451:         require(result < 0);

488:         require(result < 0);

590:         require(result < INT_MAX);

658:         require(sf >= MIN_BALANCE);

661:         require(result < INT_MAX);

842:             require(roundedAbsoluteAmount < INT_MAX);

```

#### Recommended Mitigation Steps:

Error definitions should be added to the require block, not exceeding 32 bytes or we should use custom errors

### [L-5] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

72:             block.timestamp,

73:             block.timestamp + _duration

82:         return block.timestamp - self.t_init;

```

### [L-6] Void constructor

#### Description:

Calls to base contract constructors that are unimplemented leads to misplaced assumptions.

https://github.com/crytic/slither/wiki/Detector-Documentation#void-constructor

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

243:     constructor(

```

### [L-7] NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES

#### Description:

The package.json configuration file says that the project is using 4.7.3 of OpenZeppelin which has a not last update version

#### **Proof Of Concept**

```solidity
File: package.json

12:    "@openzeppelin/contracts": "^4.7.3",

```

#### Recommended Mitigation Steps:

Use patched versions.
