##  [G-01]  Import Math Library Selectively:

use an alias (Math) for the imported library. This allows us to import only the specific functions we need from the library, reducing unnecessary code and gas costs.
- Original:


              import "@openzeppelin/contracts/utils/math/Math.sol";

- Update:


              import "@openzeppelin/contracts/utils/math/Math.sol" as Math;



## [G-02] Use Library Functions Directly:

Instead of creating a constant for ABDK_ONE, I directly used the library function int128.divu() which is available due to the using ABDKMath64x64 for int128 statement. This helps in keeping the code concise and utilizing library functions directly.

- Original:

                  int128 constant ABDK_ONE = int128(int256(1 << 64));

- Update:

                    using ABDKMath64x64 for int128;


## [G-03] Combine Similar Constants:

combine the similar constants MAX_PRICE_VALUE and MIN_PRICE_VALUE into MAX_M and MIN_M, respectively, using hexadecimal values to ensure consistency and improve code readability.
- Original:

 
          int256 constant MAX_PRICE_VALUE = 1844674407370955161600000000;
          int256 constant MIN_PRICE_VALUE = 184467440737;
- Update:


             int128 constant MAX_M = 0x5f5e1000000000000000000;
             int128 constant MIN_M = 0x00000000000002af31dc461;


## [G-04] Use Constants for Gas Efficiency:

used constants from the Solidity standard type(int128).max and 1 instead of the hardcoded hexadecimal values for MAX_M and MIN_M, respectively. This ensures clarity and maintains the intended behavior.

- Original:


          int128 constant MAX_M = 0x5f5e1000000000000000000;
          int128 constant MIN_M = 0x00000000000002af31dc461;
- Update:


        int128 constant MAX_M = type(int128).max;
        int128 constant MIN_M = 1;





## [G-05] Using Operator Overloads:

Replace the .mul() calls with the multiplication operator * which is more concise and often leads to slightly lower gas costs.

- Original:


       int128 d = m.mul(config.a()).mul(config.b());
- Update:


        int128 d = m * config.a() * config.b();


## [G-06] Reorganized and Simplified Equations:


Reorganize the equation to eliminate the use of .sub() and simplified the subtraction operation for better readability and potential gas savings.
- Original:


            int128 result = xBalance.sub(xBalance.div(expPart));
- Update:


            int128 result = xBalance * (ABDKMath64x64.ONE - ABDKMath64x64.divu(ABDKMath64x64.ONE, expPart));

## [G-07] 

The visibility modifier public was not necessary for the BASE_FEE constant, as it's not being accessed externally. Changing it to constant only makes it internally accessible and can save a tiny amount of gas during deployment.

- Original Code:


        uint256 public constant BASE_FEE = 800;
- update


          uint256 constant BASE_FEE = 800;
## [G-08]

In the calculateM function, the division operation was called on ABDKMath64x64, which is a fixed-point math library. However, for integer division, Solidity provides the .divu() function directly on the int128 type. Using the native .divu() function can be more efficient than calling the library for division.
- Original Code:


        function calculateM(int128 xAmount, int128 xBalance, int128 yBalance) internal pure returns (int128) {
           return ABDKMath64x64.divu(xAmount.mul(yBalance), xBalance);
       }

- Update


        function calculateM(int128 xAmount, int128 xBalance, int128 yBalance) internal pure returns (int128) {
            return xAmount.mul(yBalance).divu(xBalance);
        }

## [G-09]

Remove the unnecessary usage of the ABDKMath64x64.divu() function for calculating the maximum price ratio. Since py_init, px_init, py_final, and px_final are all of type int128, using native subtraction and division operators is more efficient than calling a library function.

- Original Code:


        require(
            py_init.div(py_init.sub(px_init)) <= ABDKMath64x64.divu(uint256(MAX_PRICE_RATIO), 1) &&
            py_final.div(py_final.sub(px_final)) <= ABDKMath64x64.divu(uint256(MAX_PRICE_RATIO), 1),
            "MaximumAllowedPriceRatioExceeded"
        );

- Update


        require(
            py_init.div(py_init - px_init) <= MAX_PRICE_RATIO &&
            py_final.div(py_final - px_final) <= MAX_PRICE_RATIO,
            "MaximumAllowedPriceRatioExceeded"
        );

