# TvmSlice

`TvmSlice` represents _TVM cell slice_ ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - 1.1.3). The compiler defines the following operators and functions to work with this type:

Comparison operators: `<=`, `<`, `==`, `!=`, `>=`, `>` (evaluate to bool).

Note: only data bits from the root cells are compared. References are ignored.

`TvmSlice` can be converted to `bytes`. It costs at least 500 gas units.

## empty()

```solidity
<TvmSlice>.empty() returns (bool);
```

Checks whether the `TvmSlice` is empty (i.e., contains no data bits and no cell references).

## size()

```solidity
<TvmSlice>.size() returns (uint16 /*bits*/, uint8 /*refs*/);
```

Returns the number of data bits and references in the `TvmSlice`.

## bits()

```solidity
<TvmSlice>.bits() returns (uint16);
```

Returns the number of data bits in the `TvmSlice`.

## refs()

```solidity
<TvmSlice>.refs() returns (uint8);
```

Returns the number of references in the `TvmSlice`.

## dataSize()

```solidity
<TvmSlice>.dataSize(uint n) returns (uint /*cells*/, uint /*bits*/, uint /*refs*/);
```

Returns the number of distinct cells, data bits in the distinct cells and cell references in the distinct cells. If number of the distinct cells exceeds `n+1` then a cell overflow [exception](../../troubleshooting/tvm-exception-codes.md) is thrown. Note that the returned `count of distinct cells` does not take into account the cell that contains the slice itself. This function is a wrapper for `SDATASIZE` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

## dataSizeQ()

```solidity
<TvmSlice>.dataSizeQ(uint n) returns (optional(uint /*cells*/, uint /*bits*/, uint /*refs*/));
```

Returns the number of distinct cells, data bits in the distinct cells and cell references in the distinct cells. If number of the distinct cells exceeds `n+1` then this function returns an `optional` that has no value. Note that the returned `count of distinct cells` does not take into account the cell that contains the slice itself. This function is a wrapper for `SDATASIZEQ` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

## depth()

```solidity
<TvmSlice>.depth() returns (uint16);
```

Returns the depth of `TvmSlice`. If the `TvmSlice` has no references, then 0 is returned, otherwise function result is one plus the maximum of depths of the cells referred to from the slice.

## hasNBits(), hasNRefs() and hasNBitsAndRefs()

```solidity
<TvmSlice>.hasNBits(uint16 bits) returns (bool);
<TvmSlice>.hasNRefs(uint8 refs) returns (bool);
<TvmSlice>.hasNBitsAndRefs(uint16 bits, uint8 refs) returns (bool);
```

Checks whether the `TvmSlice` contains the specified amount of data bits and references.

## compare()

```solidity
<TvmSlice>.compare(TvmSlice other) returns (int8);
```

Lexicographically compares the `slice` and `other` data bits of the root slices and returns result as an integer:

<table><thead><tr><th width="100" align="center">Value</th><th width="188">Meaning</th></tr></thead><tbody><tr><td align="center">1</td><td><code>slice</code> > <code>other</code></td></tr><tr><td align="center">0</td><td><code>slice</code> == <code>other</code></td></tr><tr><td align="center">-1</td><td><code>slice</code> &#x3C; <code>other</code></td></tr></tbody></table>

## ➖➖➖

## TvmSlice load primitives

