
# 🛠️ Analysis - Shell Protocol 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|e) |Systemic risks | Potential systemic risks in the project |
|f) |Competition analysis| What are similar projects? |
|g) |Security Approach of the Project | Audit approach of the Project |
|h) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|i) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|j) |Other recommendations | What is unique? How are the existing patterns used? |
|k) |New insights and learning from this audit | Things learned from the project |

## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-08-shell

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-08-shell#testing)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [Proteus-AMM](https://shell-protocol.notion.site/Proteus-AMM-Engine-Update-3-7f33b7e1561347b696874a8ba02b9782) explainer provides a high-level overview of the Project system and the describe the core components.|Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from scratch |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-08-shell/tree/main/src/test)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-08-shell#scope)|Top-down analysis of codes according to architectural design, IDE used: VsCode|
|7|Project White Paper |[WhitePaper](https://shell-protocol.notion.site/Proteus-AMM-Engine-Update-3-7f33b7e1561347b696874a8ba02b9782)||
|8|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|9|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-08-shell#use-cases-for-evolving-proteus)|Evolving Proteus's algorithm has six parameters that determine liquidity concentration|

## b) Analysis of the code base

![image](https://github.com/code-423n4/2023-08-shell/assets/104318932/400f9589-3205-44b7-acdd-fd40e294b529)
<br />
<br />


![image](https://github.com/code-423n4/2023-08-shell/assets/104318932/4b4642b2-6542-43e1-8bc8-10f82b0db842)
<br />
<br />


## c) Test analysis


<img width="1069" alt="image" src="https://github.com/code-423n4/2023-07-moonwell-findings/assets/104318932/d74148f3-53c5-414d-9da0-06eeb6ef1b6a">



## 1- ForkEvolvingProteus.t.sol : 

Invariant tests can be great tools for shaking out invalid assumptions, complex edge cases, and unexpected interactions in a smart contract system. But it can also be challenging to channel the fuzzer's unconstrained chaos into a suite of meaningful, reliable tests.


### Evolving Proteus Invariants test list ( ForkEvolvingProteus.t.sol ):

| Number |Head | Test Details|
|:--|:----------------|:------|
|1) |Token balance check | The differences in pool token balances after a swap is same as the differences in user token balances after factoring in the fee |
|2) |Token supply check | Utility & utility/lp token supply does not decrease after swap|
|3) |Deposit check | For a deposit, utility will always increase, and util/lp does not decrease |
|4) |Withdraw check | For a withdraw, utility will always decrease, and util/lp does not decrease |
|5) |Soft invariant| x price decreases over time & y price stays constant |
|6) |Soft invariant| when x price increases over time & y price stays constant  |
|7) |Soft invariant| when y price decreases over time & x price stays constant |
|8) |Soft invariant| when y price increases over time & x price stays constant |
|9) |Soft invariant| when x & y prices both increase with time |
|10) |Soft invariant| when x & y prices both decrease with time |
|11) |Soft invariant| when x & y prices both stay constant |


### Invariant tests recommended to be added ( ForkEvolvingProteus.t.sol ):

| Number |Head | Test Details|
|:--|:----------------|:------|
|1) |Soft invariant| when x price increases over time &  y price decrease |
|2) |Soft invariant | when y price increases over time &  x price decreases |
 




##  d) Centralization risks 
There is no centrality risk in the controlled scope.
  
