# TvmBuilder

`TvmBuilder` represents _TVM cell builder_ ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - 1.1.3). T-Sol Compiler defines the following functions to work with this type:

## toSlice()

```solidity
<TvmBuilder>.toSlice() returns (TvmSlice);
```

Converts a `TvmBuilder` into `TvmSlice`.

## toCell()

```solidity
<TvmBuilder>.toCell() returns (TvmCell);
```

Converts a `TvmBuilder` into `TvmCell`.

## size()

```solidity
<TvmBuilder>.size() returns (uint16 /*bits*/, uint8 /*refs*/);
```

Returns the number of data bits and references already stored in the `TvmBuilder`.

## bits()

```solidity
<TvmBuilder>.bits() returns (uint16);
```

Returns the number of data bits already stored in the `TvmBuilder`.

## refs()

```solidity
<TvmBuilder>.refs() returns (uint8);
```

Returns the number of references already stored in the `TvmBuilder`.

## remBits()

```solidity
<TvmBuilder>.remBits() returns (uint16);
```

Returns the number of data bits that can still be stored in the `TvmBuilder`.

## remRefs()

```solidity
<TvmBuilder>.remRefs() returns (uint8);
```

Returns the number of references that can still be stored in the `TvmBuilder`.

## remBitsAndRefs()

```solidity
<TvmBuilder>.remBitsAndRefs() returns (uint16 /*bits*/, uint8 /*refs*/);
```

Returns the number of data bits and references that can still be stored in the `TvmBuilder`.

## depth()

```solidity
<TvmBuilder>.depth() returns (uint16);
```

Returns the depth of `TvmBuilder`. If no cell references are stored in the builder, then 0 is returned; otherwise function result is one plus the maximum of depths of cells referred to from the builder.

## store()

```solidity
<TvmBuilder>.store(/*list_of_values*/);
```

Stores the list of values into the `TvmBuilder`.

The internal representation of the stored data:

* `uintN`/`intN`/`bytesN` - stored as an N-bit string. For example, `uint8(100), int16(-3), bytes2(0xaabb)` stored as `0x64fffdaabb`.
* `bool` - stored as a binary zero for `false` or a binary one for `true`. For example, `true, false, true` stored as `0xb_`.
* `ufixedMxN`/`fixedMxN` - stored as an M-bit string.
* `address`/`contract` - stored according to the \[TL-B scheme]\[3] of `MsgAddress`.
* `TvmCell`/`bytes`/`string` - stored as a cell in reference.
* `TvmSlice`/`TvmBuilder` - all data bits and references of the `TvmSlice` or the `TvmBuilder` are appended to the `TvmBuilder`, not in a reference as `TvmCell`. To store `TvmSlice`/`TvmBuilder` in the references use [\<TvmBuilder>.storeRef()](tvmbuilder.md#tvmbuilderstoreref).
* `mapping`/`ExtraCurrencyCollection` - stored according to the \[TL-B scheme]\[3] of `HashmapE`: if map is empty then stored as a binary zero, otherwise as a binary one and the dictionary `Hashmap` in a reference.
* `array` - stored as a 32 bit value - size of the array and a `HashmapE` that contains all values of the array.
* `optional` - stored as a binary zero if the `optional` doesn't contain value. Otherwise, stored as a binary one and the cell with serialized value in a reference.
* `struct` - stored in the order of its members in the builder. Make sure the entire struct fits into the builder.

**Note:** there is no gap or offset between two consecutive data assets stored in the `TvmBuilder`.

See [TVM](https://broxus.gitbook.io/threaded-virtual-machine/) to read about notation for bit strings.

Example:

```solidity
uint8 a = 11;
int16 b = 22;
TvmBuilder builder;
builder.store(a, b, uint(33));
```

See also: [\<TvmSlice>.load()](tvmbuilder.md#tvmsliceload).

## storeZeroes(), storeOnes() and storeSame()

```solidity
// (1)
<TvmBuilder>.storeZeroes(uint10 n);
// (2)
<TvmBuilder>.storeOnes(uint10 n);
// (3)
<TvmBuilder>.storeSame(uint10 n, uint1 value);
```

(1) Stores `n` binary zeroes into the `TvmBuilder`.

(2) Stores `n` binary ones into the `TvmBuilder`.

(3) Stores `n` binary `value`s (0 ≤ value ≤ 1) into the `TvmBuilder`.

See also: [\<TvmSlice>.loadZeroes(), \<TvmSlice>.loadOnes() and \<TvmSlice>.loadSame()](tvmbuilder.md#tvmsliceloadzeroes-tvmsliceloadones-and-tvmsliceloadsame).

{% hint style="warning" %}
The behavior of the method has been changed since **v0.64.0**: the function **`storeZeros`** has been renamed to **`storeZeroes`**.

Please update your code accordingly to use the new function name **`storeZeroes`** instead. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

## storeInt()

```solidity
<TvmBuilder>.storeInt(int256 value, uint9 bitSize);
```

Stores a signed integer **value** with given **bitSize** in the `TvmBuilder`.

{% hint style="warning" %}
**`<TvmBuilder>.storeSigned()`** has been renamed to **`<TvmBuilder>.storeInt()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

## storeUint()

```solidity
<TvmBuilder>.storeUint(uint256 value, uint9 bitSize);
```

Stores an unsigned integer **value** with given **bitSize** in the `TvmBuilder`.

{% hint style="warning" %}
**`<TvmBuilder>.storeUnsigned()`** has been renamed to **`<TvmBuilder>.storeUint()`**. The old function is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

## storeRef()

```solidity
<TvmBuilder>.storeRef(TvmBuilder b);
<TvmBuilder>.storeRef(TvmCell c);
<TvmBuilder>.storeRef(TvmSlice s);
```

Stores `TvmBuilder b`/`TvmCell c`/`TvmSlice s` in the reference of the `TvmBuilder`.

## storeTons()

```solidity
<TvmBuilder>.storeTons(uint128 value);
```

Stores (serializes) an integer **value** and stores it in the `TvmBuilder` as **VarUInteger 16**. See \[TL-B scheme]\[3].

See example of how to work with TVM specific types:

* [Message\_construction](https://github.com/tonlabs/samples/blob/master/solidity/15\_MessageSender.sol)
* [Message\_parsing](https://github.com/tonlabs/samples/blob/master/solidity/15\_MessageReceiver.sol)

## Store little-endian integers

```solidity
<TvmBuilder>.storeIntLE2(int16)
<TvmBuilder>.storeIntLE4(int32)
<TvmBuilder>.storeIntLE8(int64)
<TvmBuilder>.storeUintLE2(uint16)
<TvmBuilder>.storeUintLE4(uint32)
<TvmBuilder>.storeUintLE8(uint64)
```

Stores the little-endian integer.
