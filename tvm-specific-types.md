# TVM specific types

T-Sol Compiler expands functionality of some existing types and adds several new TVM specific types: TvmCell, TvmSlice, TvmBuilder and ExtraCurrencyCollection. Full description of these types can be found in [TVM](https://broxus.gitbook.io/threaded-virtual-machine/) and [Everscale documentation](https://docs.everscale.network/).

## TON units

A literal number can take a suffix to specify a subdenomination of TON currency, where numbers without a postfix are assumed to be nanotons.

```TVMSolidity
uint a0 = 1 nano; // a0 == 1
uint a1 = 1 nanoton; // a1 == 1
uint a2 = 1 nTon; // a2 == 1
uint a3 = 1 ton; // a3 == 1 000 000 000 (1e9)
uint a4 = 1 Ton; // a4 == 1 000 000 000 (1e9)
uint a5 = 1 micro; // a5 == 1 000
uint a6 = 1 microton; // a6 == 1 000
uint a7 = 1 milli; // a7 == 1 000 000
uint a8 = 1 milliton; // a8 == 1 000 000
uint a9 = 1 kiloton; // a9 == 1 000 000 000 000 (1e12)
uint a10 = 1 kTon; // a10 == 1 000 000 000 000 (1e12)
uint a11 = 1 megaton; // a11 == 1 000 000 000 000 000 (1e15)
uint a12 = 1 MTon; // a12 == 1 000 000 000 000 000 (1e15)
uint a13 = 1 gigaton; // a13 == 1 000 000 000 000 000 000 (1e18)
uint a14 = 1 GTon; // a14 == 1 000 000 000 000 000 000 (1e18)

uint a0 = 1 nano; // a0 == 1 == 1e-9 ever
uint a1 = 1 nanoever; // a1 == 1 == 1e-9 ever
uint a3 = 1 ever; // a3 == 1 000 000 000 (1e9)
uint a4 = 1 Ever; // a4 == 1 000 000 000 (1e9)
uint a5 = 1 micro; // a5 == 1 000 == 1e-6 ever
uint a6 = 1 microever; // a6 == 1 000 == 1e-6 ever
uint a7 = 1 milli; // a7 == 1 000 000 == 1e-3 ever
uint a8 = 1 milliever; // a8 == 1 000 000 == 1e-3 ever
uint a9 = 1 kiloever; // a9 == 1 000 000 000 000 (1e12) == 1e3 ever
uint a10 = 1 kEver; // a10 == 1 000 000 000 000 (1e12) == 1e3 ever
uint a11 = 1 megaever; // a11 == 1 000 000 000 000 000 (1e15) == 1e6 ever
uint a12 = 1 MEver; // a12 == 1 000 000 000 000 000 (1e15) == 1e6 ever
uint a13 = 1 gigaever; // a13 == 1 000 000 000 000 000 000 (1e18) == 1e9 ever
uint a14 = 1 GEver; // a14 == 1 000 000 000 000 000 000 (1e18) == 1e9 ever
```

## TvmCell

`TvmCell` represents *TVM cell* ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - 1.1.3). The compiler defines the following
operators and functions to work with this type:

Comparison operators:
`==`, `!=` (evaluate to `bool`)

### \<TvmCell\>.depth()

```TVMSolidity
<TvmCell>.depth() returns(uint16);
```

Returns the depth **d** of the `TvmCell` **c**. If **c** has no references, then **d** = 0;
otherwise **d** is equal to one plus the maximum of depths of cells referred to from **c**.
If **c** is a Null instead of a Cell, returns zero.

### \<TvmCell\>.dataSize()

```TVMSolidity
<TvmCell>.dataSize(uint n) returns (uint /*cells*/, uint /*bits*/, uint /*refs*/);
```

Returns the number of distinct cells, data bits in the distinct cells and
cell references in the distinct cells. If number of the distinct cells
exceeds `n+1` then a cell overflow [exception](#tvm-exception-codes) is thrown.
This function is a wrapper for the `CDATASIZE` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

### \<TvmCell\>.dataSizeQ()

```TVMSolidity
<TvmCell>.dataSizeQ(uint n) returns (optional(uint /*cells*/, uint /*bits*/, uint /*refs*/));
```

Returns the number of distinct cells, data bits in the distinct cells and
cell references in the distinct cells. If number of the distinct cells
exceeds `n+1` then this function returns an `optional` that has no value.
This function is a wrapper for the `CDATASIZEQ` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

### \<TvmCell\>.toSlice()

```TVMSolidity
<TvmCell>.toSlice() returns (TvmSlice);
```

Converts a `TvmCell` to `TvmSlice`.

## TvmSlice

`TvmSlice` represents *TVM cell slice* ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - 1.1.3). The compiler defines the following
operators and functions to work with this type:

