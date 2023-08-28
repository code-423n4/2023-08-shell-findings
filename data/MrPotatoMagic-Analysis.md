# Analysis Report

## Comments to contextualize my findings

Findings: 1 Medium, 1 QA Report [4NC, 6L], 1 Gas Optimizations Report

Medium issue: My medium issue focuses on a multi-statement loss of precision issue occurring due to division before multiplication in the internal calculations. The solution to this is a simplified equation that reduces the loss of precision compared to the multi-statement implementation. 

Gas Report - Although there are only 3 Gas Optimizations, these are more focused on simplifying and refactoring equations to save gas. The total gas saved from these three issues is 20582 gas.

QA Report - There are 4NC and 6L issues, which focus on improving the current maintainability of the code, reducing user errors (since the code is view-only) and optimising edge cases and terminologies. 

## Approach taken in evaluating the codebase

**Days 1-4:**
 - The first three days were completely spent behind understanding the Evolving Proteus model and gaining additional context on the [Proteus AMM engine](https://shell-protocol.notion.site/Proteus-AMM-Engine-Update-3-7f33b7e1561347b696874a8ba02b9782).
 - Since the contract only has view functions it was important for me to ensure the mathematical calculations done in this contract are correct.  
 - Checked if constrains mentioned in the README.md are respected.
 - Adding inline bookmarks for notes and optimizations wherever necessary
**Days 5-7:**
 - These three days were mainly focused on proving the inline bookmark optimizations to be correct. This included evaluation of non-critical, low-severity and any medium issues.
 - Creation of QA, Gas, Medium-severity and Analysis reports 

## Architecture recommendations
The current architecture is well-structured but due to the math-heaviness of the contract I would recommend two things:
1. Store the library LibConfig and contract EvolvingProteus in separate files.
2. Separate the internal and external view functions of the EvolvingProteus contract to further improve readability and maintainability of the contract.

## Codebase quality analysis
The team has kept themselves one step ahead in security by including a very high percentage of test coverage (approx 90%). Other than this, since the functions are of view visibility only, it was important to ensure the contract functions as expected and respects the constraints set by the developer team. There was one exception to this which I have included in my Medium-issue and QA report. Overall, the codebase quality deserves a high rank.

## Mechanism review

The mechanism is quite detailed. To understand it fully, I would recommend going through this document provided by the developer team to gain context on the Proteus AMM engine: https://shell-protocol.notion.site/Proteus-AMM-Engine-Update-3-7f33b7e1561347b696874a8ba02b9782

The high-level overview would include three types of external functions: Swap/Deposit/Withdraw which have respective input and output token functions to represent the flow of tokens from X to Y and vice-versa. The remaining internal functions can be categorised into helper functions that do the detailed calculations for these external functions or the Evolving Proteus curve.





### Time spent:
50 hours