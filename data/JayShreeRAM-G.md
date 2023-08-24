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
