# TvmCell

`TvmCell` represents a _TVM cell_ object ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - 1.1.3). The compiler defines the following operators and functions to work with this type.

## Operators

* Comparison operators: `==`, `!=` (evaluate to `bool`)

## \<TvmCell>.depth()

```solidity
<TvmCell>.depth() returns(uint16);
```

Returns the depth **d** of the `TvmCell` **c:**&#x20;

* If **c** has no references, then **d** = 0;&#x20;
* Otherwise, **d** equals one plus the maximum of depths of cells referred to from **c**.&#x20;
* If **c** is a `Null` instead of a `TvmCell`, it returns zero.

## \<TvmCell>.dataSize()

```solidity
<TvmCell>.dataSize(uint n) returns (uint /*cells*/, uint /*bits*/, uint /*refs*/);
```

Returns the number of distinct cells, data bits in the distinct cells and cell references in the distinct cells. If the number of the distinct cells exceeds `n+1` , then a cell overflow [exception](tvmcell.md#tvm-exception-codes) is thrown. This function is a wrapper for the `CDATASIZE` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

## \<TvmCell>.dataSizeQ()

```solidity
<TvmCell>.dataSizeQ(uint n) returns (optional(uint /*cells*/, uint /*bits*/, uint /*refs*/));
```

Returns the number of distinct cells, data bits in the distinct cells and cell references in the distinct cells. If number of the distinct cells exceeds `n+1` then this function returns an `optional` that has no value. This function is a wrapper for the `CDATASIZEQ` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

## \<TvmCell>.toSlice()

```solidity
<TvmCell>.toSlice() returns (TvmSlice);
```

Converts a `TvmCell` to `TvmSlice`.
