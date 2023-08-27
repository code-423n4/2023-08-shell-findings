## [01] Detailed Analysis Of Shell Protocol [In Scope]
#### [1.1] General Inroduction About Shell Protocol
The Shell Protocol consists of smart contracts built on the Arbitrum One platform using the EVM. In contrast to typical DeFi protocols that utilize large, specialized smart contracts, Shell serves as a central point for a flexible network of services. Its structure streamlines the process of combining multiple smart contracts or creating fresh ones, and allows users to group numerous services together in a single transaction.
#### [1.1] Brief Introduction About The Codebase In Scope 
The current audit have only one contract in scope which is ```EvolvingProteus.sol```. 

```What is Evolving Proteus?```

It's is explained on C4 Contest Page, furthermore, it will be more summarized here in detailed manner as below

Evolving Proteus is a financial mechanism designed to dynamically adjust liquidity concentration in a pool over time. It combines features of Balancer liquidity bootstrapping pools and Uniswap v3. Pool creators define a starting and ending liquidity concentration, as well as initial and final timestamps for the evolving liquidity. The mechanism updates its concentration every block, offering utility in various contexts.

#### [1.2] Analysis of the contract in scope

To get the better understanding of the protocol, it is better to look and understand the codebase and it’s smart contract that is in scope. 

##### [1.2.1] Key feature of ```EvolvingProteus.sol```

It implements a liquidity pool with a bonding curve that evolves over time. The bonding curve is defined by two parameters, "a" and "b", which are calculated based on the prices of two tokens in the pool. The curve evolves from an initial state to a final state over a specified duration.

##### [1.2.2] Key Functions/Components of ```EvolvingProteus.sol```

```EvolvingProteus``` includes functions for swapping tokens, depositing tokens into the pool, and withdrawing tokens from the pool. Each of these functions adjusts the state of the pool and the bonding curve accordingly.

##### [1.2.3] Lib Config Library of ```EvolvingProteus.sol```

To my understanding ```LibConfig``` library and is used to calculate various parameters and values related to a mathematical curve. This curve is defined by changing prices over time. The main purpose of this code is to create and manipulate configurations for this curve.

##### [1.2.4]  swapGivenInputAmount / swapGivenOutputAmount

The basic function of swapGivenInputAmount and swapGivenOutputAmount is dealing with swapping one reserve token for another while maintaining invariants. These functions ensure that the utility of the system is preserved and that the swap doesn't lead to negative balances or incorrect amounts.

* #### swapGivenInputAmount : How it works

- Check that inputAmount, ```xBalance```, and ```yBalance``` are below a certain threshold.
- Validate that the provided inputAmount doesn't exceed the available balance.
- Call the internal _swap function with the adjusted input amount and other parameters.
- Ensure the result of _swap is negative (indicating a valid swap output).
- Calculate the output amount based on the negated result of _swap.
- Validate that the calculated output amount doesn't exceed the available balance of the output token.

* #### swapGivenOutputAmount : How it works

