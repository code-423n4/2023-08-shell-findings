int128 a = config.a(); 
 int128 b = config.b(); 
are called 3 times in `_getUtility`,`_getPointGivenYandUtility`, `_getPointGivenXandUtility` functions

int256 a_convert = a.muli(MULTIPLIER);
int256 b_convert = b.muli(MULTIPLIER);
are called 2 times inside function `_getPointGivenYandUtility` and `_getPointGivenXandUtility` functions.

As they do the same calculations and return the same value,caching their values and reusing them is recommended to reduce gas costs
