# [A-01] Any comments for the judge to contextualize your findings

My primary objective is to enhance the gas efficiency and cost-effectiveness of the protocol. Throughout my analysis, I have diligently pursued opportunities to optimize gas and have prepared a comprehensive report comprising `10` detailed gas optimization recommendations tailored to the specific needs of the protocol.

The optimizations I have identified aim to improve gas efficiency, reduce storage reads, and overall make the smart contracts more cost-effective to deploy and interact with on the blockchain

1. **Gas Optimization**

   - **State Variable Caching (G-01)**: Recommends using stack variables to cache state variables rather than repeatedly reading them from storage, which can reduce gas costs.

   - **Unused Stack Variable (G-02)**: Suggests using a stack variable instead of a state variable if it's only used once, as this can optimize gas usage by reducing storage operations.

   - **Boolean Representation (G-03)**: Advises using uint256(1) and uint256(2) instead of boolean values for gas optimization, as state changes of boolean values can be more expensive in terms of gas.

   - **OpenZeppelin Upgrade (G-04)**: Recommends updating to the v4.9.0 version of OpenZeppelin contracts for various gas optimizations provided in that release.

   - **Assembly for Math Operations (G-05)**: Proposes using assembly for arithmetic operations like addition, subtraction, multiplication, and division to achieve better gas efficiency.

   - **Comparison Optimization (G-06)**: Suggests using != 0 instead of > 0 for unsigned integer comparisons to optimize gas usage.

   - **Refactoring Duplicated Checks (G-07)**: Recommends refactoring duplicated if() checks into modifiers or functions to avoid redundancy and improve code maintainability.

And some low severity this is the overview of my findings.

- **Incorrect Utility Calculation in `_getUtility` Function**
  In the `_getUtility` function, there is an issue with calculating the results of the quadratic formula. The problem arises from not correctly handling the signs of coefficients a and b. Currently, both signs are checked together, which can lead to inaccurate utility calculations when a and b have opposite signs.

- **Negative Calculation Result Stored in uint256 Integer Leading to Incorrect Outcome**
  In the `_applyFeeByRounding` function, there's a problem where a negative calculation result is stored in the roundedAbsoluteAmount variable, which is of type uint256. Storing a negative value in an unsigned integer variable can lead to incorrect outcomes due to truncation, potentially causing unintended behavior.

# [A-02] Approach taken in evaluating the codebase

Accordingly, I analyzed and audited the subject in the following steps;

1. **Core Protocol EvolvingProteus Contract Overview**:

   - **Constants and Parameters**:

     The contract defines various constants like `INT_MAX`, `MIN_BALANCE`, `MAX_M`, etc., used for calculations and validations.
     The constructor takes initial parameters related to the pool's configuration, including initial and final prices for both x and y reserves, and a duration for the curve to evolve.

   - **Functions**:

     `_swapGivenInputAmount`: Calculates the output amount of the other reserve token when a user swaps a specified input amount.

     `_swapGivenOutputAmount`: Calculates the input amount of the other reserve token when a user swaps a specified output amount.

     `_depositGivenInputAmount`: Calculates the amount of LP tokens minted when a user deposits a specified input amount.

     `_depositGivenOutputAmount`: Calculates the amount of reserve tokens needed for a user to deposit a specified output amount of LP tokens.

     `_withdrawGivenOutputAmount`: Calculates the amount of LP tokens burned when a user withdraws a specified output amount of reserve tokens.

     `_withdrawGivenInputAmount`: Calculates the amount of reserve tokens received when a user withdraws a specified input amount of LP tokens.

     `_getUtility`: Calculates the utility of the pool based on the balance of reserve tokens.

     `_getPointGivenXandUtility` and `_getPointGivenYandUtility`: Calculate coordinate points on the bonding curve based on utility and a known coordinate.

     `_applyFeeByRounding`: Implements a fee mechanism using rounding, which applies fees to actions.

     `_checkAmountWithBalance`: Validates that the ratio of an amount to a balance is within acceptable limits.

     `_checkBalances`: Validates that the pool's balances and ratios stay within specified limits.

   - **Utility Calculations**:

     `_getUtility`: Calculates the utility of the pool based on the balance of the reserve tokens. The utility equation is based on a quadratic formula derived from the bonding curve equation.

     `_getPointGivenXandUtility` and `_getPointGivenYandUtility`: Calculate the other coordinate of a point on the bonding curve based on utility and a known coordinate.

   - **Fee Mechanism**:

     `_applyFeeByRounding`: Implements a fee mechanism using rounding to adjust amounts based on fees.

   - **Balances and Ratios**:

     `_checkAmountWithBalance` and `_checkBalances`: Validate that the pool's balances and ratios meet acceptable criteria.

   - **Custom Errors**:

     Custom error types are defined to provide informative error messages for different types of failures or violations.

   - **Constructor**:

     The constructor initializes the pool's configuration and performs various parameter validations to ensure the pool's integrity.

