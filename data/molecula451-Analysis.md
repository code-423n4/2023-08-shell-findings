# Architecture Recommendations

## Inconsistent Imports and Dependencies

Consider restructuring the project folders to enhance manageability. Currently, there is a mix of using libraries within the same contract file. As the project evolves, this approach may increase complexity and reduce maintainability. It's advisable to organize the libraries into a dedicated folder and then import them as needed.

Example directory structure:
```plaintext
Shell
  ├── src
  └── libraries
```

## Consider Refactoring the Struct

Consider placing the `Config` struct inside an abstract contract or library to improve code organization and reusability.

```solidity
abstract contract ConfigHandler {
    struct Config {
        int128 py_init;
        int128 px_init;
        int128 py_final;
        int128 px_final;
        uint256 t_init;
        uint256 t_final;
    }
}
```

# Compiler Warnings

To enhance the overall project quality, address the following compiler warnings, which might hide potential bugs during compilation. Although most of these warnings pertain to test files, they should be reviewed and resolved:

```plaintext
Compiler run successful with warnings:
Warning (2519): This declaration shadows an existing declaration.
   --> src/test/EvolvingProteusProperties.t.sol:102:9:
    |
102 |        (int128 py_init, int128 px_init, int128 py_final, int128 px_final) = DUT.printConfig();
    |         ^^^^^^^^^^^^^^
...
[Additional warnings]
```

# Missing Return Statements

Ensure that functions have proper return statements to enhance code quality. Below is an example of adding a return statement in the `_applyFeeByRounding` function:

```solidity
function _applyFeeByRounding(
    int256 amount,
    bool feeUp
) private pure returns (int256 roundedAmount) {
    // ... [Previous code]
    
    roundedAmount = negative
        ? -int256(roundedAbsoluteAmount)
        : int256(roundedAbsoluteAmount);
    
    return roundedAmount; // Add this return statement
}
```

# Code Duplication

Identify and refactor code duplication in both the `if (feeUp)` and `else` branches to improve readability and maintainability.

# Indentation

Refactor the project's overall indentation to adhere to standard conventions, making the codebase more consistent and easier to follow.

# CamelCase Naming Convention

Adopt the camelCase naming convention for functions and variables for consistency and readability:

```solidity
function pMin(Config storage self) public view returns (int128) {
    if (t(self) > ABDK_ONE) {
        return self.pxFinal;
    } else {
        int128 term1 = self.pxInit.mul(ABDK_ONE.sub(t(self)));
        int128 term2 = self.pxFinal.mul(t(self));
        return term1.add(term2);
    }
}
```

Additionally, use proper curly braces `{}` when writing conditional statements for clarity:

```solidity
if (t(self) > ABDK_ONE) {
    return self.pxFinal;
} else {
    int128 term1 = self.pxInit.mul(ABDK_ONE.sub(t(self)));
    int128 term2 = self.pxFinal.mul(t(self));
    return term1.add(term2);
}
```

### Time spent:
48 hours