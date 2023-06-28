# Changes and extensions in Solidity types

## Integers

``int`` / ``uint``: Signed and unsigned integers of various sizes. Keywords ``uintN`` and ``intN``
where ``N`` is a number from ``1``  to ``256`` in steps of 1 denotes the number of bits. ``uint`` and ``int``
are aliases for ``uint256`` and ``int256``, respectively.

Operators:

* Comparison: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (evaluate to ``bool``)
* Bit operators: ``&``, ``|``, ``^`` (bitwise exclusive or), ``~`` (bitwise negation)
* Shift operators: ``<<`` (left shift), ``>>`` (right shift)
* Arithmetic operators: ``+``, ``-``, unary ``-``, ``*``, ``/``, ``%`` (modulo), ``**`` (exponentiation)

### bitSize() and uBitSize()

```TVMSolidity
bitSize(int x) returns (uint16)
uBitSize(uint x) returns (uint16)
```

`bitSize` computes the smallest `c` ≥ 0 such that `x` fits into a `c`-bit signed integer
(−2<sup>c−1</sup> ≤ x < 2<sup>c−1</sup>).

`uBitSize` computes the smallest `c` ≥ 0 such that `x` fits into a `c`-bit unsigned integer
(0 ≤ x < 2<sup>c</sup>).

Example:

```TVMSolidity
uint16 s = bitSize(12); // s == 5
uint16 s = bitSize(1); // s == 2
uint16 s = bitSize(-1); // s == 1
uint16 s = bitSize(0); // s == 0

uint16 s = uBitSize(10); // s == 4
uint16 s = uBitSize(1); // s == 1
uint16 s = uBitSize(0); // s == 0
```

## varInt and varUint

