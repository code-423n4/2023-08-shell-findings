The current implementation uses multiple `if` statements followed by `revert` conditions to validate certain constraints. By using the `require` statement instead, we can simplify the code and potentially save more gas costs during contract execution.

in code = https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L251C7-L264C1

        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();


        // at all times x price should be less than y price
        if (py_init <= px_init) revert InvalidPrice();
        if (py_final <= px_final) revert InvalidPrice();

      }

## Recommendation

use require instead if/revert