All functions below modify the `TvmSlice` object they were called for and all of them work
consistently. It means that if user wants to load second ref from the slice, he should
load the first one with [loadRef()](tvmslice.md#loadref), [loadRefAsSlice()](tvmslice.md#loadrefasslice)
or just skip it with [skip()](tvmslice.md#skip) and then load the ref he needs.
The same rule is applied to data bits. To load bits from 2 to 10 positions, user should load
or skip first two bits.

### decode()

```solidity
<TvmSlice>.decode(TypeA, TypeB, ...) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Sequentially decodes values of the specified types from the `TvmSlice`.
Supported types: `uintN`, `intN`, `bytesN`, `bool`, `ufixedMxN`, `fixedMxN`, `address`, `contract`,
`TvmCell`, `bytes`, `string`, `mapping`, `ExtraCurrencyCollection`, `array`, `optional` and 
`struct`.  Example:

```solidity
TvmSlice slice = ...;
(uint8 a, uint16 b) = slice.decode(uint8, uint16);
(uint16 num0, uint32 num1, address addr) = slice.decode(uint16, uint32, address);
```

{% hint style="info" %}
In version 0.70.0 of the compiler, this method was renamed. For detailed information, please refer to the corresponding version of the documentation.
{% endhint %}

### decodeQ()

```solidity
<TvmSlice>.decodeQ(TypeA, TypeB, ...) returns (optional(TypeA, TypeB, ...));
```

Sequentially decodes values of the specified types from the `TvmSlice` 
if the `TvmSlice` holds sufficient data for all specified types. Otherwise, returns `null`.

Supported types: `uintN`, `intN`, `bytesN`, `bool`, `ufixedMxN`, `fixedMxN`, `address`, `contract`,
`TvmCell`, `bytes`, `string`, `mapping`, `ExtraCurrencyCollection`, and `array`.

```solidity
TvmSlice slice = ...;
optional(uint) a = slice.decodeQ(uint);
optional(uint8, uint16) b = slice.decodeQ(uint8, uint16);
```

See also: [\<TvmBuilder\>.store()](#tvmbuilderstore).

See also: [\<TvmBuilder>.store()](tvmbuilder.md#store).

{% hint style="info" %}
In version 0.70.0 of the compiler, this method was renamed. For detailed information, please refer to the corresponding version of the documentation.
{% endhint %}

### loadRef()

```solidity
<TvmSlice>.loadRef() returns (TvmCell);
```

Loads a cell from the `TvmSlice` reference.

### loadRefAsSlice()

```solidity
<TvmSlice>.loadRefAsSlice() returns (TvmSlice);
```

Loads a cell from the `TvmSlice` reference and converts it into a `TvmSlice`.

### loadSigned()

```solidity
<TvmSlice>.loadSigned(uint16 bitSize) returns (int);
```

Loads a signed integer with the given **bitSize** from the `TvmSlice`.

### loadUnsigned()

```solidity
<TvmSlice>.loadUnsigned(uint16 bitSize) returns (uint);
```

Loads an unsigned integer with the given **bitSize** from the `TvmSlice`.

### loadTons()

```solidity
<TvmSlice>.loadTons() returns (uint128);
```

Loads (deserializes) **VarUInteger 16** and returns an unsigned 128-bit integer. See [TL-B scheme][3].

### loadSlice()

```solidity
<TvmSlice>.loadSlice(uint length) returns (TvmSlice);
<TvmSlice>.loadSlice(uint length, uint refs) returns (TvmSlice);
```

Loads the first `length` bits and `refs` references from the `TvmSlice` into a separate `TvmSlice`.

### decodeFunctionParams()

```solidity
// Decode parameters of the public/external function without "responsible" attribute
<TvmSlice>.decodeFunctionParams(functionName) returns (TypeA /*a*/, TypeB /*b*/, ...);

// Decode parameters of the public/external function with "responsible" attribute
<TvmSlice>.decodeFunctionParams(functionName) returns (uint32 callbackFunctionId, TypeA /*a*/, TypeB /*b*/, ...);

// Decode constructor parameters
<TvmSlice>.decodeFunctionParams(ContractName) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Decodes parameters of the function or constructor (if contract type is provided). This function is usually used in
**[onBounce](../special-contract-functions/onbounce.md)** function.

See example of how to use **onBounce** function:

* [onBounceHandler](https://github.com/tonlabs/samples/blob/master/solidity/16_onBounceHandler.sol)

### decodeStateVars()

```solidity
<TvmSlice>.decodeStateVars(ContractName) returns (uint256 /*pubkey*/, uint64 /*timestamp*/, bool /*constructorFlag*/, Type1 /*var1*/, Type2 /*var2*/, ...);
```

Decode state variables from `slice` that is obtained from the field `data` of `stateInit`

Example:

```
contract A {
	uint a = 111;
	uint b = 22;
	uint c = 3;
	uint d = 44;
	address e = address(12);
	address f;
}

contract B {
	function f(TvmCell data) public pure {
		TvmSlice s = data.toSlice();
		(uint256 pubkey, uint64 timestamp, bool flag,
			uint a, uint b, uint c, uint d, address e, address f) = s.decodeStateVars(A);
			
		// pubkey - pubkey of the contract A
		// timestamp - timestamp that used for replay protection
		// flag - always equals to true
		// a == 111
		// b == 22
		// c == 3
		// d == 44
		// e == address(12)
		// f == address(0)
		// s.empty()
	}
}
```

### skip()

```solidity
<TvmSlice>.skip(uint length);
<TvmSlice>.skip(uint length, uint refs);
```

Skips the first `length` bits and `refs` references from the `TvmSlice`.

{% hint style="warning" %}
In version 0.68.0, new functions were added to this method.
For more details, please refer to the corresponding version of the documentation.
{% endhint %}