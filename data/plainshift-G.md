### All state variables can be immutable instead of reading them from storage

During the [`EvolvingProteus` deployment initial configuration is set](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L243-L263).
The duration for which the curve will evolve is usually expected to last anywhere from 12 hours to a couple of days, which is negligible compared to the lifetime of the contract.
In order to calculate the [utility](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L706-L707) and points on the curve given [X](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L739) or
[Y](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L774-L775), each and every time `a` and `b` are read from storage.
`a` and `b` are computed from `p_max` and `p_min` which in turn are computed from the duration of the auction and starting and final prices.
[All of the computed configuration values](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L97-L125) are going to be constant as soon as the auction finishes, so it makes little sense to keep on computing them indefinitely.
[Also the whole Config](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L10-L17) stores the values in storage which is not needed, because there are no setters for these values and they cannot change.
The proposed solution is to get rid of the `Config` struct, make all the variables immutable, and compute the final values for `a` and `b` during deployment, and use those once the auction finishes.

```solidity
int128 immutable py_init;
int128 immutable px_init;
int128 immutable py_final;
int128 immutable px_final;
int128 immutable t_init;
int128 immutable t_final;

// As the auction finishes these values are used for a and b

int128 immutable a_final;
int128 immutable b_final;
```

The whole contract is read-only, so it shouldn't have any state variables.