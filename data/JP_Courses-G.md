0. Add an unchecked block `unchecked {}` because this can never underflow/overflow.

https://github.com/code-423n4/2023-08-shell/blob/8ed551004f470489e070c1dd617d67eb4bf114e6/src/proteus/EvolvingProteus.sol#L82

    function elapsed(Config storage self) public view returns (uint256) {
        unchecked {
        return block.timestamp - self.t_init;
        }
    }
    
    
