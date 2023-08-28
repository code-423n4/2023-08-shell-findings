Concentrated Liquidity is a enhancement to AMMs in DeFi. The ```EvolvingProteus``` contract handles the logic for evaluating input and output value during user interactions.
The contract does well in handling logical evaluations and 

I suggest that the protocol should mint LP tokens to the tune of at least ```MIN_BAL``` to prevent users funds from loosing funds owing the check in the first require statement in the ```getUtilityFinalLP(...)``` function.

### Time spent:
46 hours