2. **Documentation Review**:

   I don't go deep into the documentation; I only review what is in the readme file.

   The documentation introduces Evolving Proteus as a liquidity concentration updating primitive, likening it to a combination of Balancer liquidity bootstrapping pools and Uniswap v3. It highlights key parameters governing the evolution of liquidity concentration.

   - **Use Cases**:
     Two main use cases are described: Dutch auctions and dynamic pools. Dutch auctions utilize changing liquidity concentration for price discovery, while dynamic pools adapt based on external factors to minimize losses.

   - **Key Terminology**:
     The document clarifies terms such as numeraire token and abstract references to reserve tokens. It also covers the concept of utility as the internal value measure of the pool.

   - **Algorithm Explanation**:
     The algorithm is detailed, with six parameters determining liquidity concentration. It explains how current high and low prices are calculated for each block and presents the bonding curve equation. The conditions for the algorithm to stop evolving are specified.

   - **Type of Errors**:
     A list of errors due to constraint violations is provided. These include boundary, amount, curve, balance, and price-related errors.

   - **Price Ranges**:
     Constraints regarding price ranges and liquidity concentration are outlined.

   - **Invariants**:
     Critical invariants are listed, including utility per LP token and relationships between swap amounts and balances. There are also "soft" invariants.

   - **Architecture**:
     User interactions with the contract occur through the Ocean accounting layer. Six methods used via Ocean are explained for calculations.