`varInt`/`varInt16`/`varInt32`/`varUint`/`varUint16`/`varUint32` are kinds of [Integer](#integers)
types. But they are serialized/deserialized according to [their TLB schemes](https://github.com/ton-blockchain/ton/blob/master/crypto/block/block.tlb#L112).
These schemes are effective if you want to store or send integers, and they usually have small size.
Use these types only if you are sure.
`varInt` is equal to `varInt32`, `varInt32` - `int248` , `varInt16` - `int120`.
`varUint` is equal to `varUint32`, `varUint32` - `uint248` , `varUint16` - `uint120`.
Example:

```TVMSolidity
mapping(uint => varInt) m_map; // use `varInt` as mapping value only if values have small size
m_map[10] = 15;
```

## struct

Structs are custom defined types that can group several variables.

### struct constructor

```TVMSolidity
struct Stakes {
    uint total;
    mapping(uint => uint) stakes;
}

// create struct with default values of its members
Stakes stakes;

// create struct using parameters
mapping(uint => uint) values;
values[100] = 200;
Stakes stakes = Stakes(200, values);

// create struct using named parameters
Stakes stakes = Stakes({stakes: values, total: 200});
```

### \<struct\>.unpack()

```TVMSolidity
<struct>.unpack() returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Unpacks all members stored in the `struct`.

Example:

```TVMSolidity
struct MyStruct {
    uint a;
    int b;
    address c;
}

function f() pure public {
    MyStruct s = MyStruct(1, -1, address(2));
    (uint a, int b, address c) = s.unpack();
}
```

## Arrays


### Array literals

An array literal is a comma-separated list of one or more expressions, enclosed in square brackets.
For example: `[100, 200, 300]`.

Initializing constant state variable:
`uint[] constant fib = [uint(2), 3, 5, 8, 12, 20, 32];`

### Creating new arrays

```TVMSolidity
uint[] arr; // create 0-length array

uint[] arr = new uint[](0); // create 0-length array

uint constant N = 5;
uint[] arr = new uint[](N); // create 5-length array

uint[] arr = new uint[](10); // create 10-length array
```

Note: If `N` is constant expression or integer literal then the complexity of array creation -
`O(1)`. Otherwise, `O(N)`.

### \<array\>.empty()

```TVMSolidity
<array>.empty() returns (bool);
```

Returns status flag whether the `array` is empty (its length is 0).

Example:

```TVMSolidity
uint[] arr;
bool b = arr.empty(); // b == true
arr.push(...);
bool b = arr.empty(); // b == false
```

## bytesN

Variables of the `bytesN` types can be explicitly converted to `bytes`. Note: it costs ~500 gas.

```TVMSolidity
bytes3 b3 = 0x112233;
bytes b = bytes(b3);
```

## bytes

`bytes` is an array of `byte`. It is similar to `byte[]`, but they are encoded in different ways.

Example of `bytes` initialization:

```TVMSolidity
// initialised with string
bytes a = "abzABZ0129";
// initialised with hex data
bytes b = hex"01239abf";
```

`bytes` can be converted to `TvmSlice`. Warning: if length of the array is greater than 127 then extra bytes are stored in the first reference of the slice. Use [\<TvmSlice\>.loadRef()](#tvmsliceloadref) to load that extra bytes.

### \<bytes\>.empty()

```TVMSolidity
<bytes>.empty() returns (bool);
```

Returns status flag whether the `bytes` is empty (its length is 0).

### \<bytes\>.operator[]

```TVMSolidity
<bytes>.operator[](uint index) returns (byte);
```

Returns the byte located at the **index** position.

Example:

```TVMSolidity
bytes byteArray = "abba";
int index = 0;
byte a0 = byteArray[index]; // a0 = 0x61
```

### \<bytes\> slice

```TVMSolidity
<bytes>.operator[](uint from, uint to) returns (bytes);
```

Returns the slice of `bytes` [**from**, **to**), including **from** byte and
excluding **to**.
Example:

```TVMSolidity
bytes byteArray = "01234567890123456789";
bytes slice = byteArray[5:10]; // slice == "56789"
slice = byteArray[10:]; // slice == "0123456789"
slice = byteArray[:10]; // slice == "0123456789"
slice = byteArray[:];  // slice == "01234567890123456789"
```

### \<bytes\>.length

```TVMSolidity
<bytes>.length returns (uint)
```

Returns length of the `bytes` array.

### \<bytes\>.dataSize()

```TVMSolidity
<bytes>.dataSize(uint n) returns (uint /*cells*/, uint /*bits*/, uint /*refs*/);
```

Same as [\<TvmCell\>.dataSize()](#tvmcelldatasize).

### \<bytes\>.dataSizeQ()

```TVMSolidity
<bytes>.dataSizeQ(uint n) returns (optional(uint /*cells*/, uint /*bits*/, uint /*refs*/));
```

Same as [\<TvmCell\>.dataSizeQ()](#tvmcelldatasizeq).

### \<bytes\>.append()

```TVMSolidity
<bytes>.append(bytes tail);
```

Modifies the `bytes` by concatenating **tail** data to the end of the `bytes`.

### bytes conversion

```TVMSolidity
bytes byteArray = "1234";
bytes4 bb = byteArray;
```

`bytes` can be converted to `bytesN`.
If `bytes` object has less than **N** bytes, extra bytes are padded with zero bits.

## string

T-Sol Compiler expands `string` type with the following functions:

**Note**: Due to VM restrictions string length can't exceed `1024 * 127 = 130048` bytes.

`string` can be converted to `TvmSlice`.

### \<string\>.empty()

```TVMSolidity
<string>.empty() returns (bool);
```

Returns status flag whether the `string` is empty (its length is 0).

### \<string\>.byteLength()

```TVMSolidity
<string>.byteLength() returns (uint32);
```

Returns byte length of the `string` data.

### \<string\>.substr()

```TVMSolidity
<string>.substr(uint from[, uint count]) returns (string);
```

Returns the substring starting from the byte with number **from** with byte length **count**.  
**Note**: if count is not set, then the new `string` will be cut from the **from** byte to the end
of the string.

```TVMSolidity
string long = "0123456789";
string a = long.substr(1, 2); // a = "12"
string b = long.substr(6); // b = "6789"
```

### \<string\>.append()

```TVMSolidity
<string>.append(string tail);
```

Appends the tail `string` to the `string`.

### \<string\>.operator+

```TVMSolidity
<string>.operator+(string) returns (string);
<string>.operator+(bytesN) returns (string);
```

Creates new `string` as a concatenation of left and right arguments of the operator +. Example:

```TVMSolidity
string a = "abc";
bytes2 b = "12";
string c = a + b; // "abc12"
```

### \<string\>.find() and \<string\>.findLast()

```TVMSolidity
<string>.find(bytes1 symbol) returns (optional(uint32));
<string>.find(string substr) returns (optional(uint32));
<string>.findLast(bytes1 symbol) returns (optional(uint32));
```

Looks for **symbol** (or substring) in the `string` and returns index of the first (`find`) or the
last (`findLast`) occurrence. If there is no such symbol in the `string`, an empty optional is returned.

Example:

```TVMSolidity
string str = "01234567890";
optional(uint32) a = str.find(byte('0'));
bool s = a.hasValue(); // s == true
uint32 n = a.get(); // n == 0

byte symbol = 'a';
optional(uint32) b = str.findLast(symbol);
bool s = b.hasValue(); // s == false

string sub = "111";
optional(uint32) c = str.find(sub);
bool s = c.hasValue(); // s == false
```

### \<string\>.dataSize()

```TVMSolidity
<string>.dataSize(uint n) returns (uint /*cells*/, uint /*bits*/, uint /*refs*/);
```

Same as [\<TvmCell\>.dataSize()](#tvmcelldatasize).

### \<string\>.dataSizeQ()

```TVMSolidity
<string>.dataSizeQ(uint n) returns (optional(uint /*cells*/, uint /*bits*/, uint /*refs*/));
```

Same as [\<TvmCell\>.dataSizeQ()](#tvmcelldatasizeq).

### \<string\>.toUpperCase()` and \<string\>.toLowerCase()

```TVMSolidity
<string>.toUpperCase() returns (string)
<string>.toLowerCase() returns (string)
```

Converts `string` to upper/lower case. It treats `string` as ASCII string and converts only 'A'..'Z'
and 'a'..'z' symbols. Example:

```TVMSolidity
string s = "Hello";
string a = s.toUpperCase(); // a == "HELLO" 
string b = s.toLowerCase(); // b == "hello" 
```

###  format()

```TvmSolidity
format(string template, TypeA a, TypeB b, ...) returns (string);
```

Builds a `string` with arbitrary parameters. Empty placeholder {} can be filled with integer
(in decimal view), address, string or fixed point number. Fixed point number is printed
based on its type (`fixedMxN` is printed with `N` digits after dot).
Placeholder should be specified in such formats:

* `"{}"` - empty placeholder
* `"{:[0]<width>{"x","d","X","t"}}"` - placeholder for integers. Fills num with 0 if format starts with "0".
Formats integer to have specified `width`. Can format integers in decimal ("d" postfix), lower hex ("x")
or upper hex ("X") form. Format "t" prints number (in nanotons) as a fixed point Ton sum.

Warning: this function consumes too much gas, that's why it's better not to use it onchain.
Example:

```TVMSolidity
string str = format("Hello {} 0x{:X} {}  {}.{} tons", 123, 255, address.makeAddrStd(-33,0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF123456789ABCDE), 100500, 32);
// str == "Hello 123 0xFF -21:7fffffffffffffffffffffffffffffffffffffffffffffffff123456789abcde  100500.32 tons"
str = format("Hello {}", 123); // str == "Hello 123"
str = format("Hello 0x{:X}", 123); // str == "Hello 0x7B"
str = format("{}", -123); // str == "-123"
str = format("{}", address.makeAddrStd(127,0)); // str == "7f:0000000000000000000000000000000000000000000000000000000000000000"
str = format("{}", address.makeAddrStd(-128,0)); // str == "-80:0000000000000000000000000000000000000000000000000000000000000000"
str = format("{:6}", 123); // str == "   123"
str = format("{:06}", 123); // str == "000123"
str = format("{:06d}", 123); // str == "000123"
str = format("{:06X}", 123); // str == "00007B"
str = format("{:6x}", 123); // str == "    7b"
uint128 a = 1 ton;
str = format("{:t}", a); // str == "1.000000000"
a = 123;
str = format("{:t}", a); // str == "0.000000123"
fixed32x3 v = 1.5;
str = format("{}", v); // str == "1.500"
fixed256x10 vv = -987123.4567890321;
str = format("{}", vv); // str == "-987123.4567890321"
```

### stoi()

```TvmSolidity
stoi(string inputStr) returns (optional(int) /*result*/);
```

Converts a `string` into an integer. If `string` starts with '0x' it will be converted from a hexadecimal format,
otherwise it is meant to be number in decimal format. Function returns the integer in case of success.
Otherwise, returns `null`.

Warning: this function consumes too much gas, that's why it's better not to use it onchain.
Example:

```TVMSolidity
optional(int) res;
res = stoi("123"); // res ==123
string hexstr = "0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF123456789ABCDE";
res = stoi(hexstr); // res == 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF123456789ABCDE
res = stoi("0xag"); // res == null
```

### string conversion

```TVMSolidity
string s = "1";
bytes2 b = bytes2(s); // b == 0x3100

string s = "11";
bytes2 b = bytes2(s); // b = 0x3131
```

`string` can be converted to `bytesN` which causes **N** * 8 bits being loaded from the cell and saved to variable.
If `string` object has less than **N** bytes, extra bytes are padded with zero bits.

## address

`address` represents different types of TVM addresses: **addr_none**, **addr_extern**,
**addr_std** and **addr_var**. T-Sol Compiler expands `address` type with the following
members and functions:

### Object creating

#### constructor()

```TVMSolidity
uint address_value;
address addrStd = address(address_value);
```

Constructs an `address` of type **addr_std** with zero workchain id and given address value.

#### address.makeAddrStd()

```TVMSolidity
int8 wid;
uint address;
address addrStd = address.makeAddrStd(wid, address);
```

Constructs an `address` of type **addr_std** with given workchain id **wid** and value **address_value**.

#### address.makeAddrNone()

```TVMSolidity
address addrNone = address.makeAddrNone();
```

Constructs an `address` of type **addr_none**.

#### address.makeAddrExtern()

```TVMSolidity
uint addrNumber;
uint bitCnt;
address addrExtern = address.makeAddrExtern(addrNumber, bitCnt);
```

Constructs an `address` of type **addr_extern** with given **value** with **bitCnt** bit-length.

## Members

### \<address\>.wid

```TVMSolidity
<address>.wid returns (int8);
```

Returns the workchain id of **addr_std** or **addr_var**. Throws "range check error" [exception](#tvm-exception-codes) for other `address` types.

### \<address\>.value

```TVMSolidity
<address>.value returns (uint);
```

Returns the `address` value of **addr_std** or **addr_var** if **addr_var** has 256-bit `address` value. Throws "range check error" [exception](#tvm-exception-codes) for other `address` types.

### \<address\>.balance

```TVMSolidity
address(this).balance returns (uint128);
```

Returns balance of the current contract account in nanotons.

### \<address\>.currencies

```TVMSolidity
address(this).currencies returns (ExtraCurrencyCollection);
```

Returns currencies on the balance of the current contract account.

## Functions

### \<address\>.getType()

```TVMSolidity
<address>.getType() returns (uint8);
```

Returns type of the `address`:
0 - **addr_none**
1 - **addr_extern**
2 - **addr_std**

### \<address\>.isStdZero()

```TVMSolidity
<address>.isStdZero() returns (bool);
```

Returns the result of comparison between this `address` with zero `address` of type **addr_std**.

### \<address\>.isStdAddrWithoutAnyCast()

```TVMSolidity
<address>.isStdAddrWithoutAnyCast() returns (bool);
```

Checks whether this `address` is of type **addr_std** without any cast.

### \<address\>.isExternZero()

```TVMSolidity
<address>.isExternZero() returns (bool);
```

Returns the result of comparison between this `address` with zero `address` of type **addr_extern**.

### \<address\>.isNone()

```TVMSolidity
<address>.isNone() returns (bool);
```

Checks whether this `address` is of type **addr_none**.

### \<address\>.unpack()

```TVMSolidity
<address>.unpack() returns (int8 /*wid*/, uint256 /*value*/);
```

Parses `address` containing a valid `MsgAddressInt` (`addr_std`), applies rewriting
from the anycast (if present) to the same-length prefix of the address, and returns both the
workchain `wid` and the 256-bit address `value`. If the address `value` is not 256-bit, or if
`address` is not a valid serialization of `MsgAddressInt`, throws a cell deserialization
[exception](#tvm-exception-codes).

It's wrapper for opcode `REWRITESTDADDR`.

Example:

```TVMSolidity
(int8 wid, uint addr) = address(this).unpack();
```

### \<address\>.transfer()

```TVMSolidity
<address>.transfer(uint128 value, bool bounce, uint16 flag, TvmCell body, ExtraCurrencyCollection currencies, TvmCell stateInit);
```

Sends an internal outbound message to the `address`. Function parameters:

* `value` (`uint128`) - amount of nanotons sent attached to the message. Note: the sent value is
withdrawn from the contract's balance even if the contract has been called by internal inbound message.
* `currencies` (`ExtraCurrencyCollection`) - additional currencies attached to the message. Defaults to
an empty set.
* `bounce` (`bool`) - if it's set and transaction (generated by the internal outbound message) falls
(only at the computing phase, not at the action phase!) then funds will be returned. Otherwise, (flag isn't
set or transaction terminated successfully) the address accepts the funds even if the account
doesn't exist or is frozen. Defaults to `true`.
* `flag` (`uint16`) - flag that used to send the internal outbound message. Defaults to `0`.
* `body` (`TvmCell`) -  body (payload) attached to the internal message. Defaults to an empty
TvmCell.
* `stateInit` (`TvmCell`) - represents field `init` of `Message X`. If `stateInit` has a wrong
format, a cell underflow [exception](#tvm-exception-codes) at the computing phase is thrown.
See [here](https://github.com/ton-blockchain/ton/blob/master/crypto/block/block.tlb#L148).
Normally, `stateInit` is used in 2 cases: to deploy the contract or to unfreeze the contract.

All parameters can be omitted, except ``value``.

Possible values of parameter `flag`:

* `0` - message carries funds equal to the `value` parameter. Forward fee is subtracted from
the `value`.
* `128` - message carries all the remaining balance of the current smart contract. Parameter `value` is
ignored. The contract's balance will be equal to zero after the message processing.
* `64` - carries funds equal to the `value` parameter plus all the remaining value of the inbound message
(that initiated the contract execution).

Parameter `flag` can also be modified:

* `flag + 1` - means that the sender wants to pay transfer fees separately from contract's balance.
* `flag + 2` - means that any errors arising while processing this message during the action phase
should be ignored. But if the message has wrong format, then the transaction fails and `+ 2` has
no effect.
* `flag + 32` - means that the current account must be destroyed if its resulting balance is zero.
For example, `flag: 128 + 32` is used to send all balance and destroy the contract.

In order to clarify flags usage see [this sample](https://github.com/tonlabs/samples/blob/master/solidity/20_bomber.sol).

```TVMSolidity
address dest = ...;
uint128 value = ...;
bool bounce = ...;
uint16 flag = ...;
TvmCell body = ...;
ExtraCurrencyCollection c = ...;
TvmCell stateInit = ...;
// sequential order of parameters
addr.transfer(value);
addr.transfer(value, bounce);
addr.transfer(value, bounce, flag);
addr.transfer(value, bounce, flag, body);
addr.transfer(value, bounce, flag, body, c);
// using named parameters
destination.transfer({value: 1 ton, bounce: false, flag: 128, body: cell, currencies: c});
destination.transfer({bounce: false, value: 1 ton, flag: 128, body: cell});
destination.transfer({value: 1 ton, bounce: false, stateInit: stateInit});
```

See example of `address.transfer()` usage:

* [giver](https://github.com/tonlabs/samples/blob/master/solidity/7_Giver.sol)

## mapping

T-Sol Compiler expands `mapping` type with the following functions. In examples
below `\<map\>` defines the object of `mapping(KeyType => ValueType)` type.

Address, bytes, string, bool, contract, enum, fixed bytes, integer and struct types can
be used as a `KeyType`. Struct type can be used as `KeyType` only if it contains only
integer, boolean, fixed bytes or enum types and fits ~1023 bit. Example of mapping that
has a struct as a `KeyType`:

```TVMSolidity
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

If you use `mapping(KeyType => TvmSlice) map;` be sure that sum of bit-length of `KeyType`
and bit-length of `TvmSlice` is less than 1023 bit.

Struct `KeyType` can be used to sort keys of the mapping in ascending order. In the example
above addresses in the mapping are sorted by their keys. `x` field of the Point struct
is used in comparison first, second is `y` and the last is `z`.

If `bytes`, `string` or `TvmCell` types are used as `KeyType` then `mapping` stores
only hashes of mapping keys. That's why for these types the `delMin`/`min`/`next` and
another mapping methods return `uint256` as key (not `bytes`/`string`/`TvmCell`).

If you use mapping as an input or output param for public/external functions,
`KeyType` of the mapping can only be of type `address` or of `int<M>`/`uint<M>` types with M from 8 to 256.

See example of how to work with mappings:

* [database](https://github.com/tonlabs/samples/blob/master/solidity/13_BankCollector.sol)
* [client](https://github.com/tonlabs/samples/blob/master/solidity/13_BankCollectorClient.sol)

### Keyword `emptyMap`

Keyword `emptyMap` is a constant that is used to indicate a mapping of arbitrary type without values.

Example:

```TVMSolidity
struct Stakes {
    uint total;
    mapping(uint => uint) stakes;
}

// create struct with empty mapping `stakes`
Stakes stakes = Stakes({stakes: emptyMap, total: 200});
```

### \<mapping\>.operator[]

```TVMSolidity
<map>.operator[](KeyType index) returns (ValueType);
```

Returns the item of `ValueType` with **index** key or returns the default value
if key is not in the mapping.

### \<mapping\>.at()

```TVMSolidity
<map>.operator[](KeyType index) returns (ValueType);
```

Returns the item of `ValueType` with **index** key. Throws an [exception](#tvm-exception-codes) if key
is not in the mapping.

### \<mapping\>.min() and \<mapping\>.max()

```TVMSolidity
<map>.min() returns (optional(KeyType, ValueType));
<map>.max() returns (optional(KeyType, ValueType));
```

Computes the minimal (maximal) key in the `mapping` and returns an `optional`
value containing that key and the associated value. If `mapping` is empty,
this function returns an empty `optional`.

### \<mapping\>.next() and \<mapping\>.prev()

```TVMSolidity
<map>.next(KeyType key) returns (optional(KeyType, ValueType));
<map>.prev(KeyType key) returns (optional(KeyType, ValueType));
```

Computes the minimal (maximal) key in the `mapping` that is lexicographically
greater (less) than **key** and returns an `optional` value containing that
key and the associated value. Returns an empty `optional` if there is no such key.
If KeyType is an integer type, argument for this functions can not possibly fit `KeyType`.

Example:

```TVMSolidity
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

### \<mapping\>.nextOrEq() and \<mapping\>.prevOrEq()

```TVMSolidity
<map>.nextOrEq(KeyType key) returns (optional(KeyType, ValueType));
<map>.prevOrEq(KeyType key) returns (optional(KeyType, ValueType));
```

Computes the minimal (maximal) key in the `mapping` that is lexicographically greater than
or equal to (less than or equal to) **key** and returns an `optional` value containing that
key and the associated value. Returns an empty `optional` if there is no such key.
If KeyType is an integer type, argument for this functions can not possibly fit `KeyType`.

### \<mapping\>.delMin() and \<mapping\>.delMax()

```TVMSolidity
<map>.delMin() returns (optional(KeyType, ValueType));
<map>.delMax() returns (optional(KeyType, ValueType));
```

If mapping is not empty then this function computes the minimal (maximum) key of the `mapping`,
deletes that key and the associated value from the `mapping` and returns an `optional` value
containing that key and the associated value. Returns an empty `optional` if there is no such key.

### \<mapping\>.fetch()

```TVMSolidity
<map>.fetch(KeyType key) returns (optional(ValueType));
```

Checks whether **key** is present in the `mapping` and returns an `optional` with the associated value.
Returns an empty `optional` if there is no such key.

### \<mapping\>.exists()

```TVMSolidity
<map>.exists(KeyType key) returns (bool);
```

Returns whether **key** is present in the `mapping`.

### \<mapping\>.empty()

```TVMSolidity
<map>.empty() returns (bool);
```

Returns whether the `mapping` is empty.

### \<mapping\>.replace()

```TVMSolidity
<map>.replace(KeyType key, ValueType value) returns (bool);
```

Sets the value associated with **key** only if **key** is present in the `mapping` and
returns the success flag.

### \<mapping\>.add()

```TVMSolidity
<map>.add(KeyType key, ValueType value) returns (bool);
```

Sets the value associated with **key** only if **key** is not present in the `mapping`.

### \<mapping\>.getSet()

```TVMSolidity
<map>.getSet(KeyType key, ValueType value) returns (optional(ValueType));
```

Sets the value associated with **key**, but also returns an `optional` with the
previous value associated with the **key**, if any. Otherwise, returns an empty `optional`.

### \<mapping\>.getAdd()

```TVMSolidity
<map>.getAdd(KeyType key, ValueType value) returns (optional(ValueType));
```

Sets the value associated with **key**, but only if **key** is not present in the `mapping`.
Returns an `optional` with the old value without changing the dictionary if that value is present
in the `mapping`, otherwise returns an empty `optional`.

### \<mapping\>.getReplace()

```TVMSolidity
<map>.getReplace(KeyType key, ValueType value) returns (optional(ValueType));
```

Sets the value associated with **key**, but only if **key** is present in the `mapping`.
On success, returns an `optional` with the old value associated with the **key**.
Otherwise, returns an empty `optional`.

## Fixed point number

`fixed` / `ufixed`: Signed and unsigned fixed point number of various sizes. Keywords `ufixedMxN`
and `fixedMxN`, where `M` represents the number of bits taken by the type and `N` represents how
many decimal points are available. `M` must be divisible by 8 and goes from 8 to 256 bits. `N` must
be between 0 and 80, inclusive. `ufixed` and `fixed` are aliases for `ufixed128x18` and
`fixed128x18`, respectively.

Operators:

* Comparison: `<=`, `<`, `==`, `!=`, `>=`, `>` (evaluate to `bool`)
* Arithmetic operators: `+`, `-`, unary `-`, `*`, `/`, `%` (modulo)
* Math operations: [math.min(), math.max()](#mathmin-mathmax), [math.minmax()](#mathminmax),
[math.abs()](#mathabs), [math.divr(), math.divc()](#mathdivr-mathdivc)

## Function type

Function types are the types of functions. Variables of function type can be assigned from functions
and function parameters of function type can be used to pass functions to and return functions from
function calls.

If unassigned variable of function type is called then exception with code 65 is thrown.

```TVMSolidity
function getSum(int a, int b) internal pure returns (int) {
    return a + b;
}

function getSub(int a, int b) internal pure returns (int) {
    return a - b;
}

function process(int a, int b, uint8 mode) public returns (int) {
    function (int, int) returns (int) fun;
    if (mode == 0) {
        fun = getSum;
    } else if (mode == 1) {
        fun = getSub;
    }
    return fun(a, b); // if `fun` isn't initialized then exception is thrown
}
```

## require, revert

In case of exception state variables of the contract are reverted to the state before
[tvm.commit()](#tvmcommit) or to the state of the contract before it was called.
Use error codes that are greater than 100 because other error codes can be
[reserved](#solidity-runtime-errors).
**Note**: if a nonconstant error code is passed as the function argument and the error code
is less than 2 then the error code will be set to 100.

### require

```TVMSolidity
require(bool condition, [uint errorCode = 100, [Type exceptionArgument]]);
```

**require** function can be used to check the condition and throw an exception if the condition
is not met. The function takes condition and optional parameters: error code (unsigned integer)
and the object of any type.

Example:

```TVMSolidity
uint a = 5;

require(a == 5); // ok
require(a == 6); // throws an exception with code 100
require(a == 6, 101); // throws an exception with code 101
require(a == 6, 101, "a is not equal to six"); // throws an exception with code 101 and string
require(a == 6, 101, a); // throws an exception with code 101 and number a
```

### revert

```TVMSolidity
revert(uint errorCode = 100, [Type exceptionArgument]);
```

**revert** function can be used to throw exceptions. The function takes an optional error code
(unsigned integer) and the object of any type.

Example:

```TVMSolidity
uint a = 5;
revert(); // throw exception 100
revert(101); // throw exception 101
revert(102, "We have a some problem"); // throw exception 102 and string
revert(101, a); // throw exception 101 and number a
```

## Libraries

Libraries are similar to contracts, but they can't have state variables
and can't inherit nor be inherited. Libraries can be seen as implicit
base contracts of the contracts that use them. They will not be
explicitly visible in the inheritance hierarchy, but calls of library
functions look just like calls of functions of explicit base contracts
(using qualified access like `LibName.func(a, b, c)`). There is also
another way to call library function: `obj.func(b, c)`.
For now libraries are stored as a part of the code of the contact that
uses libraries. In the future, it can be changed.

### Function call via library name

Example of using library in the manner `LibName.func(a, b, c)`:

```TVMSolidity
// file MathHelper.sol
pragma solidity >= 0.6.0;

// Library declaration
library MathHelper {
    // State variables are forbidden in library but constants are not
    uint constant MAX_VALUE = 300;

    // Library function
    function sum(uint a, uint b) internal pure returns (uint) {
        uint c = a + b;
        require(c < MAX_VALUE);
        return c;
    }
}


// file MyContract.sol
pragma solidity >= 0.6.0;

import "MathHelper.sol";

contract MyContract {
    uint s;

    function addValue(uint value) public returns (uint) {
        s = MathHelper.sum(s, value);
        return s;
    }
}
```

### Function call via object

In TON solidity **arguments of a function call passed by value not by
reference**. It's effective for numbers and even for huge arrays.
See ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.2.3.2).
**But if a library function is called like `obj.func(b, c)` then the
first argument `obj` is passed by reference.**  It's similar to
the `self` variable in Python.
The directive `using A for B;` can be used to attach library functions
(from the library `A`) to any type (`B`) in the context of the contract.
These functions will receive the object they were called for as their
first parameter.
The effect of `using A for *;` is that the functions from the library
`A` are attached to all types.

Example of using library in the manner `obj.func(b, c)`:

```TVMSolidity
// file ArrayHelper.sol
pragma solidity >= 0.6.0;

library ArrayHelper {
    // Delete value from the `array` at `index` position
    function del(uint[] array, uint index) internal pure {
        for (uint i = index; i + 1 < array.length; ++i){
            array[i] = array[i + 1];
        }
        array.pop();
    }
}


// file MyContract.sol
pragma solidity >= 0.6.0;

import "ArrayHelper.sol";

contract MyContract {
    // Attach library function `del` to the type `uint[]`
    using ArrayHelper for uint[];

    uint[] array;

    constructor() public {
        array = [uint(100), 200, 300];
    }

    function deleteElement(uint index) public {
        // Library function call via object.
        // Note: library function `del` has 2 arguments:
        // array is passed by reference and index is passed by value
        array.del(index);
    }
}
```

#### Import

T-Sol Compiler allows user to import remote files using link starting with `http`.
If import file name starts with `http`, then compiler tries to download the file using this
link and saves it to the folder `.solc_imports`. If compiler fails to create this folder or
to download the file, then an error is emitted.

**Note**: to import file from GitHub, one should use link to the raw version of the file.

Example:

```TVMSolidity
pragma ton-solidity >= 0.35.0;
pragma AbiHeader expire;
pragma AbiHeader pubkey;

import "https://github.com/tonlabs/debots/raw/9c6ca72b648fa51962224ec0d7ce91df2a0068c1/Debot.sol";
import "https://github.com/tonlabs/debots/raw/9c6ca72b648fa51962224ec0d7ce91df2a0068c1/Terminal.sol";

contract HelloDebot is Debot {
  ...
}
```