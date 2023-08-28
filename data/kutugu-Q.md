# Findings Summary

| ID     | Title                                                                                    | Severity |
| ------ | ---------------------------------------------------------------------------------------- | -------- |
| [L-01] | The fixed fee value of different swap directions is not equal, which affects user income | Low      |

# Detailed Findings

# [L-01] The fixed fee value of different swap directions is not equal, which affects user income

## Description

```solidity
    function _applyFeeByRounding(int256 amount, bool feeUp)
        private
        pure
        returns (int256 roundedAmount)
    {
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
```

The swap direction is different, such as `swapGivenInputAmount` / `swapGivenOutputAmount`, which requires different x or y tokens to pays part of `FIXED_FEE` as the fee, but the value of the same amount of x and y is different, such as BTC and USDT.     
If the user swaps BTC for USDT and adopts `swapGivenOutputAmount` method, additional BTC will be paid as fee, and more value will be paid compared with `swapGivenInputAmount`.     

## Recommendations

FIXED_FEE should use the corresponding USD value, not the number of tokens.