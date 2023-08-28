# [L-01] Fuzz testing uses hard-coded `init` and `final` values

## Description
Fuzz testing is meant to automatically explore the contract attack surface. To maximize the value from fuzz testing, all relevant variables should be dynamic.

In the `setUp()` method of `EvolvingProteusProperties.t.sol`, we see that the init and final values are all hardcoded. (There are some additional scenarios, but they are commented out.)

```
src/test/EvolvingProteusProperties.t.sol:52
52:     function setUp() public {
...
84:        // x & y both increase
85:        py_init_val = 6951000000000000;
86:        px_init_val = 69510000000000;
87:        py_final_val = 695100000000000000;
88:        px_final_val = 695100000000000;
...
98:        DUT = new EvolvingInstrumentedProteus(py_init, px_init, py_final, px_final, duration);
```

## Recommendation
Either uncomment the other scenarios, or make `py_init_val`, `px_init_val`, `py_final_val`, `px_final_val` dynamic parameters.

# [L-02] Lack of Evolving Proteus documentation is error prone

## Description
While there is documentation for The Ocean and Static Proteus, there is minimal documentation for Evolving Proteus.

Effectively, the only documentation is the `readme.md` and the `EvolvingProteus.sol` contract.

This lack of documentation makes the protocol difficult to review and it is possible that certain vulnerabilities will be missed.

## Recommendation
At a minimum:
1. A discussion of the 2 primary use cases (Dutch auctions and dynamic pools)
2. Graphs demonstrating how Evolving Proteus improves on the state of the art
3. Documentation about the various configuration parameters and the effects thereof
4. Documentation about the various errors that can occur and the significance of those errors
5. Documentation about how fees work

# [L-03] Lack of visualizable test output is error prone

## Description
Assessing the performance of `EvolvingProteus` using purely numerical test outputs is very challenging.

Given that the fundamental nature of the solution is to produce an evolving bonding curve, an improvement would be to produce visualizations of the evolving bonding curves.

## Recommendation
Implement a `forge script` to run `EvolvingProteus` through various representative uses cases and scenarios. Output curve point data into a format that can be used to graph the curves. Produce these graphs as part of the build to visualize test output for additional implementation confidence.

# [L-04] `ABDKMath64x64` dependency has no unit tests

## Description
`EvolvingProteus` makes extensive use of the `ABDKMath64x64` library. This library appears to have no unit tests.

While the `echidna` and `medusa` communities have explored fuzz testing with the library, this is not equivalent to a reliable test suite.

## Recommendation
Either consider adding unit tests for `ABDKMath64x64` that covers all the major operations used in `EvolvingProteus`, or contribute the tests directly to the `ABDKMath64x64` project.