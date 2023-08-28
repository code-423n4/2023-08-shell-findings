Evolving Proteus analysis
---------------------------------

**Approach taken in evaluating the codebase:**

1) Read the documentation: The documentation provided by the protocol developers were mostly about Static Proteus and Ocean protocol. However, it gave a detailed idea of how one can iteract with Evolving Proteus through Ocean.

2) Manual review: Manual review helped to understand the mathematical model of the evolving AMM along with finding bugs in the Evolving Proteus codebase.

**Evolving Proteus codebase analysis:**
1) The codebase is well-structured: State variables ----> Error declarations ----> Constructor ----> External functions ----> Internal functions ----> Private functions.

2) The codebase has sufficient comments (in quantity). However, there are some areas where wrong comments are provided.

3) Few lines of code make use of require() statements without error messages.

4) The contract makes use of a critical variable ```config```. This variable can be set by the deployer only at the time of deployment. However, there is no function for the deployer (or the owner) to change the value of ```config``` and restart the evolution process in case wrong values of ```config``` are set during deployment.

**Recommendations:**

It is recommended to devise an incentive strategy for the LP token holders so that they do not sell their LP tokens. This is a common problem with AMMs, when the LP token holders sell their LP tokens leading to a downfall in the LP token price. An example would be the way [Pearl Exchange](https://www.pearl.exchange/swap) have implemented a strategy for their LP token holders to deposit their LP tokens for a long period of time to gain additional incentives.

**Time Spent:**
40 hours




### Time spent:
40 hours