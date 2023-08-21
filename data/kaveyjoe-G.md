## GAS Optimization 1: Use ABDKMath64x64 functions consistently

use ABDKMath64x64.fromUInt to convert the INT_MAX value, we ensure consistent usage of the ABDK library functions for mathematical operations.
- original line 

              uint256 constant INT_MAX = uint256(type(int256).max);
- Suggested Fix 

             int256 constant INT_MAX = ABDKMath64x64.fromUInt(type(int256).max);


## Gas Optimization 2: Remove unnecessary calculations
Use ABDKMath64x64.fromUInt to define the MULTIPLIER constant ensures accurate calculations with the ABDK library.
- Original line 

               int256 constant MULTIPLIER = 1e18;
- suggested Fix

                int128 constant MULTIPLIER = ABDKMath64x64.fromUInt(1e18);


## Gas Optimization 3: Use ABDKMath64x64 for mathematical operations
Use ABDKMath64x64 for square root calculations ensures consistent and accurate mathematical operations throughout the contract.
- original code


               int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
_ Suggested fix


                  int256 disc = ABDKMath64x64.to64x64(ABDKMath64x64.sqrt(ABDKMath64x64.fromUInt(uint256(bQuad**2 
                   - (aQuad.muli(cQuad)*4)))));


## Optimization 4: Use ABDKMath64x64 for division
Use ABDKMath64x64 for division ensures precision and accurate results in the calculation.
- original line 


                uint256 result = Math.mulDiv(uint256(ui), uint256(sf), uint256(si));

- Suggested Fix 


                uint256 result = ABDKMath64x64.toUInt(ABDKMath64x64.divu(ABDKMath64x64.fromUInt(uint256(ui)), 
                ABDKMath64x64.fromUInt(uint256(si))).mulu(ABDKMath64x64.fromUInt(uint256(sf)))));


## Gas Optimization 5: Replace conditions with ternary operators
Use ternary operators in this context simplifies the code and makes it more concise.
- original line


                   if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;
                   else utility = (r0 > r1) ? r0 : r1;
- Suggested fix 


               utility = ((a < 0 && b < 0) ? (r0 > r1) ? r1 : r0 : (r0 > r1) ? r0 : r1);

## Gas Optimization 6: Use ABDKMath64x64 for absolute value
Use ABDKMath64x64.abs to calculate the absolute value of utility ensures accuracy and consistent usage of the library functions.
- Orignal line 


             if (utility < 0) revert CurveError(utility);
- Suggested fix


                if (ABDKMath64x64.toUInt(utility) > ABDKMath64x64.toUInt(utility.abs())) revert CurveError(utility);


## Gas Optimization 7: Simplify boundary checks

Combine the two boundary checks into a single condition simplifies the code and reduces redundancy.
- Original code 
                
                  if (finalBalanceRatio < MIN_M) revert BoundaryError(x,y);
                 else if (MAX_M <= finalBalanceRatio) revert BoundaryError(x,y);
- suggested fix

                    if (finalBalanceRatio < MIN_M || MAX_M <= finalBalanceRatio) revert BoundaryError(x,y);
