I found there are multiple call of `t(self)`. so I use tValue and using it in function.


EvolvingProteus.sol
```
    /**
       @notice The minimum price (at the x asymptote) at the current block
       @param self config instance
    */
    // function p_min(Config storage self) public view returns (int128) {
    //     if (t(self) > ABDK_ONE) return self.px_final;
    //     else return self.px_init.mul(ABDK_ONE.sub(t(self))).add(self.px_final.mul(t(self)));
    // }
    function p_min(Config storage self) public view returns (int128) {
        int128 tValue = t(self);
        if (tValue > ABDK_ONE) return self.px_final;
        else return self.px_init.mul(ABDK_ONE.sub(tValue)).add(self.px_final.mul(tValue));
    }

    /**
       @notice The maximum price (at the y asymptote) at the current block
       @param self config instance
    */
    // function p_max(Config storage self) public view returns (int128) {
    //     if (t(self) > ABDK_ONE) return self.py_final;
    //     else return self.py_init.mul(ABDK_ONE.sub(t(self))).add(self.py_final.mul(t(self)));
    // }
    function p_max(Config storage self) public view returns (int128) {
        int128 tValue = t(self);
        if (tValue > ABDK_ONE) return self.py_final;
        else return self.py_init.mul(ABDK_ONE.sub(tValue)).add(self.py_final.mul(tValue));
    }

```


This is original result.
```
$ forge test --gas-report
...
| Function Name                                                                 | min             | avg   | median | max   | # calls |
...
| p_max                                                                         | 4594            | 6830  | 4594   | 15094 | 93      |
| p_min                                                                         | 4696            | 4696  | 4696   | 4696  | 93      |
```

This is after result.
```
$ forge test --gas-report
...
| Function Name                                                                 | min             | avg   | median | max   | # calls |
...
| p_max                                                                         | 2003            | 4799  | 2800   | 13300 | 96      |
| p_min                                                                         | 2088            | 2885  | 2902   | 2902  | 96      |
```