# Mapping

T-Sol Compiler expands `mapping` type with the following functions. In examples below `\<map\>` defines the object of `mapping(KeyType => ValueType)` type.

Address, bytes, string, bool, contract, enum, fixed bytes, integer and struct types can be used as a `KeyType`. Struct type can be used as `KeyType` only if it contains only integer, boolean, fixed bytes or enum types and fits \~1023 bit. Example of mapping that has a struct as a `KeyType`:

```solidity
struct Point {
    uint x;
    uint y;
    uint z;
}

mapping(Point => address) map;

function test(uint x, uint y, uint z, address addr) public {
    Point p = Point(x, y, z);
    map[p] = addr;
}
```

If you use `mapping(KeyType => TvmSlice) map;` be sure that sum of bit-length of `KeyType` and bit-length of `TvmSlice` is less than 1023 bit.

Struct `KeyType` can be used to sort keys of the mapping in ascending order. In the example above addresses in the mapping are sorted by their keys. `x` field of the Point struct is used in comparison first, second is `y` and the last is `z`.

If `bytes`, `string` or `TvmCell` types are used as `KeyType` then `mapping` stores only hashes of mapping keys. That's why for these types the `delMin`/`min`/`next` and another mapping methods return `uint256` as key (not `bytes`/`string`/`TvmCell`).

If you use mapping as an input or output param for public/external functions, `KeyType` of the mapping can only be of type `address` or of `int<M>`/`uint<M>` types with M from 8 to 256.

See example of how to work with mappings:

* [database](https://github.com/tonlabs/samples/blob/master/solidity/13\_BankCollector.sol)
* [client](https://github.com/tonlabs/samples/blob/master/solidity/13\_BankCollectorClient.sol)

## Keyword `emptyMap`

Keyword `emptyMap` is a constant that is used to indicate a mapping of arbitrary type without values.

Example:

```solidity
struct Stakes {
    uint total;
    mapping(uint => uint) stakes;
}

// create struct with empty mapping `stakes`
Stakes stakes = Stakes({stakes: emptyMap, total: 200});
```

## operator\[]

```solidity
<map>.operator[](KeyType index) returns (ValueType);
```

Returns the item of `ValueType` with **index** key or returns the default value if key is not in the mapping.

## at()

```solidity
<map>.operator[](KeyType index) returns (ValueType);
```

Returns the item of `ValueType` with **index** key. Throws an [exception](../../troubleshooting/tvm-exception-codes.md) if key is not in the mapping.

## min() and max()

```solidity
<map>.min() returns (optional(KeyType, ValueType));
<map>.max() returns (optional(KeyType, ValueType));
```

Computes the minimal (maximal) key in the `mapping` and returns an `optional` value containing that key and the associated value. If `mapping` is empty, this function returns an empty `optional`.

## next() and prev()

```solidity
<map>.next(KeyType key) returns (optional(KeyType, ValueType));
<map>.prev(KeyType key) returns (optional(KeyType, ValueType));
```

Computes the minimal (maximal) key in the `mapping` that is lexicographically greater (less) than **key** and returns an `optional` value containing that key and the associated value. Returns an empty `optional` if there is no such key. If KeyType is an integer type, argument for this functions can not possibly fit `KeyType`.

Example:

```solidity
KeyType key;
// init key
optional(KeyType, ValueType) nextPair = map.next(key);
optional(KeyType, ValueType) prevPair = map.prev(key);

if (nextPair.hasValue()) {
    (KeyType nextKey, ValueType nextValue) = nextPair.get(); // unpack optional value
    // using nextKey and nextValue
}

mapping(uint8 => uint) m;
optional(uint8, uint) = m.next(-1); // ok, param for next/prev can be negative 
optional(uint8, uint) = m.prev(65537); // ok, param for next/prev can not possibly fit to KeyType (uint8 in this case)
```

## nextOrEq() and prevOrEq()

```solidity
<map>.nextOrEq(KeyType key) returns (optional(KeyType, ValueType));
<map>.prevOrEq(KeyType key) returns (optional(KeyType, ValueType));
```

Computes the minimal (maximal) key in the `mapping` that is lexicographically greater than or equal to (less than or equal to) **key** and returns an `optional` value containing that key and the associated value. Returns an empty `optional` if there is no such key. If KeyType is an integer type, argument for this functions can not possibly fit `KeyType`.

## delMin() and delMax()

```solidity
<map>.delMin() returns (optional(KeyType, ValueType));
<map>.delMax() returns (optional(KeyType, ValueType));
```

If mapping is not empty then this function computes the minimal (maximum) key of the `mapping`, deletes that key and the associated value from the `mapping` and returns an `optional` value containing that key and the associated value. Returns an empty `optional` if there is no such key.

## fetch()

```solidity
<map>.fetch(KeyType key) returns (optional(ValueType));
```

Checks whether **key** is present in the `mapping` and returns an `optional` with the associated value. Returns an empty `optional` if there is no such key.

## exists()

```solidity
<map>.exists(KeyType key) returns (bool);
```

Returns whether **key** is present in the `mapping`.

## empty()

```solidity
<map>.empty() returns (bool);
```

Returns whether the `mapping` is empty.

## replace()

```solidity
<map>.replace(KeyType key, ValueType value) returns (bool);
```

Sets the value associated with **key** only if **key** is present in the `mapping` and returns the success flag.

## add()

```solidity
<map>.add(KeyType key, ValueType value) returns (bool);
```

Sets the value associated with **key** only if **key** is not present in the `mapping`.

## getSet()

```solidity
<map>.getSet(KeyType key, ValueType value) returns (optional(ValueType));
```

Sets the value associated with **key**, but also returns an `optional` with the previous value associated with the **key**, if any. Otherwise, returns an empty `optional`.

## getAdd()

```solidity
<map>.getAdd(KeyType key, ValueType value) returns (optional(ValueType));
```

Sets the value associated with **key**, but only if **key** is not present in the `mapping`. Returns an `optional` with the old value without changing the dictionary if that value is present in the `mapping`, otherwise returns an empty `optional`.

## getReplace()

```solidity
<map>.getReplace(KeyType key, ValueType value) returns (optional(ValueType));
```

Sets the value associated with **key**, but only if **key** is present in the `mapping`. On success, returns an `optional` with the old value associated with the **key**. Otherwise, returns an empty `optional`.
