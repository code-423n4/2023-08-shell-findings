# Shell Protocol Smart Contract Analysis  
![Shell Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FYeAbAEHLV2t.0&w=256&q=75)

## Description overview of Shell Protocol project 
`Evolving Proteus` is a concept applied within the `Shell` Protocol, a project aiming to implement an automated mechanism for adjusting liquidity concentration and exchange rates in a decentralized cryptocurrency exchange setting. Think of it as a unique combination of a market where liquidity is initiated and a platform where exchange rates evolve over time.

**It's used primarily for two things**:

- **Dutch Auctions**: 
    Here, you start with a lot of one type of coin and very little of another (like they're very expensive), and you gradually shift to having more of the initially scarce type (making them cheaper). This helps discover how much each coin is worth and execute large purchases or sales without affecting the price too much.

- **Dynamic Pools**: 
    These are places where the amounts of coins change automatically to keep the price close to the current value. This way, large changes don't affect those lending money for the exchange.

In this system, there are two coins used as `"reserves."` One is the standard measuring currency for everything, like a meter for measuring distances. Usually, it's `ETH` or a `Stablecoin`. Exchange rates are referred to in terms of this standard currency.

The system has parameters that define how the amount of money will change over time. These changes are calculated with mathematical formulas using those parameters and the current time. Exchange rates are calculated according to these formulas.

To prevent errors, there are rules to keep the amounts of money within certain limits and to make sure exchanges are not too small. There are also rules to prevent balances from going negative and to maintain a certain level of money in the system.

If extreme prices are set or if the concentration of money is very high, errors can occur. That's why one must be cautious when adjusting the parameters.


## Summary
![Shell](https://github.com/catellaTech/shell1/blob/main/Shell1.drawio.png?raw=true)

## 1- Approach we followed when reviewing the code
To begin, we assessed the code's scope, which guided our approach to reviewing and analyzing the project.

### **Scope**:
-  **EvolvingProteus**: 
    The contract manages liquidity, exchanges, and deposits within a blockchain-based financial protocol using a mathematical curve called `Proteus`. Its objective is to provide an efficient and secure token exchange system that maintains token utility and ratios in the reserve as it evolves over time.

### **General Reading**: 
We started by reading the code in its entirety to understand its overall structure, the involved contracts, and the defined functions.

- **Import Statements**: 
    We checked the import statements for other contracts and libraries to understand what functionalities are being used and whether they are integrated correctly.

- **Comments and Documentation**: 
    We read the comments in the code to understand the intentions behind the defined functions, variables, and structures. These comments provide important guidance in understanding how the code works.

- **Structures and Enums**: 
    We reviewed the structures and enums defined in the code to understand how they are used and what data is stored in them.

- **Libraries and Utility Functions**: 
    We analyzed the libraries used in the code, such as ABDKMath64x64.sol and Math.sol, to understand what utility functions they provide and how they are applied in the code.

- **Main Functions**: 
    We reviewed the main functions of the contract, such as:
    1. `swapGivenInputAmount` 
    2. `swapGivenOutputAmount` 
    3. `depositGivenInputAmount` 
    4. `depositGivenOutputAmount` 
    5. `withdrawGivenOutputAmount`  
    6. `withdrawGivenInputAmount` 
   
   We analyzed how calculations, validations, and constraints are handled.

- **Business Logic**: 
    We tried to understand the business logic behind the contract and how interactions with reserves and tokens are managed.

- **Error Handling**: 
    We inspected possible error conditions handled in the functions and how custom exceptions are thrown if something goes wrong.

- **Constants and Parameters**: 
    We checked the constants and parameters used in formulas and equations to understand their purpose and how they affect results.

- **Events**: 
    We analyzed the events defined in the contract to understand what information is recorded and when these events are emitted.

- **Constructor**: 
    We reviewed the contract's constructor to understand how the contract is initialized and configured upon creation.

- **Error Comments**: 
    We analyzed custom error comments to understand the specific conditions under which they are thrown.

- **Compatibility with Standards**: 
    We checked if the contract implements specific interfaces or standards, such as `ILiquidityPoolImplementation`, and if it complies with associated requirements.

- **Final Review**: 
    Finally, we performed a general review to ensure We understand the overall structure of the contract and how all the pieces interconnect.

## 2- Analysis of the code base
The **`EvolvingProteus` contract** combines the principles of `bonding curves` and `liquidity provision` to create a dynamic pricing mechanism for tokens in the liquidity pool. Users can `swap`, `deposit`, and `withdraw` tokens while the curve's parameters evolve over time, allowing for more flexible and efficient trading and liquidity management.

- Below is a comprehensive overview of the fundamental elements outlined in this contract::

![EvolvingProteus](https://github.com/catellaTech/shell1/blob/main/shell2.drawio.png?raw=true)

## 3- Test analysis
The audit scope of the contracts to be reviewed is 95%, with the aim of reaching 100%.

### How could they have done it better?
While the provided code seems to be functional, there are always areas for improvement to ensure better security, efficiency, and readability in the contract and the protocol. 

- Here are some potential areas for improvement:

- **Modularization**: 
    Break down the code into smaller, well-defined functions and contracts. This makes the code easier to understand, test, and maintain.

- **Comments and Documentation**: 
    Provide thorough comments and documentation throughout the code to explain the purpose, functionality, and any potential gotchas of each component.

- **Input Validation**: 
    Validate and sanitize all user inputs to prevent unexpected behavior or attacks. For example, checking that input amounts are positive, non-zero, and within reasonable bounds.

- **Consistent Naming Conventions**: 
    Use consistent and descriptive variable and function names to make the code more understandable.

- **Gas Efficiency**: 
    Optimize the code for gas efficiency. This involves minimizing unnecessary computations, storage, and external calls to save on transaction costs.

- **Avoid Reentrancy Vulnerabilities**: 
    Implement safeguards to prevent reentrancy attacks, such as using the "Checks-Effects-Interactions" pattern and using the reentrancyGuard modifier to prevent multiple calls to the same function.

- **Use Latest Solidity Version**: 
    Keep up-to-date with the latest Solidity version and use its latest features and improvements.

## 4- Architectural 
![Architectural](https://github.com/catellaTech/shell1/blob/main/shell3.drawio.png?raw=true)

**In this diagram**:

- **User** represents the users who interact with the system.
- **Ocean** is the abstraction layer that allows users to conveniently interact with the methods of the `Evolving Proteus contract`.
- **Evolving Proteus** is the contract itself, which contains methods to perform different actions in the system, such as swaps and liquidity management.


## 5- Documents 
The documentation provided for the `Evolving Proteus` contract is quite comprehensive and detailed in terms of explaining its functionality, parameter usage, purpose, and overall architecture. However, here are some suggestions that could further enhance the clarity and understanding of the documentation:

- **Visual Examples**: 
    Include diagrams or visual aids to help readers understand concepts better, such as illustrating `price calculations` or `liquidity concentration evolution`.

- **Practical Examples**: 
    Provide specific numerical examples to help readers apply calculations and parameters in real-world scenarios.

- **Detailed Use Cases**: 
    Expand use cases to demonstrate how the contract applies in real life, like explaining `Dutch auctions` or `gradual buy/sell operations`.

- **Highlighted Key Terms**: 
    Boldly highlight technical terms when first mentioned to help readers identify important concepts.

##  6- Systemic & Centralization Risks
Here's an analysis of potential systemic and centralization risks in the contract:

### Systemic Risks:

- **Algorithmic Complexity**: 
    The contract involves complex mathematical calculations and interactions. Errors in these calculations could lead to incorrect outcomes, affecting the functioning of the contract and potentially leading to unexpected losses for users.

- **Rounding Errors**: 
    Rounding errors can accumulate over time due to frequent calculations involving floating-point arithmetic. These errors could lead to discrepancies between expected and actual token balances, causing loss of funds.

- **Front-Running and Manipulation**: 
    Complex financial systems can be susceptible to front-running attacks where malicious actors place transactions ahead of others to gain an advantage. The evolving nature of the bonding curve could provide opportunities for manipulation.

- **Unintended Consequences**: 
    The interactions between different parts of the contract could lead to unintended consequences, negatively impacting users. For instance, changes in the curve parameters over time could lead to unforeseen behaviors.

### Centralization Risks:

The contract relies on external data `(e.g., timestamps from block.timestamp)` to determine the curve's parameters. If this data source becomes compromised or manipulated, it could negatively impact the contract's behavior.

## 7- Security Approach of the Project

- **Minimize Complexity**: 
    Keep the smart contract logic as simple as possible to reduce the attack surface. Complex systems are more prone to bugs and vulnerabilities.

- **Separation of Concerns**: 
    Divide the contract's functionalities into smaller, modular components. This can help isolate vulnerabilities and make the contract easier to audit.

- **Secure Coding Guidelines**: 
    Enforce secure coding practices among developers. Follow best practices for Solidity development to minimize the risk of introducing vulnerabilities.

Remember that security is an ongoing process, and regular reviews, updates, and improvements are essential to stay ahead of emerging threats. Collaborating with experienced blockchain security professionals and conducting frequent security assessments can significantly enhance the security posture of the project.

## 8- Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-08-shell/blob/main/bot-report.md 

## 9- New insights and learning from this audit 
The insights we gained from auditing this protocol have been significant. We learned that `Evolving Proteus` introduces an innovative approach to passively adjusting liquidity concentration in DeFi token pools. It combines concepts from `Balancer` and `Uniswap v3`, enabling dynamic adaptation of liquidity concentration for scenarios like `Dutch auctions` and `dynamic pools`. Detailed parameters and mathematical formulas provide guidance, ensuring precision and control. The architecture is built around interaction through the `Ocean` layer. Challenges and potential risks are addressed, with invariants and precautions highlighted. Overall, `Evolving Proteus` presents a technically advanced solution to enhance liquidity and efficiency in DeFi, albeit requiring a strong understanding of decentralized finance and mathematics for its implementation.

## Conclusion 
In general, the `Shell Protocol projec` exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project. 

## Time Spent ⏱
A total of `3 days` were dedicated to completing this audit, distributed as follows:

1. 1st Day: Devoted to comprehending the protocol's functionalities and implementation.
2. 2nd Day: Initiated the analysis process, leveraging the documentation offered by the `Shell Protocol`.
3. 3rd Day: Focused on conducting a thorough analysis, incorporating meticulously crafted diagrams derived from the contracts and information provided by the protocol.













### Time spent:
72 hours