## Timestamp reliability on L2
Timestamp information on Arbitrum can be less reliable than on mainnet. As mentioned in [Uniswap docs](https://docs.uniswap.org/concepts/protocol/oracle#optimism):
> The `block.timestamp` of these blocks, however, reflect the `block.timestamp` of the last L1 block ingested by the Sequencer.
## Redundant code
- `a` and `b` are the results of the square root. Thus, their values won't be less than 0, making the check `if(a < 0 && b < 0)` redundant.
  ```js
  if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
  else utility = (r0 > r1) ? r0 : r1;
  ```
  https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L720-L721
- The return statement `return uf;` can be safely removed, simplifying the code.
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L663
## Typos in comments
- In the `swapGivenInputAmount` function, due to the fact that the return value `result` of the `_swap` function represents the direction of change for the output asset, it should be in the decreasing direction, meaning a negative value. Therefore, the comment here should be modified to `amount cannot be greater than 0`. The same for `withdrawGivenInputAmount` and `withdrawGivenOutputAmount` functions.
  ```js
  // amount cannot be less than 0
  require(result < 0);
  ```
  https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L295
  https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L450
  https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L487
- Calculates the b variable in the curve eq which is basically a sq. root of the ~~inverse of~~ x instantaneous price
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L120
- Instead of `10^12`, it should be `10^10`.
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L173
- @param px_final The final price at the ~~y axis~~ x axis
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L240
- `FEE_UP` should be `FEE_DOWN`
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L459