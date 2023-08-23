## QA
---

### natSpec missing [1]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol

```solidity
10:  struct Config {
44:library LibConfig {

// @return missing
59:    function newConfig(
81:    function elapsed(Config storage self) public view returns (uint256) {
89:    function t(Config storage self) public view returns (int128) {
97:    function p_min(Config storage self) public view returns (int128) {
106:    function p_max(Config storage self) public view returns (int128) {
115:    function a(Config storage self) public view returns (int128) {
123:    function b(Config storage self) public view returns (int128) {
132:    function duration(Config storage self) public view returns (uint256) {

137:  contract EvolvingProteus is ILiquidityPoolImplementation {
222:    error AmountError();
223:    error BalanceError(int256 x, int256 y);
224:    error BoundaryError(int256 x, int256 y);
225:    error CurveError(int256 errorValue); 
226:    error InvalidPrice();
227:    error MinimumAllowedPriceExceeded();
228:    error MaximumAllowedPriceExceeded();
229:    error MaximumAllowedPriceRatioExceeded();

// @param and @return missing
272:    function swapGivenInputAmount(
312:    function swapGivenOutputAmount(
353:    function depositGivenInputAmount(
389:    function depositGivenOutputAmount(
426:    function withdrawGivenOutputAmount(
463:    function withdrawGivenInputAmount(
506:    function _swap(
563:    function _reserveTokenSpecified(
607:    function _lpTokenSpecified(
652:    function _getUtilityFinalLp(
739:    function _getPointGivenXandUtility(
770:    function _getPointGivenYandUtility(
824:    function _applyFeeByRounding(int256 amount, bool feeUp)

// @param missing
796:    function _checkAmountWithBalance(uint256 balance, uint256 amount)
809:    function _checkBalances(int256 x, int256 y) private pure {

// @return missing
681:    function _findFinalPoint(
701:    function _getUtility(
```

---

### Initialized variables [2]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol

```solidity
211:    bool constant FEE_DOWN = false;
```