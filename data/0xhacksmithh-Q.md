
### [Low-01]
comment says `newConfig()` Calculates the equation parameters "a" & "b" described above & returns the config instance but it only returns config.

```solidity
 /**
       @notice Calculates the equation parameters "a" & "b" described above & returns the config instance
       @param py_init The initial price at the y axis
       @param px_init The initial price at the x axis
       @param py_final The final price at the y axis
       @param px_final The final price at the x axis
       @param _duration duration over which the curve will evolve
     */
    function newConfig( // @audit-issue comment says Calculates the equation parameters "a" & "b" described above & returns the config instance but it only returns config
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 _duration
    ) public view returns (Config memory) {

        return Config(
            py_init,
            px_init,
            py_final,
            px_final,
            block.timestamp,
            block.timestamp + _duration
        );
    }
```
*Instances(1)*
```
File: EvolvingProteus.sol
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L52
```

### [Low-02] Protocol will incompatible with tokens having less decimal than 18 like USDC 
Some tokens have decimal of 6, In that case to satisfy MIN_BALANCE there should be 10^6 USDC

```solidity
    /*
    When a token has 18 decimals, this is one microtoken
    */ 
    int256 constant MIN_BALANCE = 10**12;
````
*Instances(1)*
```
File: EvolvingProteus.sol
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L151
```
