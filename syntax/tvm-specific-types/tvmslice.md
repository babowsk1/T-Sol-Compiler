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
<TvmSlice>.hasNBits(uint10 bits) returns (bool);
<TvmSlice>.hasNRefs(uint2 refs) returns (bool);
<TvmSlice>.hasNBitsAndRefs(uint10 bits, uint2 refs) returns (bool);
```

Checks whether the `TvmSlice` contains the specified amount of data bits and references.

## compare()

```solidity
<TvmSlice>.compare(TvmSlice other) returns (int2);
```

Lexicographically compares the `slice` and `other` data bits of the root slices and returns result as an integer:

<table><thead><tr><th width="100" align="center">Value</th><th width="188">Meaning</th></tr></thead><tbody><tr><td align="center">1</td><td><code>slice</code> > <code>other</code></td></tr><tr><td align="center">0</td><td><code>slice</code> == <code>other</code></td></tr><tr><td align="center">-1</td><td><code>slice</code> &#x3C; <code>other</code></td></tr></tbody></table>

## ➖➖➖

## TvmSlice load primitives

All `load*` functions below modify the `TvmSlice` object. If you wants to load second reference from the `TvmSlice`, you should load the first one with [\<TvmSlice>.loadRef()](tvmslice.md#loadref) and then load the reference you need. The same rule is applied to data bits. To load bits from 2 to 10 positions, you should load or skip first two bits.

### load()

```solidity
<TvmSlice>.load(TypeA, TypeB, ...) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Sequentially loads values of the specified types from the `TvmSlice`. Supported types: `uintN`, `intN`, `bytesN`, `bool`, `ufixedMxN`, `fixedMxN`, `address`, `contract`, `TvmCell`, `bytes`, `string`, `mapping`, `ExtraCurrencyCollection`, `array`, `optional` and `struct`. Example:

```solidity
TvmSlice slice = ...;
(uint8 a, uint16 b) = slice.load(uint8, uint16);
(uint16 num0, uint32 num1, address addr) = slice.load(uint16, uint32, address);
```

