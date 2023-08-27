### QA Report Issues List

- [x] **Low 01** → ` result ` require  block  is inconsistent with NatSpec comment
- [x] **Low 02** → `libusb` dependency can be a security risk
- [x] **Low 03** → There is a difference in the formula with the NatSpec comments
- [x] **Low 04** → There may be problems with the use of Arbitrum
- [x] **Non-Critical  05** → Project Upgrade and Stop Scenario should be
- [x] **Non-Critical  06** → Missing Event for  initialize




### [Low-01] ` result ` require  block  is inconsistent with NatSpec comment

` result ` require block and Natspec comment are inconsistent, correct it if the error is caused by the comment here, if the formula is incorrect, this is an important issue

```solidity
src\proteus\EvolvingProteus.sol:
  271       */
  272:     function swapGivenInputAmount(
  273:         uint256 xBalance,
  274:         uint256 yBalance,
  275:         uint256 inputAmount,
  276:         SpecifiedToken inputToken
  277:     ) external view returns (uint256 outputAmount) {
  278:         // input amount validations against the current balance
  279:         require(
  280:             inputAmount < INT_MAX && xBalance < INT_MAX && yBalance < INT_MAX
  281:         );
  282: 
  283:         _checkAmountWithBalance(
  284:             (inputToken == SpecifiedToken.X) ? xBalance : yBalance,
  285:             inputAmount
  286:         );
  287: 
  288:         int256 result = _swap(
  289:             FEE_DOWN,
  290:             int256(inputAmount),
  291:             int256(xBalance),
  292:             int256(yBalance),
  293:             inputToken
  294:         );
  295:         // amount cannot be less than 0
  296:         require(result < 0);		 // @audit => NatSpec comment and require is inconsistent
  297: 
  298:         // output amount validations against the current balance
  300:         outputAmount = uint256(-result);
  301:         _checkAmountWithBalance(
  302:             (inputToken == SpecifiedToken.X) ? yBalance : xBalance,
  303:             outputAmount
  304:         );
  305:     }
```


### [Low-02] `libusb` dependency can be a security risk

When the `libusb` dependency is checked from the npm site, it does not provide any references, links to github or versions, creates a malicious dependency security risk, I recommend checking it and documenting what it serves

https://www.npmjs.com/package/libusb?activeTab=readme

```solidity

package.json:
  26:   },
  27:   "dependencies": {
  28:     "libusb": "^0.0.0-pre"
  29:   }
  30  }

```


### [Low-03] There is a difference in the formula with the NatSpec comments

In the NatSpec comment, x price should be less than y price. But in the formula there is also in the equation

```solidity
src\proteus\EvolvingProteus.sol:
  253  
  254:         // at all times x price should be less than y price
  255:         if (py_init <= px_init) revert InvalidPrice();
  256:         if (py_final <= px_final) revert InvalidPrice();
```


### [Low-04] There may be problems with the use of Arbitrum

According to the scope information of the project, it is stated that it  be used  Layer2 Arbitrum 

```js
README.md:
Which blockchains will this code be deployed to, and are considered in scope for this audit?: Arbitrum

```

The problem with this is that Arbitrum is [Compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with these versions will result in a non-functional or potentially damaged version that does not behave as expected.

Although the project was compiled with 0.8.10, in a possible live deploy time, it may be compiled with a different compile version and this may cause it to not work on the Arbitrum network, this risk should be documented and included in the project's deploy checklist



### [Non Critical-05] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".


https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol



### [Non Critical-06] Missing Event for  initialize

Use event-emit when setting the startup items in the project's constructor.

This is recommended so that Web3 Dapps and Analysis tools can monitor the set data and the project.

```solidity
src\proteus\EvolvingProteus.sol:
  242      */
  243:     constructor(
  244:         int128 py_init,
  245:         int128 px_init,
  246:         int128 py_final,
  247:         int128 px_final,
  248:         uint256 duration
  249:     ) { 
  250:         // price value checks
  251:         if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) revert MaximumAllowedPriceExceeded();
  252:         if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) revert MinimumAllowedPriceExceeded();
  253: 
  254:         // at all times x price should be less than y price
  255:         if (py_init <= px_init) revert InvalidPrice();
  256:         if (py_final <= px_final) revert InvalidPrice();
  257: 
  258:         // max. price ratio check
  259:         if (py_init.div(py_init.sub(px_init)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
  260:         if (py_final.div(py_final.sub(px_final)) > ABDKMath64x64.divu(uint(MAX_PRICE_RATIO), 1)) revert MaximumAllowedPriceRatioExceeded();
  261: 
  262:         config = LibConfig.newConfig(py_init, px_init, py_final, px_final, duration);
  263:       }



```
