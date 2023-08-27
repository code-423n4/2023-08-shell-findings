.On L295 there is a commment that says "// amount cannot be less than 0" but in the next line of the code,there is a require statement `require(result < 0)`.
And In the same function that there is a negative sign before the result variable.I am assuming that the result variable will store a positive value(Because,later they are 
they are casted to `outputAmount` as a uint negative variable).Due to that
the function will revert because of the requrie statement.
Same kind of scenario can be seen on L450 and L487

suggesting to set the require statements according the comments or to set the comments according the require statements as comments gives us critical insights of the code and wrong comments can create confusions in future updates

Also,there are multiple requrie statements which does not have any revert strings.
Attaching error reason strings along with require statements is recommended to make it easier to understand why a contract call reverted.
