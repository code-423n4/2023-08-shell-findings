# [01] Summary of EvolvingProteus contract 

### 1.1 Description 
The `EvolvingProteus` contract is a contract responsible for passively update liquidity concentration over time for a Liquidity Pool. 
Its main function is to make computation operations through six differents functions. 
In construction the contract takes a starting liquidity concentration and an ending liquidity concentration. Also takes a starting timestamp and 
an ending timestamp during which liquidity concentration evolves.
Every block, the primitive amm contract will update its concentration and when the duration is over, liquidity concentration ceases to evolve and trading remains open. 

### 1.2 Utility 

Utility is the pool's internal measure of how much value it holds, the pool values the x reserve and y reserve based on how much of one it holds compared to the other.
This is the main tool of the contract and all the functions rely on it and need it to work correctly.

### 1.3 Swap functions

The `swapGivenInputAmount()` and `swapGivenOutputAmount()` functions compute amount for tokens swap in the pool, this is made with the utility (describe above) and the balances of tokenX and tokenY.
These functions make some verifications with amount, balance, etc, compute fees for the swap, and compute the output/input amount needed to perform the swap.

### 1.4 Deposit liquidity and receive LP Tokens

You can specify LP tokens amount you would like to receive and the `depositGivenOutputAmount()` function give you the amount or liquidity you have to provide. Or you can specify the amount of liquidity you
want to provide and the `depositGivenInputAmount()` function compute the amount of LP tokens you will receive. Computations are performed with LP tokens supply, balances of tokenX and tokenY,
calculated amount and utility.
 
### 1.5 Send back LP Tokens and recover liquidity.

You can specify LP tokens amount you would like to send back and the `withdrawGivenInputAmount()` function give you the amount or liquidity you will receive. Or you can specify the amount of liquidity you
want to receive and the `withdrawGivenOutputAmount()` function compute the amount of LP tokens you will send back. Computations are performed with LP tokens supply, balances of tokenX and tokenY,
calculated amount and utility.

# [02] Improvements

### 2.1 Check input variables

In the contract, proper verification of input variables is very important for good computations 
Example of the `withdrawGivenOutputAmount()` function not revert if the balances of tokenX or tokenY = 0, can lead to break the LP tokens supply. This mistake lead to break good computations for functions
 of the contract, break good computation of the utility in `_getUtilityFinalLp()`function and cause user to lose funds.
You can improve the inputs validations of the functions in the `EvolvingProteus.sol` contract, even if they are included in the `Ocean.sol` contract. This can be helpful if you use 
the `EvolvingProteus.sol` contract without the classical process (user -> ocean -> liquidity proxy -> evolving proteus).
Yes more verifications cost more gas, but I think it can be very helpful to make the computations more secure.

### 2.2 Documentations

You can improve your contract documentation, making it richer by adding more NatSpec descriptions for the input parameters, return value, etc.
More detailed documentation will help you first of all, but it will also help new developers or security researchers to better understand your contract.
You should also avoid writing incorrect documentation, as this can lead to a misunderstanding of your contract.

# [03] Time Spent

Day 1/Day 2: I read the contract, took a few notes and potentials ways for an issue. I tried to understand the context with the other contracts.
Day 3/Day 4: I took a deep dive into mathematics and make a lot of tests
Day 5: Write Analysis and Issues
Time spent:
50 hours


### Time spent:
50 hours