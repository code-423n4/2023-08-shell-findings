### Bonding curve malleability issue
The evolving bonding curve adds great value to the existing AMM landscape by allowing more customized control over the relationship between supply and price.

It should be also noted that the control over the shape of the curve should be more precisely restricted to ensure an added layer of sanity check on the behavior of a pool, such that, there wouldn’t be situations where a swap, deposit, or withdraw with reasonable parameters yield unrealistic behaviors from the pool. 

I conducted several fuzz tests on basic scenarios of liquidity setup and trading activities, there are error cases observed where it should not be an edge case in a generalized constant product(x*y=k) AMM model. See the test cases and results as follows.

There are (3) cases below and these are the criteria:

(1) All three cases are tested with the same liquidity setup - 6000 ether Token A, 6000 ether Token B. 

(2) The initial parameters(`py`, `px`) for the pool varies between cases, which results in the difference in the initial a and b variables of the curve.

(3) Swap in both directions are tested in each case. Each direction has two tests - one fuzz test and one fixed amount test. The bounds for fuzz tests and the fixed amount for all cases are the same.

(4) In test results, `a` , `b`, and `y` (`x`) are printed in the terminal.

(5) `t` variable is not tested here. So in each case setup `py_init` == `py_final` , `px_init` == `px_final` .

#### Case 1:  token B (~4.9 ether) →token A -  Reverts (reverse swap direction succeeded)
- Initial Parameters:
```solidity
// small values even
py_init_val = 695000000000000;
px_init_val = 692930500000000;
py_final_val = 695000000000000;
px_final_val = 692930500000000;
```
- Swap: (failed case)
token B (~4.9 ether) →token A
- Results:
Revert due to calculated x0 is a negative value. This means the swap move out of the bound of the curve and returns a negative x reserve value.
```bash
Running 4 tests for src/test/fork/ForkEvolvingProteus5.t.sol:ForkEvolvingProteus5
[PASS] testSwapSmallAmmountAToB(uint256) (runs: 1000, μ: 266235, ~: 266329)
[PASS] testSwapSmallAmmountAToBFixed() (gas: 265098)
Logs:
  A to B Fixed amount:
  4900000000000000000
  ai:
  699724886246734115434
  bi:
  485584215818131120
  y0:
  5999996598763857400309

[FAIL. Reason: CurveError(-1041552782665688291154) Counterexample: calldata=0xabd0cad20000000000000000000000000000000000000000000000000000000000004c32, args=[19506 [1.95e4]]] testSwapSmallAmmountBToA(uint256) (runs: 2, μ: 252215, ~: 252215)
Logs:
  Bound Result 4900000000000019507
  ai:
  699724886246734115434
  bi:
  485584215818131120
  x0:
  -1041552782665688291154

[FAIL. Reason: CurveError(-1041552782665688291154)] testSwapSmallAmmountBToAFixed() (gas: 157204)
Logs:
  B to A Fixed amount:
  4900000000000000000
  ai:
  699724886246734115434
  bi:
  485584215818131120
  x0:
  -1041552782665688291154

Test result: FAILED. 2 passed; 2 failed; finished in 4.83s
```
#### Case 2: token B  ↔ token A - Succeeds in both swap direction
- Initial Parameters:
```solidity
// large values uneven
py_final_val = 2000000000000000000000;
px_init_val = 990000000000000000;
py_init_val = 2000000000000000000000;
px_final_val = 990000000000000000;
```
- Swap: (all tests succeeded)
- Results:
Fuzzing in both swap directions succeeds.
```bash
Running 4 tests for src/test/fork/ForkEvolvingProteus5.t.sol:ForkEvolvingProteus5
[PASS] testSwapSmallAmmountAToB(uint256) (runs: 1000, μ: 266434, ~: 266534)
[PASS] testSwapSmallAmmountAToBFixed() (gas: 265316)
Logs:
  A to B Fixed amount:
  4900000000000000000
  ai:
  412481737123559467
  bi:
  18354278608861996862
  y0:
  5987508208546127588577

[PASS] testSwapSmallAmmountBToA(uint256) (runs: 1000, μ: 252495, ~: 252603)
[PASS] testSwapSmallAmmountBToAFixed() (gas: 251493)
Logs:
  B to A Fixed amount:
  4900000000000000000
  ai:
  412481737123559467
  bi:
  18354278608861996862
  x0:
  5998084836340125883552

Test result: ok. 4 passed; 0 failed; finished in 5.22s
```
#### Case 3:  token A (~3.1 ether) →token B - Reverts (reverse swap direction succeeded)
- Initial Parameters:
```solidity
// large values even
        py_final_val = 2000000000000000000000;
        px_init_val = 1900000000000000000000;
        py_init_val = 2000000000000000000000;
        px_final_val = 1900000000000000000000;

```
- Swap: (failed case)
token A (~3.1 ether) →token B
- Results:
Revert due to calculated y0 is a negative value. This means the swap move out of the bound of the curve and returns a negative y reserve value.
```bash
Running 4 tests for src/test/fork/ForkEvolvingProteus5.t.sol:ForkEvolvingProteus5
[FAIL. Reason: CurveError(-8040124236) Counterexample: calldata=0x8678a4e17f5effc0727ffd9e55beaa65f575cffceef927e6dd5b47e66685824c41b6c115, args=[57611580527850444706201830112761370039507736006302729436560003484731470954773 [5.761e76]]] testSwapSmallAmmountAToB(uint256) (runs: 152, μ: 266337, ~: 266344)
Logs:
  Bound Result 3161804932677380050
  ai:
  412481737123559467
  bi:
  804074932546577452735
  y0:
  -8040124236

[FAIL. Reason: CurveError(-3298418262532018127774)] testSwapSmallAmmountAToBFixed() (gas: 199389)
Logs:
  A to B Fixed amount:
  4900000000000000000
  ai:
  412481737123559467
  bi:
  804074932546577452735
  y0:
  -3298418262532018127774

[PASS] testSwapSmallAmmountBToA(uint256) (runs: 1000, μ: 252305, ~: 252398)
[PASS] testSwapSmallAmmountBToAFixed() (gas: 251288)
Logs:
  B to A Fixed amount:
  4900000000000000000
  ai:
  412481737123559467
  bi:
  804074932546577452735
  x0:
  5999997424344960539987

Test result: FAILED. 2 passed; 2 failed; finished in 4.45s
```

As seen from the above cases, swapping a small amount in a pool with deep liquidity (relative to the swap amount) yields inconsistent results depending on the initial parameters.

Case 2 succeeds in both swap directions. However, Case 1 failed when swapping token B for token A, while Case 3 failed when swapping token A for token B.

When liquidity is equally split between token A and token B, Case 2 with initial parameters - large values with py greatly larger than px, gives more stable swapping results. Case 3 with large initial values and py more or less equals px failed. Case 1 with small initial values and py more or less equals px failed. 

This is not a conclusive study, but it goes to show how properly setting up initial curve parameters of py and px relative to a realistic token reserve ratio is crucial to ensure stable performance of the bonding curve. Even though many initial curve parameter combinations can be mathematically sound, not all of these combinations make sense to a particular pair of tokens.

My recommendation is to do further comparative studies with actual token pairs in live AMMs: ETH/ARB, ETH/BTC, etc. And compare the swapping results with other generalized constant product pools such as UniswapV2 or UniswapV3.  For each token pair, there might be a specific combination of py and px that performs most stably and yields more desirable results.

See test files here: (https://gist.github.com/bzpassersby/370494a58b4f48ee0314c5b0ba6d9384)

### Time spent:
20 hours