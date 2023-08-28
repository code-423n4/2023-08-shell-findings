## [N-01] Constants in comparisons should appear on the left side

Doing so will prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

There are 6 instances of this issue in 1 file:

    File: src/proteus/EvolvingProteus.sol	

    296: require(result < 0);

    336: require(result > 0);

    378: require(result > 0);

    414: require(result > 0);

    451: require(result < 0);

    488: require(result < 0);

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol

## [N-02] Use scientific notation (e.g. *1e18*) rather than exponentiation (e.g. *10**18*)

There are 4 instances of this issue in 1 file:

    File: src/proteus/EvolvingProteus.sol	

    151: int256 constant MIN_BALANCE = 10**12;

    181: uint256 constant MAX_BALANCE_AMOUNT_RATIO = 10**11;

    191: uint256 constant FIXED_FEE = 10**9;

    201: int256 constant MAX_PRICE_RATIO = 10**4; // to be comparable with the prices calculated through abdk math

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol

## [N-03] Large multiples of ten should use scientific notation

Use (e.g. 1e6) rather than decimal literals (e.g. 1000000), for better code readability

There are 2 instances of this issue in 1 file:

    File: src/proteus/EvolvingProteus.sol	

    157: int128 constant MAX_M = 0x5f5e1000000000000000000;

    169: int256 constant MAX_PRICE_VALUE = 1844674407370955161600000000;

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol

