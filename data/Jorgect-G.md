# GAS OPTMIZATION.

## [G-01] Save one multiplication in _getUtility function

The contract can applied [Common Factor of a polynomial  ](  https://www.khanacademy.org/math/algebra2/x2ec2f6f830c9fb89:poly-factor/x2ec2f6f830c9fb89:common-factor/a/taking-common-factors) to save gas, see the next example to undertand what is the common Factor of a polynomial:

```
a*(b+c) = a*b + a*c
```

See the implementation:

``` 
function _getUtility(int256 x, int256 y) internal view returns (int256 utility) {
        ...

        int256 r0 = (-bQuad * MULTIPLIER + disc * MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER); 
        int256 r1 = (-bQuad * MULTIPLIER - disc * MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER); 

       ...
    }
```
https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L717C9-L718C93


We can implement the common factor of the above code to save gas avoiding adiccionals multiplication:

``` 
function _getUtility(int256 x, int256 y) internal view returns (int256 utility) {
        ...

        int256 r0 = ((-bQuad  + disc) * MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER); 
        int256 r1 = ((-bQuad  - disc) * MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER); 

       ...
    }
```








