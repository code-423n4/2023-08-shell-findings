## No need of if else condition in _getUtility function 
In the line https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L720

if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
        else utility = (r0 > r1) ? r0 : r1;
this if condition can be omitted as a and b will always be positive as a and b are calculated as follows  a=sqrt(1/p_max_current)

b = sqrt(p_min_current) which will always give positive value
Can directly use utility = (r0 > r1) ? r0 : r1; and if utility will be negative it will automatically revert in the next line which is 
if (utility < 0) revert CurveError(utility);