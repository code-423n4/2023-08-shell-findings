- L589/660 - Possible division by 0 is not prevented
These functions can be called with 0 value in the input and this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

- L812/813 - If there is no difference between executing the if or the else if, the validation could be unified like this:
	if(finalBalanceRatio < MIN_M || MAX_M <= finalBalanceRatio) revert BoundaryError(x,y);
