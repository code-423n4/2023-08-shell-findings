## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | State variables only set in the constructor should be declared immutable | 1 |
| [GAS-2](#GAS-2) | Optimize names to save gas | 2 |
| [GAS-3](#GAS-3) | Use != 0 instead of > 0 for unsigned integer comparison | 5 |
 
### [GAS-1] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*Instances (1)*:
```solidity
File: example/1.sol

243:     constructor(

```
### [GAS-2] Optimize names to save gas
public/external function names and public member variable names can be optimized to save gas. See this link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted,Refer:https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

*Instances (2)*:
```solidity
File: example/1.sol

7: import "@openzeppelin/contracts/utils/math/Math.sol";

137: contract EvolvingProteus is ILiquidityPoolImplementation {

```

### [GAS-3] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (5)*:
```solidity
File: example/1.sol

336:         require(result > 0);

378:         require(result > 0);

414:         require(result > 0);

755:         if (y0 < 0) revert CurveError(y0);

786:         if (x0 < 0) revert CurveError(x0);

```