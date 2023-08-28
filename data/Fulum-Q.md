# [01] Code and comment mismatch

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L119-L125

Notice line is wrong, you need to change it.

	/**
      -- @notice Calculates the b variable in the curve eq which is basically a sq. root of the inverse of x instantaneous price
      ++ @notice Calculates the b variable in the curve eq which is basically a sq. root of x instantaneous price
       @param self config instance
    */
    function b(Config storage self) public view returns (int128) {
        return p_min(self).sqrt();
    }
	
# [02] Code and comment mismatch

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L254-L256

The comment doesn't match the code.

	-- // at all times x price should be less than y price
	++ // at all times y price should be less than x price
        if (py_init <= px_init) revert InvalidPrice();
        if (py_final <= px_final) revert InvalidPrice();
		
# [03] Code and comment mismatch		

In some functions, the amount must be less than 0 because the sign is inverted just after it.
These comment lines doesn't match the code.

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L335-L336

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L450-L451

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L487-L488

	-- // amount cannot be less than 0
	++ // amount cannot be more than 0
        require(result < 0);

# [04] Code and comment mismatch

The comment doesn't match the code.

https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L455-L490

	 -- * @dev We use FEE_UP because we want to increase the perceived amount of
     -- *  reserve tokens leaving the pool and to increase the observed amount of
     -- *  LP tokens being burned.
	 ++ * @dev We use FEE_DOWN because we want to decrease the perceived amount of
     ++ *  reserve tokens leaving the pool and to decrease the observed amount of
     ++ *  LP tokens.
     */
    function withdrawGivenInputAmount(
        uint256 xBalance,
        uint256 yBalance,
        uint256 totalSupply,
        uint256 burnedAmount,
        SpecifiedToken withdrawnToken
    ) external view returns (uint256 withdrawnAmount) {
	
        ...

        int256 result = _lpTokenSpecified(
            withdrawnToken,
            -int256(burnedAmount),
            FEE_DOWN,
            int256(totalSupply),
            int256(xBalance),
            int256(yBalance)
        );

