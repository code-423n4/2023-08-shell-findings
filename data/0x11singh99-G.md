### Gas Optimizations List

| Number | Optimization Details                                                                                                     | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Struct can be packed into fewer storage slots                   |     1    |
| [G-02] | Multiplication/Division by powers of two should use bit shifting |     2     |
| [G-03] | Cache complex calculated value rather than recalculating them saves gas way                                                                                |     1     |
| [G-04] | Use assembly for math (add, sub, mul, div) gas                                                                      |     25     |

Total 4 issues

## [G-01] Struct can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

### Reduce `uint` type for *`t_init`* and *`t_final`* to store time and pack into a single storage slot to save 1 SLOT (~2000 gas)

```solidity
File : src/proteus/EvolvingProteus.sol

10:  struct Config {
11:    int128 py_init;
12:    int128 px_init;
13:    int128 py_final;
14:    int128 px_final;
15:    uint256 t_init;
16:    uint256 t_final;
17:   }
```
[10-17](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L10C1-L17C2)

```diff
File : src/proteus/EvolvingProteus.sol

struct Config {
    int128 py_init;
    int128 px_init;
    int128 py_final;
    int128 px_final;
-   uint256 t_init;
-   uint256 t_final;
+   uint128 t_init;
+   uint128 t_final;
}

```
## [G‑02] Multiplication/Division by powers of two should use bit shifting.

`<x> * 2` is the same as `<x> << 1`. While the compiler uses the SHL opcode to accomplish both, the version that uses multiplication incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the multiplication by two. 

`<x> / 2` is the same as `<x> >> 1`. While the compiler uses the SHR opcode to accomplish both, the version that uses division incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.

Same applied for other powers of 2.

*Note: These are instances missed by the Bot Race.*

```solidity
File : src/proteus/EvolvingProteus.sol

709: int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));

716: int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
```
[709](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L709C9-L709C87),[716](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L716C9-L716C86)

```diff
File : src/proteus/EvolvingProteus.sol

- 709:  int128 two = ABDKMath64x64.divu(uint256(2 * MULTIPLIER), uint256(MULTIPLIER));
+       int128 two = ABDKMath64x64.divu(uint256(MULTIPLIER << 1), uint256(MULTIPLIER));
        int128 one = ABDKMath64x64.divu(uint256(MULTIPLIER), uint256(MULTIPLIER));

        int128 aQuad = (a.mul(b).sub(one));
        int256 bQuad = (a.muli(y) + b.muli(x));
        int256 cQuad = x * y;

- 716:  int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad)*4)))));
+       int256 disc = int256(Math.sqrt(uint256((bQuad**2 - (aQuad.muli(cQuad) << 2)))));

```
## [G-03] Cache complex calculated value rather than recalculating them saves gas

 Recalculating the same values multiple times would result in higher gas costs due to redundant computations. Caching allows you to pay the gas cost once for the initial calculation and then reuse the result multiple times without incurring additional gas costs.

### Cache `aQuad.mul(two).muli(MULTIPLIER)` rather than recalculating. 

```solidity
File : src/proteus/EvolvingProteus.sol

717:   int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
718:   int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

```
[717-718](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L717C8-L718C93)

```diff
File : src/proteus/EvolvingProteus.sol

+   aQuadTimesTwoTimesMultiplier = aQuad.mul(two).muli(MULTIPLIER);
-   int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);   
-   int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
+   int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuadTimesTwoTimesMultiplier;  
+   int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuadTimesTwoTimesMultiplier;

```
## [G-04] Use assembly for math (add, sub, mul, div)

When we say "use assembly for math operations," we mean that when performing basic math operations in a smart contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in math functions. This is because Solidity's built-in math functions require additional gas consumption to perform the operation.

```solidity
File : src/proteus/EvolvingProteus.sol


714:  int256 cQuad = x * y;

717:  int256 r0 = (-bQuad*MULTIPLIER + disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);
718:  int256 r1 = (-bQuad*MULTIPLIER - disc*MULTIPLIER) / aQuad.mul(two).muli(MULTIPLIER);

750:  int256 f_0 = ((( x0  * MULTIPLIER ) / utility) + a_convert);
751:  int256 f_1 = ((MULTIPLIER * MULTIPLIER / f_0) -  b_convert);
752:  int256 f_2 = (f_1 * utility) / MULTIPLIER;

781:  int256 f_0 = (( y0  * MULTIPLIER ) / utility) + b_convert;
782:  int256 f_1 = ( ((MULTIPLIER)*(MULTIPLIER) / f_0) - a_convert );
783:  int256 f_2 = (f_1 * utility) / (MULTIPLIER);

```
[714](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L714) , [717-718](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L717C9-L718C93) , [750-752](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L750C9-L752C52) , [781-783](https://github.com/code-423n4/2023-08-shell/blob/main/src/proteus/EvolvingProteus.sol#L781C9-L783C54)
