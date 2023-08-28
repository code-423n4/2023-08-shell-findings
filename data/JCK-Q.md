


# LOW RISK

| Number |  Issue  | Instances | 
|--------|---------|-----------|
|[LOW-01]| Upgradeable contract uses non-upgradeable version of the OpenZeppelin libraries/contracts| 1 | 
|[LOW-01]| Require statements should have error string | 16 | 
|[LOW-01]| Calculation will revert when utility and amount is  zero | 3 | 


## [LOW‑01] Upgradeable contract uses non-upgradeable version of the OpenZeppelin libraries/contracts

OpenZeppelin has an Upgradeable variants of each of its libraries and contracts, and upgradeable contracts should use those variants.

There is one instance of this issue:

```solidity
file:  src/proteus/EvolvingProteus.sol

7   import {Math} from  "@openzeppelin/contracts/utils/math/Math.sol";

```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L7


## [LOW-02] Require statements should have error string

Adding error strings to require statements in Solidity contracts, although not mandatory, enhances error handling, debugging, and overall contract maintainability. Providing a descriptive error message with each require statement helps identify the specific reason for a transaction failure, making it easier for developers to troubleshoot issues and for users to understand the cause of a revert. Including error strings improves code readability and fosters transparency, as the logic and conditions behind each requirement are clearly communicated

```solidity
file:  src/proteus/EvolvingProteus.sol

179   require(
            inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );


196   require(result < 0);

319    require(
            outputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
        );

336    require(result > 0);

361    require(
            depositedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

378    require(result > 0);

397   require(
            mintedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

414     require(result > 0);

434    require(
            withdrawnAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

451    require(result < 0);

471   require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

488   require(result < 0);

590   require(result < INT_MAX);  

658   require(sf >= MIN_BALANCE);

661   require(result < INT_MAX);

842   require(roundedAbsoluteAmount < INT_MAX);

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L179-L180

## [LOW-03] Calculation will revert when utility and amount is  zero

In the instance where the variable utility and amount  is  zero, it will inevitably lead to a division by zero error when used in mathematical operations, causing the transaction to fail and potentially disrupting contract variableality. This situation can inadvertently serve as a blocking mechanism, preventing valid transactions and operations. It's crucial to accommodate this special scenario in your code. One resolution could be implementing condition checks in your variable to detect a zero utility and amount  and handle it differently, perhaps by returning a specific value or altering the operational flow, thus ensuring that transactions do not revert and the contract variables smoothly even in this edge case.

```solidity
file:   src/proteus/EvolvingProteus.sol

750    int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);

781    int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;

800    if (balance / amount >= MAX_BALANCE_AMOUNT_RATIO) revert AmountError();

```
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L1781