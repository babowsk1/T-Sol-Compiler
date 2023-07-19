# Bytes

`bytes` is an array of `byte`. It is similar to `byte[]`, but they are encoded in different ways.

Example of `bytes` initialization:

```solidity
// initialised with string
bytes a = "abzABZ0129";
// initialised with hex data
bytes b = hex"01239abf";
```

`bytes` can be converted to `TvmSlice`. Warning: if length of the array is greater than 127 then extra bytes are stored in the first reference of the slice. Use [\<TvmSlice>.loadRef()](../tvm-specific-types/tvmslice.md#loadref) to load that extra bytes.

### empty()

```solidity
<bytes>.empty() returns (bool);
```

Returns status flag whether the `bytes` is empty (its length is 0).

### operator\[]

```solidity
<bytes>.operator[](uint index) returns (byte);
```

Returns the byte located at the **index** position.

Example:

```solidity
bytes byteArray = "abba";
int index = 0;
byte a0 = byteArray[index]; // a0 = 0x61
```

### Slice

```solidity
<bytes>.operator[](uint from, uint to) returns (bytes);
```

Returns the slice of `bytes` \[**from**, **to**), including **from** byte and excluding **to**. Example:

```solidity
bytes byteArray = "01234567890123456789";
bytes slice = byteArray[5:10]; // slice == "56789"
slice = byteArray[10:]; // slice == "0123456789"
slice = byteArray[:10]; // slice == "0123456789"
slice = byteArray[:];  // slice == "01234567890123456789"
```

### length

```solidity
<bytes>.length returns (uint)
```

Returns length of the `bytes` array.

### toSlice

```solidity
<bytes>.toSlice() returns (TvmSlice);
```

Converts `bytes` to `TvmSlice`.
Warning: if length of the array is greater than 127 then extra bytes are
stored in the first reference of the slice. Use [\<TvmSlice\>.loadRef()](../tvm-specific-types/tvmslice.md#loadref) to load that extra bytes.


### dataSize()

```solidity
<bytes>.dataSize(uint n) returns (uint /*cells*/, uint /*bits*/, uint /*refs*/);
```

Same as [\<TvmCell>.dataSize()](bytes.md#datasize).

### dataSizeQ()

```solidity
<bytes>.dataSizeQ(uint n) returns (optional(uint /*cells*/, uint /*bits*/, uint /*refs*/));
```

Same as [\<TvmCell>.dataSizeQ()](bytes.md#datasizeq).

### append()

```solidity
<bytes>.append(bytes tail);
```

Modifies the `bytes` by concatenating **tail** data to the end of the `bytes`.

### Bytes conversion

```solidity
bytes byteArray = "1234";
bytes4 bb = byteArray;
```

`bytes` can be converted to `bytesN`. If `bytes` object has less than **N** bytes, extra bytes are padded with zero bits.
