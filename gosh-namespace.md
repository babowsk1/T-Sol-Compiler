# **gosh** namespace

All `gosh.*` functions are experimental features and are available only in certain blockchain
networks.

 gosh.diff and gosh.zipDiff


```TVMSolidity
(1)
gosh.diff(string oldText, string newText) returns (string patch)
(2)
gosh.zipDiff(bytes oldText, bytes newText) returns (bytes patch)
```
(1) Calculates [patch](https://en.wikipedia.org/wiki/Diff) between `oldText` and `newText`. Example:
(2) It's the same as `gosh.diff` but it calculates `patch` between compressed strings.


```TVMSolidity
string oldText = ...;
string newText = ...;
string patch = gosh.diff(oldText, newText);
```

 gosh.applyPatch, gosh.applyPatchQ, gosh.applyZipPatch, gosh.applyZipPatchQ, gosh.applyZipBinPatch and gosh.applyZipBinPatchQ

```TVMSolidity
(1)
gosh.applyPatch(string oldText, string patch) returns (string newText)
gosh.applyPatchQ(string oldText, string patch) returns (optional(string) newText)
(2)
gosh.applyBinPatch(bytes oldText, bytes patch) returns (bytes newText)
gosh.applyBinPatchQ(bytes oldText, bytes patch) returns (optional(bytes) newText)
(3)
gosh.applyZipPatch(bytes oldText, bytes patch) returns (bytes newText)
gosh.applyZipPatchQ(bytes oldText, bytes patch) returns (optional(bytes) newText)
(4)
gosh.applyZipBinPatch(bytes oldText, bytes patch) returns (bytes newText)
gosh.applyZipBinPatchQ(bytes oldText, bytes patch) returns (optional(bytes) newText)
```

(1)
Applies `patch` to the `oldText`. If it's impossible (bad patch), `gosh.applyPatch` throws an exception with type check
error code (-8) but`gosh.applyPatchQ` returns `null`. Example:
(2)
These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to binary arrays.
(3)
These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to compressed strings.
(4)
These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to compressed binary arrays.

```TVMSolidity
string oldText = ...;
string patch = ...;
string newText = gosh.applyPatch(oldText, patch);
```

 gosh.zip and gosh.unzip

```TVMSolidity
gosh.zip(string text) returns (bytes zip)
gosh.unzip(bytes zip) returns (optional(string) text)
```

`gosh.zip` converts the `text` to compressed `bytes`. `gosh.unzip` reverts such compression.

 Exponentiation

Exponentiation `**` is only available for unsigned types in the exponent. The resulting type of an
exponentiation is always equal to the type of the base. Please take care that it is large enough to
hold the result and prepare for potential assertion failures or wrapping behaviour.

Note that `0**0` throws an exception.

Example:

```TVMSolidity
uint b = 3;
uint32 p = 4;
uint res = b ** p; // res == 81
```

 selfdestruct

```TVMSolidity
selfdestruct(address dest_addr);
```

Creates and sends the message that carries all the remaining balance
of the current smart contract and destroys the current account.

See example of how to use the `selfdestruct` function:

* [Kamikaze](https://github.com/tonlabs/samples/blob/master/solidity/8_Kamikaze.sol)

 sha256

```TVMSolidity
// (1)
sha256(TvmSlice slice) returns (uint256)
// (2)
sha256(bytes b) returns (uint256)
// (3)
sha256(string str) returns (uint256)
```

1. Computes the SHA-256 hash. If the bit-length of `slice` is not divisible by eight, throws a cell
underflow [exception](#tvm-exception-codes). References of `slice` are not used to compute the hash. Only data bits located
in the root cell of `slice` are used.
2. Computes the SHA-256 hash only for the first 127 bytes. If `bytes.length > 127` then `b[128],
b[129], b[130] ...` elements are ignored.
3. Same as for `bytes`: only the first 127 bytes are taken into account.

See also [tvm.hash()](#tvmhash) to compute representation hash of the whole tree of cells.

 gasToValue

```TVMSolidity
gasToValue(uint128 gas) returns (uint128 value)
gasToValue(uint128 gas, int8 wid) returns (uint128 value)
```

Returns worth of **gas** in workchain **wid**.
Throws an exception if **wid** is not equal to `0` or `-1`.
If `wid` is omitted than used the contract's `wid`.

 valueToGas

```TVMSolidity
valueToGas(uint128 value) returns (uint128 gas)
valueToGas(uint128 value, int8 wid) returns (uint128 gas)
```

Counts how much **gas** could be bought on **value** nanotons in workchain **wid**.
Throws an exception if **wid** is not equal to `0` or `-1`.
If `wid` is omitted than used the contract's `wid`.