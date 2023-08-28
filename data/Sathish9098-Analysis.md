# Analysis - Shell Protocol

## Protocol Overview

A set of ``EVM-based`` smart contracts on ``Arbitrum One``. In a ``nutshell`` it is ``DeFi made simple``.

This primitive can passively update ``liquidity concentration`` over time. You can think of it like a ``hybrid`` between a ``Balancer liquidity bootstrapping pool`` and ``Uniswap v3``. The pool creator picks a starting liquidity concentration and an ending liquidity concentration. The creator also sets a starting timestamp and an ending timestamp during which liquidity concentration evolves. Every block, the primitive will update its concentration.

Two primary use cases for this primitive

#### Dutch auctions
``Dutch auctions`` work by setting the ``initial liquidity concentration`` at a ``high price`` range and setting the ending liquidity concentration at a ``low price range``. This is similar to liquidity ``bootstrapping pools``. In addition to facilitating price discovery and liquidity for newly created tokens, evolving proteus can also be used to execute large ``buy`` or ``sell orders``. For example, if ``Alice`` wants to sell ``100,000 ETH``, but wants to avoid price impact, she can put her ETH into an Evolving ``Proteus pool`` and let the pool gradually sell her ETH over a specified time duration

#### Dynamic pools
``Dynamic pools`` are ``AMMs`` that update their liquidity concentration dynamically in response to external variables. For example, a dynamic pool could continually update its ``liquidity concentration`` so that liquidity was always allocated near the current ``spot price``. Large, discontinuous updates to liquidity concentration can result in losses for ``LPs``. Small changes between each block can help mitigate these losses

### How does the algorithm work?

Evolving Proteus has the ability to passively update liquidity concentration ``every block``. The algorithm has ``six`` parameters that determine ``liquidity concentration``

- ``int128 p_max_init``: the maximum price at the initial block
- ``int128 p_min_init``: the minimum price at the initial block
- ``int128 p_max_final`` : the maximum price at the final block
- ``int128 p_min_final`` : the minimum price at the final block
- ``uint256 t_init`` : the block the AMM starts evolving
- ``uint256 t_final`` : the block the AMM stops evolving


## Approach taken in evaluating the codebase

### Steps:

- Read all documents with very fast forward way
- Then read the ``Readme.md`` file understood following things 

   -  The ``test coverage`` is only ``99%``
   -  ``AMM``
   -  Smart contracts deployed ``Arbitrum`` 
   - Main areas would be if the ``computation logic`` of the primitive can be broken and taken advantage of terms of ``swaps``, ``deposits`` & ``withdrawals`` or any scenario in terms of the computation where funds get stuck etc

- Then ``clone`` the ``repo`` and setup test environment to make sure all written ``tests`` are working correct
- Then Jump into the ``codebase analysis``. In first iteration just identified the important ``GAS`` and ``QA`` related findings. 
- Then ``deep dig`` to the documents thoroughly and ``understood protocols flow``
- Deep dive into code base with line by ``line analysis``. The weak and vulnerable areas are marked with ``@audit`` tags
- Then analyzing each and every ``@audit`` tags more deep. Then performed all possible ``unit`` and ``fuzzing tests`` to ensured the functions are ``working indented way``.
- Then i reported final reports as per ``C4`` Formats 

## Codebase quality analysis and Possible Risks Comments

[EvolvingProteus.sol](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol)

### Code Suggestions:

