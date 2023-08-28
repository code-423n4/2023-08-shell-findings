## Analysis

EvolvingProteus is an engaging primitive to audit, however it falls short when it comes to clear code documentation and concise function naming.

The Desmos "Time Evolving Curve Demo" graph proved to be the best reference point when it came to expected behaviour and equation implementation. It was a great addition to the audit, although it would've been better if it was attached directly to the C4 README prior to the start instead of midway through in the chat. We'd recommend adding the Desmos graph to the Github README as a way for users to visualize how EvolvingProteus functions.

Off the limited scope, it's difficult to analyze and assess integration risks relative to other contracts, however solely in the context of EvolvingProteus there inherently aren't any integration or centralization risks.

### Ambiguous function naming
EvolvingProteus makes use of fairly ambiguous function naming that makes understanding functions from a user standpoint difficult. As auditors, we found ourselves continuously going back to the functions to clarify our understanding as it unnecessarily took up more of of our time. Without the descriptions given under the [Architecture section](https://code4rena.com/contests/2023-08-shell-protocol#top) for the audit, understanding from the perspective of a user would be more difficult. Consider renaming the functions to be more straightforward.

A more direct and clear way of naming these functions would be similar to their ERC4626 equivalents (given LP tokens stand for shares, reserve tokens for assets):

depositGivenInputAmount = deposit assets

depositGivenOutputAmount = mint shares

withdrawGivenOutputAmount = withdraw assets

withdrawGivenInputAmount = redeem shares

---
### Fee system needs to be rethought
There is a core design issue with the fee system that isn't negligible. 

Through testing we were able to find that it's possible to receive the same amount of tokens through withdrawGivenOutputAmount as you would with withdrawGivenInputAmount but with a smaller amount of LP tokens needed. See bug report "Exact input functions yield less value than exact output for the same parameters" for further details.

Otherwise, consider restating the discrepancy with use of these functions within documentation.

Affected functions:
- depositGivenOutputAmount/depositGivenInputAmount
- withdrawGivenOutputAmount/withdrawGivenInputAmount
- swapGivenOutputAmount/swapGivenInputAmount

---
### Flawed code documentation
Code comments are an essential part to speeding up auditing and in general getting a comprehensive understanding of the codebase's functions for users lacking Solidity experience.

Shell Protocol's EvolvingProteus has a number of flawed and incorrect comments. For example, the Updates linked to in Shell Wiki used outdated equations referring to kappa. The C4 README links to the whitepaper showing kappa as a variable value as well. The use of kappa in the code comments is mentioned throughout, throwing auditors off until it was clarified that with the update, k will evaluate to 1, hence being negligible. This constant should be taken off the comments to further simplify and only represent what's given in the code, or should be noted that k is 1.

Other essential parts to the working of EvolvingProteus such as the calculation of "a" is wrong as per the comment. Shell is not taking the inverse of the square root of the y axis price, but taking the square root of the inverse.

The comments are in need of further review prior to deployment.

---

### Complexity

Although the mathematical concepts utilized for utility calculation are straightforward, it should be clarified that the quadratic equation is being used (i.e. disc stands for discriminant) off the base equation.

aQuad, bQuad, cQuad values are used in place for a, b, c in the quadratic equation to evaluate utility.

Evaluating for the Y or X final point is simply the same base curve formula but isolated to solve for the unknown variable.

Given EvolvingProteus' similarity to the existing [AMM Engine Update 3](https://shell-protocol.notion.site/Proteus-AMM-Engine-Update-3-7f33b7e1561347b696874a8ba02b9782), we recommend giving similar if not the same examples for EvolvingProteus. The Swapping section of the Update helped clarify doubts, and aside from the Desmos, was the best resource to refer back to.



### Time spent:
30 hours