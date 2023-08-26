QA - 01
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L335-L336
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L377-L378
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L413-L414
```solidity
// amount cannot be less than 0
        require(result > 0);
```
The comment is wrong. The correct comment should be "Amount cannot be less than or equal to 0"

QA - 02
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L251
```solidity
	 if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
        
```

According to the documentation: `MaximumAllowedPriceExceeded` - The y_init/final price cannot be more than 10^8. Here the contract checks for more than or equal.



QA - 03
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L252
```solidity
	if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();

```

According to the documentation: `MinimumAllowedPriceExceeded` - The x_init/final price cannot be less than 10^-8. Here the contract checks for less than or equal.

QA - 04
https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L255-L256
```solidity
	if (py_init <= px_init) revert InvalidPrice();
        if (py_final <= px_final) revert InvalidPrice();

```

According to the documentation: `InvalidPrice` - The x_init/final price cannot be less than y_init/final. Here the contract checks for less than or equal.
