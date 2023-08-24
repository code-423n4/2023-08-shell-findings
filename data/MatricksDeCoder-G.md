### GAS-1 - Library Embedded vs Linked - Public vs Internal Library Functions 

Make LibConfig library [from lines 44-135 functions internal instead of public](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L44C9-L44C18) e.g functions 

A library with public functions results in a linked library whereas library with internal functions only  is embedded in the contract consuming it. The LibConfig results in a library deployed on its own address and then utilizes 'delegatecall' for function calls. 

**Impact**-> Gas Savings: The usage of delegatecall by linked library implies extra costs. Therefore although linked libraries save on deployment costs by being reused from address without need for delegatecall, they make contract operations more costly due to delegatecall. It is important to put users first and save on gas costs for all operations in smart contracts. 

**Recommendation** -> Make all functions in library internal so that it is embedded instead of linked. Embedded will increase deployment costs but that is better than increasing user costs.

### GAS-2 - Solidity Gas Optimizor settings 

The deployment settings for Solidity Optimizer settings are not explicitly set as a policy or in the configurations we only see in foundry.toml runs=1000 and have no idea or documentation how this was chosen and if it was the best for gas efficiencies or if it will be the one carried through to deployments. It appears there is no policy on this crucial issue 

**Impact**-> This can result in using optimizer settings that are no best for reducing gas costs for the users. The more the runs e.g 10,000 the cheaper the gas costs for functions for users. The smaller the runs, the cheaper the deployment costs. It's essential to put users first in order to attract more and better usage of protocol.

**Recommendation** -> It is recommended to test various solidity optimizer runs(document this optimization process) until the optimal value that reduces the gas costs of functions e.g external functions affecting especially users. Projects need to put users first and help reduce their costs of interacting with projects.

### GAS-3 - Combine ternary operators into single one 

[EvolvingProteus.sol lines 829 -830](https://github.com/code-423n4/2023-08-shell/blob/c61cf0e01bada04c3d6055acb81f61955ed600aa/src/proteus/EvolvingProteus.sol#L829C9-L830C79)
```
bool negative = amount < 0 ? true : false;
uint256 absoluteValue = negative ? uint256(-amount) : uint256(amount);
```
vs 
```
uint256 absoluteValue = amount < 0 ? uint256(-amount) : uint256(amount);
```

**Impact**-> The above results in wastage of gas by performing more computations and operations. 
The first one will have to check if amount is less than zero, if true save true in memory else save false in memory as negative. Then it proceeds to get negative from memory and query if negative value is true [this is akin to doing boolean comparison], if true its saves the negated value in memory as absoluteValue or saves the actual value.

Whereas the combined one that checks if amount less than zero and proceeds to negate or use actual value and store it in memory 

Running a simple function to check approximate gas differences, in Remix produced about 30 gas savings 
```
contract Empty { 
    function test(int _amount) external pure returns(uint256) { //743 gas 
        uint256 absoluteValue = _amount < 0 ? uint256(-_amount) : uint256(_amount);
        return absoluteValue ;
    }
   
}
// The original cost about 774 
contract Empty { 
    function test(int _amount) external pure returns(uint256) { //774 gas 
        bool negative = _amount < 0 ? true : false;
        uint256 absoluteValue = negative ? uint256(-_amount) : uint256(_amount);
        return absoluteValue ;
    }
   
}

```

**Recommendation** -> Combine as in below 
```
uint256 absoluteValue = amount < 0 ? uint256(-amount) : uint256(amount);
```