- Check that outputAmount, ```xBalance```, and ```yBalance``` are below a certain threshold.
- Validate that the provided outputAmount doesn't exceed the available balance.
- Call the internal ```_swap``` function with the negated output amount and other parameters.
- Ensure the result of ```_swap`` is positive (indicating a valid swap input).
- Calculate the input amount based on the result of _swap.
- Validate that the calculated input amount doesn't exceed the available balance of the input token.

#### [1.2.5] depositGivenInputAmount and depositGivenOutputAmount

These two functions handle the depositing of assets into a pool and the corresponding minting of LP tokens. These functions ensure that the utility of the pool is scaled proportionally to the deposited or minted amounts while considering the current balances of reserve tokens and the total supply of LP tokens.

* #### depositGivenInputAmount : How it works

- Check that depositedAmount, ```xBalance```, ```yBalance```, and ```totalSupply``` are below a certain threshold.
- Make Sure that the provided depositedAmount doesn't exceed the available balance.
- ```_reserveTokenSpecified``` is called internally with adjusted deposited amount.
-The Results of ```_reserveTokenSpecified``` must be positive (indicating a valid minting amount).

* #### depositGivenOutputAmount : How it works

- ```depositGivenOutputAmount``` is similar to depositGivenInputAmount for first two steps but 
- ```_lpTokenSpecified``` function is called internally with the adjusted minted amount

#### [1.2.6] withdrawGivenInputAmount and withdrawGivenOutputAmount

These two functions manage the extraction of assets from a pool and the concurrent reduction of LP tokens through burning. They guarantee that the pool's overall value aligns with the withdrawn or burned quantities, all while factoring in the existing reserve token balances and the total supply of LP tokens.

* #### withdrawGivenInputAmount : How it works

- It takes several inputs: xBalance and yBalance represent the balances of two reserve tokens in the pool, totalSupply is the total supply of LP tokens, withdrawnAmount is the desired decrease in LP tokens, and withdrawnToken is the token being withdrawn.
- It performs various validations to ensure that the inputs are within acceptable limits.
- It calls the _reserveTokenSpecified function with relevant parameters. This function calculates the amount of the specified token that needs to be withdrawn given the specified decrease in LP tokens and some other factors.
- The result of the _reserveTokenSpecified call is checked to ensure it's negative (indicating a valid withdrawal) and then converted to an unsigned integer, representing the burned amount of the specified token.

* #### withdrawGivenOutputAmount : How it works

- Similar to the first function, this function takes inputs like xBalance, yBalance, totalSupply, burnedAmount, and withdrawnToken.
- Input validations are performed.
- It calls the _lpTokenSpecified function with appropriate parameters. This function calculates the amount of LP tokens that need to be burned given the desired decrease in the specified token. 
- The result of the _lpTokenSpecified call is checked to ensure it's negative (indicating a valid withdrawal) and then converted to an unsigned integer, representing the amount of LP tokens that need to be burned.

#### [1.2.7] _swap

It  handles the process of making swaps in a liquidity pool. Imagine you have a pool with two types of tokens, let's call them X and Y. Swaps can happen in four ways: you can add X, remove X, add Y, or remove Y from the pool. This function helps manage these swaps and ensures that the pool remains balanced and fair.

* #### How it works

- Inputs: When you want to make a swap, you provide some information: whether fees should be added or subtracted (feeDirection), how much you want to swap (specifiedAmount), what the initial balances of X and Y are (xi and yi), and which token you're swapping (specifiedToken).

- Fee and Rounding: The amount you want to swap is adjusted to account for fees using a process called rounding. This adjusted amount is stored as roundedSpecifiedAmount.

- Calculating New Balances: Next, the function calculates what the new balances of X and Y will be after the swap. This calculation considers the utility of the pool, which is like a measure of how well the pool is functioning.

- Finalizing the Swap: Depending on whether you're swapping X or Y, the function calculates how much of the other token needs to be swapped to keep things fair. It also checks if these swaps maintain the proper balance of tokens in the pool.

- Result: The function provides you with the actual amount that will be swapped in the other token (the non-specified one). This helps you understand exactly what the swap will involve.

#### [1.2.8] Remaining Components To Make The Analysis Short And Precise

These functions work together to manage the mechanisms of a liquidity pool and facilitate token swaps while considering factors such as fees, utility, and token balances. Let's provide a summary for each function:

1. **Function: `_reserveTokenSpecified`**
   - This function calculates the amount of a specified token to be deposited or withdrawn from a liquidity pool, while adjusting for fees and maintaining utility equilibrium.
   - It considers the change in balance, specified token, fee direction, initial balances, and utility.
   - Calculates the new token balance considering the fee.
   - Calculates initial and final utilities and uses these to adjust the total supply.
   - Returns the computed amount considering fees.

2. **Function: `_lpTokenSpecified`**
   - This function calculates the amount of LP tokens to be minted or burned during a deposit or withdrawal, while accommodating fees and utility preservation.
   - It calculates the final utility considering fees.
   - Determines the final price points and new balances based on the specified token.
   - Checks balances to ensure the validity of the swap.
   - Returns the computed amount considering fees.

3. **Function: `_getUtilityFinalLp`**
   - Calculates the final utility after a deposit or withdrawal using the change in total supply.
   - Considers the initial and final total supply, as well as initial token balances.
   - Returns the updated utility value.

4. **Function: `_findFinalPoint`**
   - Utilizes the properties of the pool to find the final state of balances after a given action.
   - Takes a known coordinate and utility to find the other coordinate.
   - Works with two different coordinate-getting functions depending on the specified token.

5. **Function: `_getUtility`**
   - Calculates the utility of the liquidity pool based on the balances of reserve tokens.
   - Utilizes the pool's internal measure of token value and considers the pool's bonding curve parameters.
   - Returns the calculated utility.

6. **Functions: `_getPointGivenXandUtility`, `_getPointGivenYandUtility`**
   - Calculate the coordinate (X or Y) based on the utility and a known coordinate.
   - These functions are used interchangeably to handle different swap directions.

7. **Function: `_checkAmountWithBalance`**
   - Ensures that the ratio between a balance and an input/output amount doesn't exceed a certain threshold.
   - Prevents precision errors during swaps with very small amounts against a large pool.

8. **Function: `_checkBalances`**
   - Checks that the pool's balances are above a minimum threshold and the ratio of reserve tokens is within a defined interval.

9. **Function: `_applyFeeByRounding`**
   - Adjusts an amount by applying rounding or fees in the appropriate direction.
   - Considers both fixed and variable fees and ensures stability in calculations.

## [02] Architectural Improvements

some potential architectural improvements that can be made to enhance its precision and efficiency:

1. **Library Separation:**
   The contract could benefit from separating the configuration-related functions into their own standalone library. This would help in keeping the code modular and easier to read. The `LibConfig` library could be extracted into its own file, making it more maintainable.

2. **Rounding and Overflow Handling:**
   The contract currently uses integer and fixed-point arithmetic extensively. To prevent potential rounding errors and overflow issues, consider using the SafeMath library for arithmetic operations. SafeMath prevents integer underflows and overflows by reverting the transaction if they are detected.

3. **Parameter Validation and Error Handling:**
   While the contract has error checks for maximum and minimum price values, it might be helpful to provide more detailed error messages in the require statements. This can make it easier to identify the exact condition that triggered the error.

4. **Constants and Magic Numbers:**
   The contract has a number of magic numbers and constants that are used in calculations. It's a good practice to define these constants as named variables at the top of the contract for better readability and maintainability.

5. **Efficient Use of Storage:**
   In some calculations, the `uint256` variables are used even though smaller data types like `uint128` could suffice. Using the smallest data type that can accommodate the value helps to save storage costs.

6. **Code Comments and Documentation:**
   While the contract has some comments, adding more explanatory comments throughout the code can make it easier for other developers (and future you) to understand the logic and purpose of various functions.

7. **Optimization of Calculations:**
   Some calculations involve repeated operations. Consider caching results when possible to improve efficiency.

8. **Gas Optimization:**
    Careful consideration should be given to gas costs. The contract involves complex mathematical operations that could potentially be gas-intensive. It's important to ensure that the contract remains cost-effective for users.

9. **Testing:**
    Rigorous testing,invariant testing , should be performed on the contract to verify its correctness and robustness. This is especially important given the complexity of the mathematical calculations involved.

## [03] Centralization Risk

There aren't any specific Centralisations risks in the codebase, however, there are some general risks that are mentioned below:


1. **Configuration Parameters and Initialization**:
   The initial parameters of the contract (py_init, px_init, py_final, px_final, duration) are set during deployment. These parameters dictate how the curve evolves over time. If these parameters are set in a way that creates unfair advantages for certain users or parties, it could lead to centralization of control over the AMM.

2. **Curve Equations and Calculations**:
   The curve equations used in the contract's utility calculations and point extrapolations are complex. If there are any vulnerabilities or inaccuracies in these equations, it could be exploited by certain participants, potentially leading to centralization of control over the AMM.

3. **Constraints on Ratios and Balances**:
   The contract includes checks on the ratio between balances and the amount being swapped or deposited/withdrawn. If these checks are not properly implemented or if they are biased toward certain participants, it could lead to centralization.

## [04] Time Spend
Day1 : Reading The Docs And Info Provided about Protocol on C4 Audit Page

Day2: Thoroughly and Deeply Studied Code

Day3: Tried Understanding Code And Finding Weak Spots

Day4: Writing Report
#### [4.1] Total Time Spent
30-32 Hours




### Time spent:
32 hours