3. **Graphical Analysis**: Solidity-metrics is a popular tool used to analyze and visualize Solidity code. It provides various metrics and visualizations to gain insights into the codebase's complexity and maintainability.
   [solidity-metrics](https://github.com/ConsenSys/solidity-metrics)

4. **Utilizing Static Analysis Tools**: During the static analysis phase, I initiated the Slither tool. The initial run of Slither on the EvolvingProteus contract led to the identification of some low vulnerabilities. Notably,some of these vulnerabilities align with known issues categorized under `BotRace`. The result of the Slither analysis indicated the presence of both false positives and negatives.

5. **Test Coverage Evaluation**: During this phase, I play with the tests, initially encountering challenges when attempting to run them first. However, after resolving the issues, I found the well-written tests to be quite interesting.

6. **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode, Despite the complexity of this codebase, with its heavy mathematical elements, I didn't come across any noteworthy insights during the first two modes. Consequently, I shifted my focus to the Blackboxing mode.

   - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

   - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

   - **Blackboxing Mode**: Particularly useful for complex codebases. Grasp the protocol's expected behavior through tests, and experiment with them to better understand and navigate the intricacies of the code.

# [A-03] Architecture recommendations

The following architecture improvements and feedback could be considered:

1. The Proteus AMM Engine introduces a composite bonding curve model that holds the potential for precise liquidity concentration and fungible LP tokens. To further enhance the architecture, consider implementing the following recommendations

   - **Automated Curve Parameterization**: Implement a mechanism that automatically derives the curve parameters based on predefined utility functions or user-specified liquidity distribution preferences. This simplifies the process for users and reduces the risk of human error when setting up new curve segments.

   - **Automated Arbitrage Mechanism**: Build an automated arbitrage mechanism that monitors price imbalances between the Proteus AMM and external markets. Traders can take advantage of these arbitrage opportunities, contributing to price stability.

2. **algorithm**

   - **Gradual Liquidity Transition**: Implement a mechanism for gradual liquidity transition between the initial and final states. Instead of instantly jumping to the final liquidity concentration, introduce intermediate steps or time intervals during which the concentration gradually shifts. This can help liquidity providers adjust and adapt more smoothly to changing market conditions.

   - **Liquidity Participation Factors**: Introduce liquidity participation factors that allow liquidity providers to specify the rate at which they want their liquidity to transition between initial and final concentrations. This can give liquidity providers more control over the speed of concentration adjustments.

3. **Mathematical Model Complexity**: The mathematical model used in the contract is quite intricate, and understanding its behavior might be challenging for other auditors. Consider adding more detailed comments explaining the equations, their significance, and any external mathematical concepts involved.

4. **Usage Examples**: Provide usage examples for the main functionalities of the contract. This will help other auditors understand how to interact with your contract and test its functionality.

5. **Consider Enumerations**: In some parts of the code, you're using boolean flags (FEE_UP and FEE_DOWN) to indicate fee direction. Consider using an enumeration with more descriptive names to improve code readability and reduce the risk of mixing up the flags.

# [A-04] Codebase analysis

- **Import Statements**:

  The contract imports various libraries and interfaces. Notable imports include "abdk-libraries-solidity/ABDKMath64x64.sol" for fixed-point math operations and "@openzeppelin/contracts/utils/math/Math.sol" for general math utilities.
  Config Struct:

  The Config struct defines the parameters needed to describe the curve's evolution. It includes initial and final prices along the x and y axes, as well as initial and final times.

- **LibConfig Library**:

  The LibConfig library provides utility functions for working with the Config struct. These functions include calculations for various properties of the curve, such as elapsed time, ratios, and utility values.

- **EvolvingProteus Contract**:

  This contract implements the ILiquidityPoolImplementation interface and defines the main functionality of the liquidity pool.
  Constants and Variables:

  `Use Descriptive Variable Names`: Use descriptive variable names that explain the purpose of the variable. This will make the code self-explanatory and easier to read. Variable names like xf, yf, ui, uf are not very clear on their own.

  `Custom Errors`:

  The contract defines custom errors, such as AmountError, BalanceError, BoundaryError, CurveError, InvalidPrice, and others. These errors are used to provide more descriptive error messages when certain conditions are not met.
  Constructor:

  The constructor initializes the config struct with the provided initial and final prices, as well as the duration of the curve's evolution. It also performs checks to ensure that the provided price values and ratios are within acceptable limits.

  `Swap, Deposit, and Withdraw Functions`:

  The contract defines functions for performing swaps, deposits, and withdrawals. Each of these functions takes input parameters, performs calculations based on the provided amounts and balances, and enforces constraints to prevent invalid transactions.

  `Utility Functions`:

  The contract includes several utility functions for performing calculations related to the curve's evolution, utility values, price points, and ratios. These functions are used internally to determine swap and balance-related calculations.

  `Check Functions`:

  The contract includes functions like `_checkAmountWithBalance` and `_checkBalances` to enforce various constraints on amounts and balances to prevent issues like precision errors and ratio limits.

  `Rounding and Fee Logic`:

  Throughout the contract, there are references to rounding and fee calculations. Rounding is used to simulate the effect of fees on input and output amounts during swaps, deposits, and withdrawals.

  `Comments and Explanations`:

  The codebase includes comments that provide explanations for the purpose and functionality of various functions, calculations, and constraints.

# [A-05] Centralization risks

The EvolvingProteus contract does not show ownership or permission mechanisms explicitly. If there are centralized mechanisms that control ownership or permissions within the contract, it could introduce centralization risks.

# [A-06] Systemic risks

1. **Complexity and Inherent Risks**: The code appears to be quite complex, involving multiple libraries, mathematical calculations, and constraints. Complex code is more prone to bugs, vulnerabilities, and unexpected behaviors.

2. **Rounding Errors and Constraints** Rounding errors and enforcing certain constraints. However, these constraints can interact in non-trivial ways, and extreme parameter values may lead to rounding errors that affect user balances. There's a need for rigorous testing to ensure that these constraints are effective and don't inadvertently cause unexpected behaviors.

3. **Mathematical Manipulation**: The code heavily relies on mathematical calculations involving custom functions and external libraries. Incorrect or imprecise calculations could lead to unexpected outcomes or vulnerabilities.

4. **No Comprehensive Security Checks**: While there are some checks for balance ratios and amounts, it's important to ensure that all possible edge cases are considered and protected against.

5. **Soft Invariants and Expected Behavior**: The concept of "soft" invariants, which might be violated due to fees and rounding errors, can add complexity to the expected behavior of the contract. Users might encounter situations where their outcomes are not as expected, which could potentially impact user trust and the overall usability of the contract.


### Time spent:
11 hours