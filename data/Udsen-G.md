## 1. CAN USE `memory` STRUCTS INSTEAD OF `storage` STRUCTS TO SAVE GAS

In the `EvolvingProteus.LibConfig` library there are multiple utility functions which are used to calculate the `elapsed time`, `duration`, `p_min value`, `p_max` value etc... The `Config` is passed in as a storage struct (input parameter) to these functions. But these functions do not perform state changes and they are `view` functions. Hence `memory` structs can be passed in inplace of `storage` structs which will save on multiple `SLOAD` operations thus saving on gas.

To resolve this issue, it is recommended to change the `storage` keyword to `memory` for the input parameter in the functions `elapsed, t, p_min, p_max, a, b, and duration`. This will help to save gas as memory is cheaper to use than storage. Here is how you can do it:

```solidity
function elapsed(Config memory self) public view returns (uint256) {
    return block.timestamp - self.t_init;
}

function t(Config memory self) public view returns (int128) {
    return elapsed(self).divu(duration(self));
}

function p_min(Config memory self) public view returns (int128) {
    if (t(self) > ABDK_ONE) return self.px_final;
    else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
}

function p_max(Config memory self) public view returns (int128) {
    if (t(self) > ABDK_ONE) return self.py_final;
    else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
}

function a(Config memory self) public view returns (int128) {
    return (p_max(self).inv()).sqrt();
}

function b(Config memory self) public view returns (int128) {
    return p_min(self).sqrt();
}

function duration(Config memory self) public view returns (uint256) {
    return self.t_final - self.t_init;
}
```

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L81-L134

## 2. REDUNDANT ARITHMETIC OPERATIONS CAN BE REDUCED TO SAVE GAS

The `EvolvingProteus._getUtility` function is used to calculate the `utility` value of the pool at the time of the function call. The `utilitiy` is calculated using a quadratic formula which is shown below:

    k(ab - 1)u**2 + (ay + bx)u + xy/k = 0

Above quadratic equation when solved gives us two solutions as implemented in the `_getUtility` function.

        int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
        int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

As it is seen above the constant `MULTIPLIER` is used to multiply with both `-bQuad` and `disc` values in both `r0` and `r1` calculations. But the above equations can be modified as shown below to limit the number of times the `MULTIPLIER` will be used to multiply to only one instead of two, as shown below:

        int256 r0 = (-bQuad + disc)*MULTIPLIER / aQuad.mul(two).muli(MULTIPLIER);
        int256 r1 = (-bQuad - disc)*MULTIPLIER / aQuad.mul(two).muli(MULTIPLIER);

The above modified code will save gas since one multiplication operation is reduced for each `r0` and `r1` calculations.

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L717-L718