Comparison operators:
`<=`, `<`, `==`, `!=`, `>=`, `>` (evaluate to bool).

Note: only data bits from the root cells are compared. References are ignored.

`TvmSlice` can be converted to `bytes`. It costs at least 500 gas units.

### \<TvmSlice\>.empty()

```TVMSolidity
<TvmSlice>.empty() returns (bool);
```

Checks whether the `TvmSlice` is empty (i.e., contains no data bits and no cell references).

### \<TvmSlice\>.size()

```TVMSolidity
<TvmSlice>.size() returns (uint16 /*bits*/, uint8 /*refs*/);
```

Returns the number of data bits and references in the `TvmSlice`.

### \<TvmSlice\>.bits()

```TVMSolidity
<TvmSlice>.bits() returns (uint16);
```

Returns the number of data bits in the `TvmSlice`.

### \<TvmSlice\>.refs()

```TVMSolidity
<TvmSlice>.refs() returns (uint8);
```

Returns the number of references in the `TvmSlice`.

### \<TvmSlice\>.dataSize()

```TVMSolidity
<TvmSlice>.dataSize(uint n) returns (uint /*cells*/, uint /*bits*/, uint /*refs*/);
```

Returns the number of distinct cells, data bits in the distinct cells and
cell references in the distinct cells. If number of the distinct cells
exceeds `n+1` then a cell overflow [exception](#tvm-exception-codes) is thrown.
Note that the returned `count of distinct cells` does not take into
account the cell that contains the slice itself.
This function is a wrapper for `SDATASIZE` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

### \<TvmSlice\>.dataSizeQ()

```TVMSolidity
<TvmSlice>.dataSizeQ(uint n) returns (optional(uint /*cells*/, uint /*bits*/, uint /*refs*/));
```

Returns the number of distinct cells, data bits in the distinct cells and
cell references in the distinct cells. If number of the distinct cells
exceeds `n+1` then this function returns an `optional` that has no value.
Note that the returned `count of distinct cells` does not take into
account the cell that contains the slice itself.
This function is a wrapper for `SDATASIZEQ` opcode ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.7).

### \<TvmSlice\>.depth()

```TVMSolidity
<TvmSlice>.depth() returns (uint16);
```

Returns the depth of `TvmSlice`. If the `TvmSlice` has no references, then 0 is returned,
otherwise function result is one plus the maximum of depths of the cells referred to from the slice.

### \<TvmSlice\>.hasNBits(), \<TvmSlice\>.hasNRefs() and \<TvmSlice\>.hasNBitsAndRefs()

```TVMSolidity
<TvmSlice>.hasNBits(uint10 bits) returns (bool);
<TvmSlice>.hasNRefs(uint2 refs) returns (bool);
<TvmSlice>.hasNBitsAndRefs(uint10 bits, uint2 refs) returns (bool);
```

Checks whether the `TvmSlice` contains the specified amount of data bits and references.

### \<TvmSlice\>.compare()

```TVMSolidity
<TvmSlice>.compare(TvmSlice other) returns (int2);
```

Lexicographically compares the `slice` and `other` data bits of the root slices and returns result as an integer:

* 1 - `slice` > `other`
* 0 - `slice` == `other`
* -1 - `slice` < `other`

### TvmSlice load primitives

