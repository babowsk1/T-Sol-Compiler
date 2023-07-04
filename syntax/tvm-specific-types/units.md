# Units

A literal number can take a suffix to specify a sub-denomination of EVER currency, where numbers without a postfix are assumed to be nano EVERs.

```solidity
uint a0 = 1 nano;        // a0 == 1 == 1e-9 ever
uint a1 = 1 nanoever;    // a1 == 1 == 1e-9 ever
uint a3 = 1 ever;        // a3 == 1 000 000 000 (1e9)
uint a4 = 1 Ever;        // a4 == 1 000 000 000 (1e9)
uint a5 = 1 micro;       // a5 == 1 000 == 1e-6 ever
uint a6 = 1 microever;   // a6 == 1 000 == 1e-6 ever
uint a7 = 1 milli;       // a7 == 1 000 000 == 1e-3 ever
uint a8 = 1 milliever;   // a8 == 1 000 000 == 1e-3 ever
uint a9 = 1 kiloever;    // a9 == 1 000 000 000 000 (1e12) == 1e3 ever
uint a10 = 1 kEver;      // a10 == 1 000 000 000 000 (1e12) == 1e3 ever
uint a11 = 1 megaever;   // a11 == 1 000 000 000 000 000 (1e15) == 1e6 ever
uint a12 = 1 MEver;      // a12 == 1 000 000 000 000 000 (1e15) == 1e6 ever
uint a13 = 1 gigaever;   // a13 == 1 000 000 000 000 000 000 (1e18) == 1e9 ever
uint a14 = 1 GEver;      // a14 == 1 000 000 000 000 000 000 (1e18) == 1e9 ever
```

## Deprecated naming

The compiler supports the legacy naming of units. However, their usage is deprecated and may become incompatible in future compiler versions.

```solidity
uint a0 = 1 nano;        // a0 == 1
uint a1 = 1 nanoton;     // a1 == 1
uint a2 = 1 nTon;        // a2 == 1
uint a3 = 1 ton;         // a3 == 1 000 000 000 (1e9)
uint a4 = 1 Ton;         // a4 == 1 000 000 000 (1e9)
uint a5 = 1 micro;       // a5 == 1 000
uint a6 = 1 microton;    // a6 == 1 000
uint a7 = 1 milli;       // a7 == 1 000 000
uint a8 = 1 milliton;    // a8 == 1 000 000
uint a9 = 1 kiloton;     // a9 == 1 000 000 000 000 (1e12)
uint a10 = 1 kTon;       // a10 == 1 000 000 000 000 (1e12)
uint a11 = 1 megaton;    // a11 == 1 000 000 000 000 000 (1e15)
uint a12 = 1 MTon;       // a12 == 1 000 000 000 000 000 (1e15)
uint a13 = 1 gigaton;    // a13 == 1 000 000 000 000 000 000 (1e18)
uint a14 = 1 GTon;       // a14 == 1 000 000 000 000 000 000 (1e18)
```
