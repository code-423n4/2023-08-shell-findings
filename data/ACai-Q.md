# The annotation of the constructor function is wrong
When describing px_final, the comment reads: "The final price at the y axis". This is obviously inaccurate, since px_final should be the final price on the x-axis.

The correct annotation should be:
```solidity
@param px_final The final price at the x axis
```

Therefore, the description of px_final should be revised.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L236-L242