All `load*` functions below modify the `TvmSlice` object. If you wants to load second reference from the `TvmSlice`, you should load the first one with [\<TvmSlice\>.loadRef()](#tvmsliceloadref) and then load the reference you need. The same rule is applied to data bits. To load bits from 2 to 10 positions, you should load or skip first two bits.

#### \<TvmSlice\>.load()

```TVMSolidity
<TvmSlice>.load(TypeA, TypeB, ...) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Sequentially loads values of the specified types from the `TvmSlice`.
Supported types: `uintN`, `intN`, `bytesN`, `bool`, `ufixedMxN`, `fixedMxN`, `address`, `contract`,
`TvmCell`, `bytes`, `string`, `mapping`, `ExtraCurrencyCollection`, `array`, `optional` and 
`struct`.  Example:

```TVMSolidity
TvmSlice slice = ...;
(uint8 a, uint16 b) = slice.load(uint8, uint16);
(uint16 num0, uint32 num1, address addr) = slice.load(uint16, uint32, address);
```

See also: [\<TvmBuilder\>.store()](#tvmbuilderstore).
**Note**: if all the argument types can't be loaded from the slice a cell underflow [exception](#tvm-exception-codes) is thrown.

#### \<TvmSlice\>.loadQ()

```TVMSolidity
<TvmSlice>.loadQ(TypeA, TypeB, ...) returns (optional(TypeA, TypeB, ...));
```

Sequentially decodes values of the specified types from the `TvmSlice` 
if the `TvmSlice` holds sufficient data for all specified types. Otherwise, returns `null`.

```TVMSolidity
TvmSlice slice = ...;
optional(uint) a = slice.loadQ(uint);
optional(uint8, uint16) b = slice.loadQ(uint8, uint16);
```

See also: [\<TvmBuilder\>.store()](#tvmbuilderstore).

#### \<TvmSlice\>.loadRef()

```TVMSolidity
<TvmSlice>.loadRef() returns (TvmCell);
```

Loads a cell from the `TvmSlice` reference.

#### \<TvmSlice\>.loadRefAsSlice()

```TVMSolidity
<TvmSlice>.loadRefAsSlice() returns (TvmSlice);
```

Loads a cell from the `TvmSlice` reference and converts it into a `TvmSlice`.

#### \<TvmSlice\>.loadInt() and \<TvmSlice\>.loadIntQ()

```TVMSolidity
(1)
<TvmSlice>.loadInt(uint9 bitSize) returns (int);
(2)
<TvmSlice>.loadIntQ(uint9 bitSize) returns (optional(int));
```

(1) Loads a signed integer with the given **bitSize** from the `TvmSlice`.

(2) Loads a signed integer with the given **bitSize** from the `TvmSlice` if `TvmSlice` contains it. Otherwise, returns `null`.

#### \<TvmSlice\>.loadUint() and \<TvmSlice\>.loadUintQ()

```TVMSolidity
(1)
<TvmSlice>.loadUint(uint9 bitSize) returns (uint);
(2)
<TvmSlice>.loadUintQ(uint9 bitSize) returns (optional(uint));
```

(1) Loads an unsigned integer with the given **bitSize** from the `TvmSlice`.

(2) Loads an unsigned integer with the given **bitSize** from the `TvmSlice` if `TvmSlice` contains it. Otherwise, returns `null`.

#### Load little-endian integers

```TVMSolidity
(1)
<TvmSlice>.loadIntLE2() returns (int16)
<TvmSlice>.loadIntLE4() returns (int32)
<TvmSlice>.loadIntLE8() returns (int64)
<TvmSlice>.loadUintLE2() returns (uint16)
<TvmSlice>.loadUintLE4() returns (uint32)
<TvmSlice>.loadUintLE8() returns (uint64)
(2)
<TvmSlice>.loadIntLE4Q() returns (optional(int32))
<TvmSlice>.loadIntLE8Q() returns (optional(int64))
<TvmSlice>.loadUintLE4Q() returns (optional(uint32))
<TvmSlice>.loadUintLE8Q() returns (optional(uint64))
```

(1) Loads the little-endian integer from `TvmSlice`.

(2) Same as (1) but returns `null` if it's impossible to load the integer.

#### \<TvmSlice\>.loadTons()

```TVMSolidity
<TvmSlice>.loadTons() returns (uint128);
```

Loads (deserializes) **VarUInteger 16** and returns an unsigned 128-bit integer. See [TL-B scheme][3].

#### \<TvmSlice\>.loadSlice() and \<TvmSlice\>.loadSliceQ()

```TVMSolidity
(1)
<TvmSlice>.loadSlice(uint10 bits) returns (TvmSlice);
(2)
<TvmSlice>.loadSlice(uint10 bits, uint refs) returns (TvmSlice);
(3)
<TvmSlice>.loadSliceQ(uint10 bits) returns (optional(TvmSlice));
(4)
<TvmSlice>.loadSliceQ(uint10 bits, uint2 refs) returns (optional(TvmSlice));
```

(1) Loads the first `bits` bits from `TvmSlice`.
(2) Loads the first `bits` bits and `refs` references from `TvmSlice`.
(3) and (4) are same as (1) and (2) but return `optional` type.

#### \<TvmSlice\>.loadFunctionParams()

```TVMSolidity
// Loads parameters of the public/external function without "responsible" attribute
<TvmSlice>.loadFunctionParams(functionName) returns (TypeA /*a*/, TypeB /*b*/, ...);

// Loads parameters of the public/external function with "responsible" attribute
<TvmSlice>.loadFunctionParams(functionName) returns (uint32 callbackFunctionId, TypeA /*a*/, TypeB /*b*/, ...);

// Loads constructor parameters
<TvmSlice>.loadFunctionParams(ContractName) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Loads parameters of the function or constructor (if contract type is provided). This function is usually used in
**[onBounce](#onbounce)** function.

See example of how to use **onBounce** function:

* [onBounceHandler](https://github.com/tonlabs/samples/blob/master/solidity/16_onBounceHandler.sol)

#### \<TvmSlice\>.loadStateVars()

```TVMSolidity
<TvmSlice>.loadStateVars(ContractName) returns (uint256 /*pubkey*/, uint64 /*timestamp*/, bool /*constructorFlag*/, Type1 /*var1*/, Type2 /*var2*/, ...);
```

Loads state variables from `slice` that is obtained from the field `data` of `stateInit`

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

#### \<TvmSlice\>.skip()

```TVMSolidity
<TvmSlice>.skip(uint10 bits);
<TvmSlice>.skip(uint10 bits, uint2 refs);
```

Skips the first `bits` bits and `refs` references from the `TvmSlice`.

#### \<TvmSlice\>.loadZeroes(), \<TvmSlice\>.loadOnes() and \<TvmSlice\>.loadSame()

```TVMSolidity
(1)
<TvmSlice>.loadZeroes() returns (uint10 n);
(2)
<TvmSlice>.loadOnes() returns (uint10 n);
(3)
<TvmSlice>.loadSame(uint1 value) returns (uint10 n);
```

(1) Returns the count `n` of leading zero bits in `TvmSlice`, and removes these bits from `TvmSlice`.

(2) Returns the count `n` of leading one bits in `TvmSlice`, and removes these bits from `TvmSlice`.

(3) Returns the count `n` of leading bits equal to `0 ≤ value ≤ 1` in `TvmSlice`, and removes these bits from `TvmSlice`.

See also: [\<TvmBuilder\>.storeZeroes(), \<TvmBuilder\>.storeOnes() and \<TvmBuilder\>.storeSame()](#tvmbuilderstorezeroes-tvmbuilderstoreones-and-tvmbuilderstoresame).

### TvmSlice preload primitives

All `preload*` functions below don't modify the `TvmSlice` object.

#### \<TvmSlice\>.preload()

```TVMSolidity
<TvmSlice>.preload(TypeA, TypeB, ...) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Same as [\<TvmSlice\>.load()](#tvmsliceload) but doesn't modify `TvmSlice`.

#### \<TvmSlice\>.preloadQ()

```TVMSolidity
<TvmSlice>.preloadQ(TypeA, TypeB, ...) returns (optional(TypeA, TypeB, ...));
```

Same as [\<TvmSlice\>.loadQ()](#tvmsliceloadq) but doesn't modify `TvmSlice`.

#### \<TvmSlice\>.preloadRef()

```TVMSolidity
(1)
<TvmSlice>.preloadRef() returns (TvmCell);
(2)
<TvmSlice>.preloadRef(uint2 index) returns (TvmCell);
```

(1) Returns the first cell reference of `TvmSlice`.

(2) Returns the `index` cell reference of `TvmSlice`, where `0 ≤ index ≤ 3`.

#### \<TvmSlice\>.preloadInt() and \<TvmSlice\>.preloadIntQ()

```TVMSolidity
(1)
<TvmSlice>.preloadInt(uint9 bitSize) returns (int);
(2)
<TvmSlice>.preloadIntQ(uint9 bitSize) returns (optional(int));
```

Same as [\<TvmSlice\>.loadInt() and \<TvmSlice\>.loadIntQ()](#tvmsliceloadint-and-tvmsliceloadintq) but doesn't modify `TvmSlice`.

#### \<TvmSlice\>.preloadUint() and \<TvmSlice\>.preloadUintQ()

```TVMSolidity
(1)
<TvmSlice>.preloadUint(uint9 bitSize) returns (uint);
(2)
<TvmSlice>.preloadUintQ(uint9 bitSize) returns (optional(uint));
```

Same as [\<TvmSlice\>.loadUint() and \<TvmSlice\>.loadUintQ()](#tvmsliceloaduint-and-tvmsliceloaduintq) but doesn't modify `TvmSlice`.

#### Preload little-endian integers

```TVMSolidity
<TvmSlice>.preloadIntLE4() returns (int32)
<TvmSlice>.preloadIntLE8() returns (int64)
<TvmSlice>.preloadUintLE4() returns (uint32)
<TvmSlice>.preloadUintLE8() returns (uint64)

<TvmSlice>.preloadIntLE4Q() returns (optional(int32))
<TvmSlice>.preloadIntLE8Q() returns (optional(int64))
<TvmSlice>.preloadUintLE4Q() returns (optional(uint32))
<TvmSlice>.preloadUintLE8Q() returns (optional(uint64))
```

Same as [Load little-endian integers](#load-little-endian-integers) but doesn't modify `TvmSlice`.

#### \<TvmSlice\>.preloadSlice() and \<TvmSlice\>.preloadSliceQ()

```TVMSolidity
(1)
<TvmSlice>.preloadSlice(uint10 bits) returns (TvmSlice);
(2)
<TvmSlice>.preloadSlice(uint10 bits, uint refs) returns (TvmSlice);
(3)
<TvmSlice>.preloadSliceQ(uint10 bits) returns (optional(TvmSlice));
(4)
<TvmSlice>.preloadSliceQ(uint10 bits, uint4 refs) returns (optional(TvmSlice));
```

Same as [\<TvmSlice\>.loadSlice() and \<TvmSlice\>.loadSliceQ()](#tvmsliceloadslice-and-tvmsliceloadsliceq) but doesn't modify `TvmSlice`.

## TvmBuilder

`TvmBuilder` represents *TVM cell builder* ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - 1.1.3). T-Sol Compiler defines the following
functions to work with this type:

### \<TvmBuilder\>.toSlice()

```TVMSolidity
<TvmBuilder>.toSlice() returns (TvmSlice);
```

Converts a `TvmBuilder` into `TvmSlice`.

### \<TvmBuilder\>.toCell()

```TVMSolidity
<TvmBuilder>.toCell() returns (TvmCell);
```

Converts a `TvmBuilder` into `TvmCell`.

### \<TvmBuilder\>.size()

```TVMSolidity
<TvmBuilder>.size() returns (uint16 /*bits*/, uint8 /*refs*/);
```

Returns the number of data bits and references already stored in the `TvmBuilder`.

### \<TvmBuilder\>.bits()

```TVMSolidity
<TvmBuilder>.bits() returns (uint16);
```

Returns the number of data bits already stored in the `TvmBuilder`.

### \<TvmBuilder\>.refs()

```TVMSolidity
<TvmBuilder>.refs() returns (uint8);
```

Returns the number of references already stored in the `TvmBuilder`.

### \<TvmBuilder\>.remBits()

```TVMSolidity
<TvmBuilder>.remBits() returns (uint16);
```

Returns the number of data bits that can still be stored in the `TvmBuilder`.

### \<TvmBuilder\>.remRefs()

```TVMSolidity
<TvmBuilder>.remRefs() returns (uint8);
```

Returns the number of references that can still be stored in the `TvmBuilder`.

### \<TvmBuilder\>.remBitsAndRefs()

```TVMSolidity
<TvmBuilder>.remBitsAndRefs() returns (uint16 /*bits*/, uint8 /*refs*/);
```

Returns the number of data bits and references that can still be stored in the `TvmBuilder`.

### \<TvmBuilder\>.depth()

```TVMSolidity
<TvmBuilder>.depth() returns (uint16);
```

Returns the depth of `TvmBuilder`. If no cell references are stored
in the builder, then 0 is returned; otherwise function result is one plus the maximum of
depths of cells referred to from the builder.

### \<TvmBuilder\>.store()

```TVMSolidity
<TvmBuilder>.store(/*list_of_values*/);
```

Stores the list of values into the `TvmBuilder`.

Internal representation of the stored data:

* `uintN`/`intN`/`bytesN` - stored as an N-bit string.
For example, `uint8(100), int16(-3), bytes2(0xaabb)` stored as `0x64fffdaabb`.
* `bool` - stored as a binary zero for `false` or a binary one for `true`. For example,
`true, false, true` stored as `0xb_`.
* `ufixedMxN`/`fixedMxN` - stored as an M-bit string.
* `address`/`contract` - stored according to the [TL-B scheme][3] of `MsgAddress`.
* `TvmCell`/`bytes`/`string` - stored as a cell in reference.
* `TvmSlice`/`TvmBuilder` - all data bits and references of the `TvmSlice` or the `TvmBuilder`
are appended to the `TvmBuilder`, not in a reference as `TvmCell`. To store `TvmSlice`/`TvmBuilder` in
the references use [\<TvmBuilder\>.storeRef()](#tvmbuilderstoreref).
* `mapping`/`ExtraCurrencyCollection` - stored according to the [TL-B scheme][3] of `HashmapE`: if map is
empty then stored as a binary zero, otherwise as a binary one and the dictionary `Hashmap` in a reference.
* `array` - stored as a 32 bit value - size of the array and a `HashmapE` that contains all values of
the array.
* `optional` - stored as a binary zero if the `optional` doesn't contain value. Otherwise, stored as
a binary one and the cell with serialized value in a reference.
* `struct` - stored in the order of its members in the builder. Make sure the entire struct fits into the
builder.

**Note:** there is no gap or offset between two consecutive data assets stored in the `TvmBuilder`.

See [TVM](https://broxus.gitbook.io/threaded-virtual-machine/) to read about notation for bit strings.

Example:

```TVMSolidity
uint8 a = 11;
int16 b = 22;
TvmBuilder builder;
builder.store(a, b, uint(33));
```

See also: [\<TvmSlice\>.load()](#tvmsliceload).

### \<TvmBuilder\>.storeZeroes(), \<TvmBuilder\>.storeOnes() and \<TvmBuilder\>.storeSame()

```TVMSolidity
(1)
<TvmBuilder>.storeZeroes(uint10 n);
(2)
<TvmBuilder>.storeOnes(uint10 n);
(3)
<TvmBuilder>.storeSame(uint10 n, uint1 value);
```

(1) Stores `n` binary zeroes into the `TvmBuilder`.

(2) Stores `n` binary ones into the `TvmBuilder`.

(3) Stores `n` binary `value`s (0 ≤ value ≤ 1) into the `TvmBuilder`.

See also: [\<TvmSlice\>.loadZeroes(), \<TvmSlice\>.loadOnes() and \<TvmSlice\>.loadSame()](#tvmsliceloadzeroes-tvmsliceloadones-and-tvmsliceloadsame).

### \<TvmBuilder\>.storeInt()

```TVMSolidity
<TvmBuilder>.storeInt(int256 value, uint9 bitSize);
```

Stores a signed integer **value** with given **bitSize** in the `TvmBuilder`.

### \<TvmBuilder\>.storeUint()

```TVMSolidity
<TvmBuilder>.storeUint(uint256 value, uint9 bitSize);
```

Stores an unsigned integer **value** with given **bitSize** in the `TvmBuilder`.

### Store little-endian integers

```TVMSolidity
<TvmBuilder>.storeIntLE2(int16)
<TvmBuilder>.storeIntLE4(int32)
<TvmBuilder>.storeIntLE8(int64)
<TvmBuilder>.storeUintLE2(uint16)
<TvmBuilder>.storeUintLE4(uint32)
<TvmBuilder>.storeUintLE8(uint64)
```

Stores the little-endian integer.

### \<TvmBuilder\>.storeRef()

```TVMSolidity
<TvmBuilder>.storeRef(TvmBuilder b);
<TvmBuilder>.storeRef(TvmCell c);
<TvmBuilder>.storeRef(TvmSlice s);
```

Stores `TvmBuilder b`/`TvmCell c`/`TvmSlice s` in the reference of the `TvmBuilder`.

### \<TvmBuilder\>.storeTons()

```TVMSolidity
<TvmBuilder>.storeTons(uint128 value);
```

Stores (serializes) an integer **value** and stores it in the `TvmBuilder` as
**VarUInteger 16**. See [TL-B scheme][3].

See example of how to work with TVM specific types:

* [Message_construction](https://github.com/tonlabs/samples/blob/master/solidity/15_MessageSender.sol)
* [Message_parsing](https://github.com/tonlabs/samples/blob/master/solidity/15_MessageReceiver.sol)

## ExtraCurrencyCollection

ExtraCurrencyCollection represents TVM type ExtraCurrencyCollection. It has the same functions as [**mapping(uint32 => uint256)**](#mapping):

```TVMSolidity
ExtraCurrencyCollection curCol;
uint32 key;
uint256 value;
optional(uint32, uint256) res = curCol.min();
optional(uint32, uint256) res = curCol.next(key);
optional(uint32, uint256) res = curCol.prev(key);
optional(uint32, uint256) res = curCol.nextOrEq(key);
optional(uint32, uint256) res = curCol.prevOrEq(key);
optional(uint32, uint256) res = curCol.delMin();
optional(uint32, uint256) res = curCol.delMax();
optional(uint256) optValue = curCol.fetch(key);
bool exists = curCol.exists(key);
bool isEmpty = curCol.empty();
bool success = curCol.replace(key, value);
bool success = curCol.add(key, value);
optional(uint256) res = curCol.getSet(key, value);
optional(uint256) res = curCol.getAdd(key, value);
optional(uint256) res = curCol.getReplace(key, value);
uint256 uintValue = curCol[index];
```

## optional(Type)

The template optional type manages an optional contained value, i.e. a value that may or may not be present.

### constructing an optional

There are many ways to set a value:

```TVMSolidity
optional(uint) opt;
opt.set(11); // just sets value
opt = 22; // just sets value, too
opt.get() = 33; // if 'opt' has value then set value. Otherwise throws an exception.

optional(uint) another = ...;
opt = another;
```

### \<optional(Type)\>.hasValue()

```TVMSolidity
<optional(Type)>.hasValue() returns (bool);
```

Checks whether the `optional` contains a value.

### \<optional(Type)\>.get()

```TVMSolidity
<optional(Type)>.get() returns (Type);
```

Returns the contained value, if the `optional` contains one. Otherwise, throws an exception.

### \<optional(Type)\>.set()

```TVMSolidity
<optional(Type)>.set(Type value);
```

Replaces content of the `optional` with **value**.

### \<optional(Type)\>.reset()

```TVMSolidity
<optional(Type)>.reset();
```

Deletes content of the `optional`.

### Keyword `null`

Keyword `null` is a constant that is used to indicate an optional type with uninitialized state.
Example:

```TVMSolidity
optional(uint) x = 123;
x = null; // reset value
```

## variant

The `variant` type acts like a union for the most common solidity data types. Supported only `uint` so far.

### variant.isUint()

```TVMSolidity
<variant>.isUint() returns (bool)
```

Checks whether `<variant>` holds `uint` type. 

### variant.toUint()

Converts `<variant>` to `uint` type if it's possible. Otherwise, throws an exception with code `77`.

## vector(Type)

`vector(Type)` is a template container type capable of storing an arbitrary set of values of a
single type, pretty much like dynamic-sized array.
Two major differences are that `vector(Type)`:

1. is much more efficient than a dynamic-sized array;
2. has a lifespan of a smart-contract execution, so it can't be neither passed nor returned as an
external function call parameter, nor stored in a state variable.

**Note:** `vector` implementation based on `TVM Tuple` type, and it has a limited
length of 255 * 255 = 65025 values.

### \<vector(Type)\>.push(Type)

```TVMSolidity
<vector(Type)>.push(Type obj);
```

Appends **obj** to the `vector`.

```TVMSolidity
vector(uint) vect;
uint a = 11;
vect.push(a);
vect.push(111);
```

### \<vector(Type)\>.pop()

```TVMSolidity
<vector(Type)>.pop() returns (Type);
```

Pops the last value from the `vector` and returns it.

```TVMSolidity
vector(uint) vect;
...
uint a = vect.pop();
```

### \<vector(Type)\>.length()

```TVMSolidity
<vector(Type)>.length() returns (uint8);
```

Returns length of the `vector`.

```TVMSolidity
vector(uint) vect;
...
uint8 len = vect.length();
```

### \<vector(Type)\>.empty()

```TVMSolidity
<vector(Type)>.empty() returns (bool);
```

Checks whether the `vector` is empty.

```TVMSolidity
vector(uint) vect;
...
bool is_empty = vect.empty();
```
