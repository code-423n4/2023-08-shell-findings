## Gas Optimizations

|       | Issue                                                                               |
| ----- | :---------------------------------------------------------------------------------- |
| GAS-1 | Setting the constructor to payable                                                  |
| GAS-2 | DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION |
| GAS-3 | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT                   |
| GAS-4 | Use a more recent version of solidity                                               |
| GAS-5 | Splitting require() statements that use && saves gas                                |
| GAS-6 | State variables only set in the constructor should be declared immutable            |
| GAS-7 | Using storage instead of memory for structs/arrays saves gas                        |
| GAS-8 | Use != 0 instead of > 0 for unsigned integer comparison                             |
| GAS-9 | USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT     |

### [GAS-1] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

243:     constructor(

```

### [GAS-2] DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION

#### Description:

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

661:         require(result < INT_MAX);

```

### [GAS-3] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

186:     uint256 public constant BASE_FEE = 800;

```

### [GAS-4] Use a more recent version of solidity

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

4: pragma solidity =0.8.10;

```

### [GAS-5] Splitting require() statements that use && saves gas

#### Description:

Instead of using operator `&&` on a single `require`. Using a two `require` can save more gas.

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

319:        require(
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );

361:     require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

397:    require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

434:     require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

471:    require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

```

### [GAS-6] State variables only set in the constructor should be declared immutable

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

243:     constructor(

```

### [GAS-7] Using storage instead of memory for structs/arrays saves gas

#### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

65:     ) public view returns (Config memory) {

```

### [GAS-8] Use != 0 instead of > 0 for unsigned integer comparison

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

336:         require(result > 0);

378:         require(result > 0);

414:         require(result > 0);

```

### [GAS-9] USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT

#### Description:

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

#### **Proof Of Concept**

```solidity
File: src/proteus/EvolvingProteus.sol

336:         require(result > 0);

378:         require(result > 0);

414:         require(result > 0);

```