##  d) Systemic risks 
We can talk about a systemic risk here because there are some Layer2 special cases.
If Arbitrum is compile [Compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with these versions will result in a non-functional or potentially damaged version that does not behave as expected. The default behavior of the compiler will be to use the latest version which means it will compile with version 0.8.20 which will produce broken code by default.

According to the given code, this problem does not exist, but it should be taken care of while deploying.
<br />



## f) Competition analysis
Other projects with a similar structure to the project;
1 - [dfx finance](https://app.dfx.finance/)

2- [Gamma](https://docs.gamma.xyz/gamma/)

3- [Osmosis](https://github.com/osmosis-labs/osmosis)

However, with its mathematical modeling, the Shell Protocol is unique.

## g) Security Approach of the Project

#### Successful current security understandings of the project;

1- Most important security point of Shell protocol is contracts are non-custodial, meaning NOBODY (not even the core team or the Shell DAO) is able to access the funds held in any of the core contracts. 

2- Proteus Pool Safeguards System ; A pool's deployer can assign a wallet the ability to freeze the pool in case of an emergency. LPs may still withdraw their tokens, but swaps and deposits are disabled.
The Shell core team controls a 2/3 multi-signature wallet, the ShellMultiSig, that can freeze trading on Proteus pools assigned to it.

3- On-Chain Monitoring System ; Shell conducts on-chain security monitoring with Forta. Anyone can subscribe to updates from the Shell Forta bot.
This bot tracks Shell transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert.
https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- First they did the main audit from a reputable auditing organization like Trail of Bits and resolved all the security concerns in the report
https://github.com/trailofbits/publications/blob/master/reviews/ShellProtocolv2.pdf

5- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.

6- They was set the $ 100,000 ImmuneFi reward
https://immunefi.com/bounty/shellprotocol/



####  What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- No Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

4- No Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.


## h) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-08-shell/blob/main/bot-report.md
Especially Low  detections in the Automated Finding Report should be taken into account;



**Other Audit Reports (TrailOfBits):**

Here is a summary of the issues found in the audit report with Exposure Anlysis and Category details:
[Audit Report TrailOfBits](https://github.com/trailofbits/publications/blob/master/reviews/ShellProtocolv2.pdf)
<img width="654" alt="image" src="https://github.com/code-423n4/2023-08-shell/assets/104318932/422c099c-7b57-4493-826e-6885fc1f20d8">

## i) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

**Gas Optimization Recommendations**

1- The highest gas expenditure in the project occurs when creating pool instances, if wish the architecture here was design like the singleton architecture in UniswapV4, gas savings will be close to 90%.

2- The most important gas usage of the contract subject to the audit is the `ABDKMath64x64.sol` library that it uses. More gas optimized [prb-math](https://github.com/PaulRBerg/prb-math) may be preferred
detailed gas comparisons are available on the project's github link

### PRBMath

Gas estimations based on the [v2.0.1](https://github.com/PaulRBerg/prb-math/releases/tag/v2.0.1) and the
[v3.0.0](https://github.com/PaulRBerg/prb-math/releases/tag/v3.0.0) releases.

| SD59x18 | Min | Max   | Avg  |     | UD60x18 | Min  | Max   | Avg  |
| ------- | --- | ----- | ---- | --- | ------- | ---- | ----- | ---- |
| abs     | 68  | 72    | 70   |     | n/a     | n/a  | n/a   | n/a  |
| avg     | 95  | 105   | 100  |     | avg     | 57   | 57    | 57   |
| ceil    | 82  | 117   | 101  |     | ceil    | 78   | 78    | 78   |
| div     | 431 | 483   | 451  |     | div     | 205  | 205   | 205  |
| exp     | 38  | 2797  | 2263 |     | exp     | 1874 | 2742  | 2244 |
| exp2    | 63  | 2678  | 2104 |     | exp2    | 1784 | 2652  | 2156 |
| floor   | 82  | 117   | 101  |     | floor   | 43   | 43    | 43   |
| frac    | 23  | 23    | 23   |     | frac    | 23   | 23    | 23   |
| gm      | 26  | 892   | 690  |     | gm      | 26   | 893   | 691  |
| inv     | 40  | 40    | 40   |     | inv     | 40   | 40    | 40   |
| ln      | 463 | 7306  | 4724 |     | ln      | 419  | 6902  | 3814 |
| log10   | 104 | 9074  | 4337 |     | log10   | 503  | 8695  | 4571 |
| log2    | 377 | 7241  | 4243 |     | log2    | 330  | 6825  | 3426 |
| mul     | 455 | 463   | 459  |     | mul     | 219  | 275   | 247  |
| pow     | 64  | 11338 | 8518 |     | pow     | 64   | 10637 | 6635 |
| powu    | 293 | 24745 | 5681 |     | powu    | 83   | 24535 | 5471 |
| sqrt    | 140 | 839   | 716  |     | sqrt    | 114  | 846   | 710  |

### ABDKMath64x64

Gas estimations based on the v3.0 release of ABDKMath. See my [abdk-gas-estimations](https://github.com/PaulRBerg/abdk-gas-estimations) repo.

| Method | Min  | Max  | Avg  |
| ------ | ---- | ---- | ---- |
| abs    | 88   | 92   | 90   |
| avg    | 41   | 41   | 41   |
| div    | 168  | 168  | 168  |
| exp    | 77   | 3780 | 2687 |
| exp2   | 77   | 3600 | 2746 |
| gavg   | 166  | 875  | 719  |
| inv    | 157  | 157  | 157  |
| ln     | 7074 | 7164 | 7126 |
| log2   | 6972 | 7062 | 7024 |
| mul    | 111  | 111  | 111  |
| pow    | 303  | 4740 | 1792 |
| sqrt   | 129  | 809  | 699  |




## j) Other recommendations

- Logic similar to singleton pattern similar to Uniswapv4 can be used in a project, this is a gas-optimized and innovative solution
(Expressed for the overall project, not for the audit contract)


- The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns;
https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

- It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted.
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

- A good model can be used to systematically assess the risk of the project, for example this modeling is recommended;
https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

- Here are my other suggestions for corrections, the details of which are in my QA Report (I didn't want to write it here again)
##### QA Report Issues List

- [x] **Low 01** → ` result ` require  block  is inconsistent with NatSpec comment
- [x] **Low 02** → `libusb` dependency can be a security risk
- [x] **Low 03** → There is a difference in the formula with the NatSpec comments
- [x] **Low 04** → There may be problems with the use of Arbitrum
- [x] **Non-Critical  05** → Project Upgrade and Stop Scenario should be
- [x] **Non-Critical  06** → Missing Event for  initialize


## k) New insights and learning from this audit 

- `ForkEvolvingProteus.t.sol` invariant tests are extensive, quality invariant tests in a high-mathematical computational project are remarkable, the project developers did a great job
- I learned in depth the usage and details of `ABDKMath64x64.sol` library



### Time spent:
13 hours