- Use ``Latest`` version version of ``solidity``
- Consider the ``named imports`` instead of over all file
- Use seperate files for ``library's`` 
- Consider adding ``events`` for off chain purposes 
- Variable names like ``xf``, ``yf``, ``ui``, ``uf``, ``f_0``, ``f_1``, and ``f_``2 might be more meaningful if named more descriptively
- Replace variable names like ``py_init`` and ``px_init`` with more descriptive names like ``initialYPrice`` and ``initialXPrice``. This makes the code self-explanatory
- Apply ``visibility modifiers`` (public, internal, private) explicitly to functions and state variables to enhance clarity
- Use ``require`` statements to check preconditions in functions. For example, in newConfig, use ``require(_duration > 0, "Duration must be greater than zero");`` to ensure a valid duration
- Ensure consistent ``indentation`` and ``formatting throughout`` the codebase. This improves readability
- Create a ``modifier`` to ensure that the balance ratio conditions ``(MIN_M and MAX_M)`` are maintained when ``depositing``, ``withdrawing``, or ``swapping tokens``.
- Follow a ``consistent naming convention`` for ``variables``, ``functions``, and ``modifiers``. For example, use ``camelCase`` for variables and functions and CapitalCase for modifiers
- Implement ``input validation`` for functions that accept ``external inputs`` to ensure that invalid or malicious inputs are rejected
- Add ``comments`` to ``complex calculations``, especially in the ``LibConfig library``, to explain the purpose and ``mathematical logic`` behind the calculations
- Consider using an ``enum`` for ``feeDirection`` instead of using boolean constants ``(FEE_UP and FEE_DOWN)``. This can improve code readability
- Identify ``common code pattern``s and extract them into ``separate utility functions``. For instance, the fee application logic can be extracted into a separate function to ``reduce redundancy``
- Replace magic numbers like 2, ``MULTIPLIER``, ``BASE_FEE``, and ``FIXED_FEE`` with well-named constants to improve code readability
- Consider providing more detailed ``error messages`` in your ``custom error types`` to provide better feedback when exceptions occur
- Use ``require`` statements for validating conditions instead of using if statements. This can help in clearly ``communicating the requirements`` for function execution
- Reverting the transaction inside internal view functions can`` lead to gas wastage``. Consider returning appropriate ``error values`` instead

### Risks: 

1. ``_getUtility``, ``_getUtilityFinalLp``, ``_getPointGivenXandUtility``, ``_getPointGivenYandUtility`` complex calculations involving square roots and fixed-point arithmetic can result in high gas costs
2. ``swapGivenInputAmount``, ``swapGivenOutputAmount``, ``depositGivenInputAmount``, ``withdrawGivenOutputAmount`` transactions that change the contract state based on input amounts can be susceptible to front-running attacks. Attackers may attempt to submit transactions that exploit these changes to their advantage. Implement safeguards to minimize the impact of front-running, such as checking the current state before executing an operation.
3. For ``Token swaps`` consider,
  - Consider Adding ``Slippage Protection``
  - ``Deadlines`` for Token swaps
  - Check ``result`` with ``expected slippage ``

4. Hardcoded ``BASE_FEE`` and ``FIXED_FEE`` may cause problems in the future
5. If the comment states that the ``amount cannot be less than 0,`` but the ``require(result < 0)`` is checking for a negative result, then indeed there is an inconsistency between the comment and the condition
6. Potential Denial-of-Service (DoS) Vulnerability in ``_checkAmountWithBalance()``


## Gas Optimization

In the pursuit of improving the efficiency and cost-effectiveness of the ``EvolvingProteus`` contract, a series of optimization techniques have been employed. These optimizations are aimed at ``reducing gas consumption``, ``enhancing storage slot utilization``, and streamlining contract functions to ``maximize gas efficiency``.

One key optimization strategy involves caching frequently accessed ``storage`` variables in ``memory`` when making ``consecutive calls``. By storing the value of these variables in memory, the contract significantly reduces the ``gas overhead`` associated with redundant storage reads. As a result, contributing to substantial gas efficiency gains.

Furthermore, an optimization technique to pack ``struct members`` efficiently has been implemented. This optimization involves grouping variables that require less than ``32 bytes`` into a single storage slot. Specifically, the ``t_init`` and ``t_final`` variables have been adjusted from ``uint256`` to ``uint128``, resulting improved storage slot utilization.

In addition, ``constant variables`` have been optimized to save gas. Instead of computing these constants at ``runtime``, their values have been directly assigned, reducing ``unnecessary computation`` and ``conserving gas``. 

Lastly, ``function call`` caching has been introduced to prevent ``repeated calls`` and reduce ``unnecessary gas consumption``. By caching function call results, the contract avoids redundant computations and lowers gas costs associated with such operations.

These optimizations collectively contribute to a more gas-efficient ``EvolvingProteus contract``, enhancing its overall performance and cost-effectiveness

## Time Spend
10 hours 

### Time spent:
10 hours