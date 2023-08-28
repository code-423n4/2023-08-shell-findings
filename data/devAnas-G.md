[G] Optimizing Fee Calculation Function with Zero Amount Handling for Reduced Gas Costs

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L824-L852

`  

   



     // adding a custom error
     error ZeroAmount();

    function _applyFeeByRounding(int256 amount, bool feeUp)
        private
        pure
        returns (int256 roundedAmount)
    {
        // adding a check that amount is not 0
        if(amount == 0) revert ZeroAmount();
        bool negative = amount < 0 ? true : false;
        uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);
        // FIXED_FEE * 2 because we will possibly deduct the FIXED_FEE from
        // this amount, and we don't want the final amount to be less than
        // the FIXED_FEE.
        if (absoluteValue < FIXED_FEE * 2) revert AmountError();

        uint256 roundedAbsoluteAmount;
        if (feeUp) {
            roundedAbsoluteAmount =
                absoluteValue +
                (absoluteValue / BASE_FEE) +
                FIXED_FEE;
            require(roundedAbsoluteAmount < INT_MAX);
        } else
            roundedAbsoluteAmount =
                absoluteValue -
                (absoluteValue / BASE_FEE) -
                FIXED_FEE;

        roundedAmount = negative
            ? -int256(roundedAbsoluteAmount)
            : int256(roundedAbsoluteAmount);
    }
`

Adding the check for amount == 0 in the _applyFeeByRounding function saves gas by preventing unnecessary calculations and conditions when the input amount is zero. By catching this condition early, the function can immediately exit and revert without performing any further operations. This optimization reduces gas consumption because it avoids executing unnecessary calculations, conditions, and requires for cases where the input amount is zero, resulting in quicker and more efficient execution of the function.