# abi namespace

## encode(), decode()

```solidity
(1)
abi.encode(TypeA a, TypeB b, ...) returns (TvmCell /*cell*/);
(2)
abi.decode(TvmCell cell, (TypeA, TypeB, ...)) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

`abi.encode` creates `cell` from the values. `abi.decode` decodes the `cell` and returns the values. Note: all types must be set in `abi.decode`. Otherwise, `abi.decode` throws an exception.

Example:

```solidity
// pack the values to the cell
TvmCell cell = abi.encode(uint(1), uint(2), uint(3), uint(4));
// unpack the cell
(uint a, uint b, uint c, uint d) = abi.decode(cell, (uint, uint, uint, uint));
// a == 1
// b == 2
// c == 3
// d == 4
```