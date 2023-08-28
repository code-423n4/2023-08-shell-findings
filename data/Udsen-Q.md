## 1. RECOMMENDED TO ADD AN INPUT VALIDATION FOR THE `duration` VARIABLE IN THE `constructor`

In the `EvolvingProteus.constructor` the `duration` is passed in as an input parameter. `duration` denotes the total duration of the curve's evolution. Hence the `duration` variable is a critical variable of the protocol. If duration is set to zero by mistake it will prompt a redeployment of the contract which could be wastage of gas and additional work load. Hence an input validity check should be performed for the `duration` variable inside the `constructor` as follows:

    require(duration != 0, `Duration can not be zero`);

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L243-L263

## 2. DISCREPENCY BETWEEN NATSPEC COMMENTS AND LOGIC IMPLEMENTATION

The `EvolvingProteus._getUtility` function is used to calculate the `utility` value of the pool at the time of the function call. The `utilitiy` is calculated using a quadratic formula which is shown below:

    k(ab - 1)u**2 + (ay + bx)u + xy/k = 0

As per the equation there is a `k` value used in the `a` adn `c` coefficients. But this `k` value is not used when calculating the `aQuad` and the `cQuad` values. This could be due to `normalization` of the equation or the `k = 1`. But the reason for the missing `k` value from the coefficient calculations is not given.

Hence it is fair to assume there is a discrepency between the Natspec comments and the actual implementation of the logic.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L697
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L712-L714

## 3. DISCREPENCY BETWEEN VALUES `uint256` CAN HOLD IN `EvolvingProteus` CONTRACT DUE TO `INT_MAX` CAP

In the `EvolvingProteus` contract the input parameter amounts to the transaction functions such as swap, deposit and withdraw are capped by `INT_MAX` value. The input parameters are required to be less than this amount as shown below:

        require(
            burnedAmount < INT_MAX &&
                xBalance < INT_MAX &&
                yBalance < INT_MAX &&
                totalSupply < INT_MAX
        );

Hence the any of the passed in input parameters can not be equal to the `INT_MAX` value.

But when deriving an output amount using helper functions of this contract, the resultant amounts upcasted from `int256` to `uint256` value. When it is converted into a `uint256` value those parameters are allowed to have a `value = INT_MAX`. This is because upcasting does not lead to data truncation. But it does not treat all the `uint256` variables in the function scope equally since the uint256 values calculated inside the function scope can have the `type(int256).max` value where as the passed in uint256 input parameters can not be equal to the `type(int256).max` value.

Hence there is a discrepency between the amounts that an uint256 can hold inside this smart contract. This discrepency in `uint256` allowed values can lead to precision loss and other unexpected behaviours of the contract.

Hence it is recommended to either allow the passed in `uint256` input parameters to the functions, to have a value equal to `type(int256).max`, or prohibit internal `uint256` variables of the `EvolvingProteus` contract from having values equal to `type(int256).max` value.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L434-L439
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L589

## 4. DISCREPENCY BETWEEN NATSPEC COMMENT AND LOGIC IMPLEMENTATION OF THE `withdrawGivenInputAmount` FUNCTION

The `natspec` comments of the `EvolvingProteus.withdrawGivenInputAmount` function states that the `feeDirection` of the function to be used is `FEE_UP`. The reason for using `FEE_UP` is described as wanting to increase the perceived amount of reserve tokens leaving the pool. But the `feeDirection` is determined by the protocol to benefit itself. Hence `withdrawGivenInputAmount` transaction will benefit the protocol if perceived amount of reserve tokens leaving the pool is decreased which is equivalent to charging a fee from the withdrawing amount.

The logic implemenation of the `withdrawGivenInputAmount` function has implemented this requirement correctly as shown below:

        int256 result = _lpTokenSpecified( //@audit-info - amount of reserved token to withdraw
            withdrawnToken,
            -int256(burnedAmount),
            FEE_DOWN, //@audit-issue - discrepency between the NATSPEC and the implementation regarding FEE_DOWN
            int256(totalSupply),
            int256(xBalance),
            int256(yBalance)
        );

Hence it is recommended to update the `natspect` comments of the `withdrawGivenInputAmount` function to state that the `feeDirection` used for the `withdrawGivenInputAmount` transaction is `FEE_DOWN` by providing it with the proper reason to do so as well.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L459-L461
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L478-L485

## 5. ERRORNEOUS NATSPEC COMMENTS SHOULD BE CORRECTED

    @param px_final The final price at the `y axis`

Above comment should be corrected as follows:

    @param px_final The final price at the `x axis`

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L240

    The minimum price value calculated with abdk library equivalent to `10^12(wei)`

The above comment should be corrected as follows since the given value (10^12) in the comment is not the minimum price value

    The minimum price value calculated with abdk library equivalent to `10^10(wei)`

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L173

     *  without caring about which particular function `is was` called.

The above comment should be corrected as follows:

     *  without caring about which particular function `was` called.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L736     
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L768

## 6. LACK OF EVENTS IN THE CRITICAL OPERATIONS OF THE PROTOCOL

The `EvolvingProteus` smart contract performs critical operations such as `swaps`, `deposits`, `withdrawals`. But these operations lack `events`. Hence it is recommended to emit events when above operations are succefully executed for off-chain analysis.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L506-L554
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L463-L490

## 7. ADD MEANINGFUL `revert` MESSAGES TO THE `require` STATEMENTS

In the `EvolvingProteus` contract, there are multiple occassions `require` statement are used for conditional checks. But these `require` statements are not using `revert messages`. Hence it these `require` statements revert it will be difficult to find the `reason for the revert`.

Hence it is recommneded to add meaningful revert messages to these `require` statements.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L414
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L397-L402