See also: [\<TvmBuilder>.store()](tvmbuilder.md#store). **Note**: if all the argument types can't be loaded from the slice a cell underflow [exception](../../troubleshooting/tvm-exception-codes.md) is thrown.

{% hint style="warning" %}
**`<TvmSlice>.decode()`** has been renamed to **`<TvmSlice>.load()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

### loadQ()

```solidity
<TvmSlice>.loadQ(TypeA, TypeB, ...) returns (optional(TypeA, TypeB, ...));
```

Sequentially decodes values of the specified types from the `TvmSlice` if the `TvmSlice` holds sufficient data for all specified types. Otherwise, returns `null`.

```solidity
TvmSlice slice = ...;
optional(uint) a = slice.loadQ(uint);
optional(uint8, uint16) b = slice.loadQ(uint8, uint16);
```

See also: [\<TvmBuilder>.store()](tvmbuilder.md#store).

{% hint style="warning" %}
**`<TvmSlice>.decodeQ()`** has been renamed to **`<TvmSlice>.loadQ()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
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

### loadInt() and loadIntQ()

```solidity
(1)
<TvmSlice>.loadInt(uint9 bitSize) returns (int);
(2)
<TvmSlice>.loadIntQ(uint9 bitSize) returns (optional(int));
```

(1) Loads a signed integer with the given **bitSize** from the `TvmSlice`.

(2) Loads a signed integer with the given **bitSize** from the `TvmSlice` if `TvmSlice` contains it. Otherwise, returns `null`.

{% hint style="warning" %}
**`<TvmSlice>.loadSigned()`** has been renamed to **`<TvmSlice>.loadInt()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

### loadUint() and loadUintQ()

```solidity
// (1)
<TvmSlice>.loadUint(uint9 bitSize) returns (uint);

// (2)
<TvmSlice>.loadUintQ(uint9 bitSize) returns (optional(uint));
```

(1) Loads an unsigned integer with the given **bitSize** from the `TvmSlice`.

(2) Loads an unsigned integer with the given **bitSize** from the `TvmSlice` if `TvmSlice` contains it. Otherwise, returns `null`.

{% hint style="warning" %}
**`<TvmSlice>.loadUnsigned()`** has been renamed to **`<TvmSlice>.loadUint()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

### Load little-endian integers

```solidity
// (1)
<TvmSlice>.loadIntLE2() returns (int16)
<TvmSlice>.loadIntLE4() returns (int32)
<TvmSlice>.loadIntLE8() returns (int64)
<TvmSlice>.loadUintLE2() returns (uint16)
<TvmSlice>.loadUintLE4() returns (uint32)
<TvmSlice>.loadUintLE8() returns (uint64)

// (2)
<TvmSlice>.loadIntLE4Q() returns (optional(int32))
<TvmSlice>.loadIntLE8Q() returns (optional(int64))
<TvmSlice>.loadUintLE4Q() returns (optional(uint32))
<TvmSlice>.loadUintLE8Q() returns (optional(uint64))
```

(1) Loads the little-endian integer from `TvmSlice`.

(2) Same as (1) but returns `null` if it's impossible to load the integer.

### loadTons()

```solidity
<TvmSlice>.loadTons() returns (uint128);
```

Loads (deserializes) **VarUInteger16** and returns an unsigned 128-bit integer. See \[TL-B scheme]\[3].

### loadSlice() and loadSliceQ()

```solidity
// (1)
<TvmSlice>.loadSlice(uint10 bits) returns (TvmSlice);

// (2)
<TvmSlice>.loadSlice(uint10 bits, uint refs) returns (TvmSlice);

// (3)
<TvmSlice>.loadSliceQ(uint10 bits) returns (optional(TvmSlice));

// (4)
<TvmSlice>.loadSliceQ(uint10 bits, uint2 refs) returns (optional(TvmSlice));
```

(1) Loads the first `bits` bits from `TvmSlice`. (2) Loads the first `bits` bits and `refs` references from `TvmSlice`. (3) and (4) are the same as (1) and (2) but return `optional` type.

### loadFunctionParams()

```solidity
// Loads parameters of the public/external function without "responsible" attribute
<TvmSlice>.loadFunctionParams(functionName) returns (TypeA /*a*/, TypeB /*b*/, ...);

// Loads parameters of the public/external function with "responsible" attribute
<TvmSlice>.loadFunctionParams(functionName) returns (uint32 callbackFunctionId, TypeA /*a*/, TypeB /*b*/, ...);

// Loads constructor parameters
<TvmSlice>.loadFunctionParams(ContractName) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Loads parameters of the function or constructor (if contract type is provided). This function is usually used in the [**onBounce**](../special-contract-functions/onbounce.md) function.

See the example of how to use **onBounce** function:

* [onBounceHandler](https://github.com/tonlabs/samples/blob/master/solidity/16\_onBounceHandler.sol)

{% hint style="warning" %}
**`<TvmSlice>.decodeFunctionParams()`** has been renamed to **`<TvmSlice>.loadFunctionParams()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

### loadStateVars()

```solidity
<TvmSlice>.loadStateVars(ContractName) returns (
    uint256 /*pubkey*/, 
    uint64 /*timestamp*/, 
    bool /*constructorFlag*/, 
    Type1 /*var1*/, Type2 /*var2*/, ...
);
```

Loads state variables from `slice` that is obtained from the field `data` of `stateInit`

Example:

```solidity
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
			uint a, uint b, uint c, uint d, address e, address f) = s.loadStateVars(A);
			
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

{% hint style="warning" %}
**`<TvmSlice>.decodeStateVars()`**has been renamed to **`<TvmSlice>.loadStateVars()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

### skip()

```solidity
<TvmSlice>.skip(uint10 bits);
<TvmSlice>.skip(uint10 bits, uint2 refs);
```

Skips the first `bits` bits and `refs` references from the `TvmSlice`.

### loadZeroes(), loadOnes() and loadSame()

```solidity
// (1)
<TvmSlice>.loadZeroes() returns (uint10 n);

// (2)
<TvmSlice>.loadOnes() returns (uint10 n);

// (3)
<TvmSlice>.loadSame(uint1 value) returns (uint10 n);
```

(1) Returns the count `n` of leading zero bits in `TvmSlice`, and removes these bits from `TvmSlice`.

(2) Returns the count `n` of leading one bits in `TvmSlice`, and removes these bits from `TvmSlice`.

(3) Returns the count `n` of leading bits equal to `0 ≤ value ≤ 1` in `TvmSlice`, and removes these bits from `TvmSlice`.

See also: [\<TvmBuilder>.storeZeroes(), \<TvmBuilder>.storeOnes() and \<TvmBuilder>.storeSame()](tvmbuilder.md#storezeroes-storeones-and-storesame).

## ➖➖➖

## TvmSlice preload primitives

All `preload*` functions below don't modify the `TvmSlice` object.

### preload()

```solidity
<TvmSlice>.preload(TypeA, TypeB, ...) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Same as [\<TvmSlice>.load()](tvmslice.md#tvmsliceload) but doesn't modify `TvmSlice`.

### preloadQ()

```solidity
<TvmSlice>.preloadQ(TypeA, TypeB, ...) returns (optional(TypeA, TypeB, ...));
```

Same as [\<TvmSlice>.loadQ()](tvmslice.md#loadq) but doesn't modify `TvmSlice`.

### preloadRef()

```solidity
// (1)
<TvmSlice>.preloadRef() returns (TvmCell);

// (2)
<TvmSlice>.preloadRef(uint2 index) returns (TvmCell);
```

(1) Returns the first cell reference of `TvmSlice`.

(2) Returns the `index` cell reference of `TvmSlice`, where `0 ≤ index ≤ 3`.

### preloadInt() and preloadIntQ()

```solidity
// (1)
<TvmSlice>.preloadInt(uint9 bitSize) returns (int);

// (2)
<TvmSlice>.preloadIntQ(uint9 bitSize) returns (optional(int));
```

Same as [\<TvmSlice>.loadInt() and \<TvmSlice>.loadIntQ()](tvmslice.md#loadint-and-loadintq) but doesn't modify `TvmSlice`.

### preloadUint() and preloadUintQ()

```solidity
// (1)
<TvmSlice>.preloadUint(uint9 bitSize) returns (uint);

// (2)
<TvmSlice>.preloadUintQ(uint9 bitSize) returns (optional(uint));
```

Same as [\<TvmSlice>.loadUint() and \<TvmSlice>.loadUintQ()](tvmslice.md#loaduint-and-loaduintq) but doesn't modify `TvmSlice`.

### Preload little-endian integers

```solidity
<TvmSlice>.preloadIntLE4() returns (int32)
<TvmSlice>.preloadIntLE8() returns (int64)
<TvmSlice>.preloadUintLE4() returns (uint32)
<TvmSlice>.preloadUintLE8() returns (uint64)

<TvmSlice>.preloadIntLE4Q() returns (optional(int32))
<TvmSlice>.preloadIntLE8Q() returns (optional(int64))
<TvmSlice>.preloadUintLE4Q() returns (optional(uint32))
<TvmSlice>.preloadUintLE8Q() returns (optional(uint64))
```

Same as [Load little-endian integers](tvmslice.md#load-little-endian-integers) but doesn't modify `TvmSlice`.

### preloadSlice() and preloadSliceQ()

```solidity
// (1)
<TvmSlice>.preloadSlice(uint10 bits) returns (TvmSlice);

// (2)
<TvmSlice>.preloadSlice(uint10 bits, uint refs) returns (TvmSlice);

// (3)
<TvmSlice>.preloadSliceQ(uint10 bits) returns (optional(TvmSlice));

// (4)
<TvmSlice>.preloadSliceQ(uint10 bits, uint4 refs) returns (optional(TvmSlice));
```

Same as [\<TvmSlice>.loadSlice() and \<TvmSlice>.loadSliceQ()](tvmslice.md#loadslice-and-loadsliceq) but doesn't modify `TvmSlice`.
