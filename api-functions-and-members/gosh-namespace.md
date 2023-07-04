# **gosh** namespace

All `gosh.*` functions are experimental features and are available only in certain blockchain networks.

## diff and zipDiff

```solidity
(1)
gosh.diff(string oldText, string newText) returns (string patch)
(2)
gosh.zipDiff(bytes oldText, bytes newText) returns (bytes patch)
```

(1) Calculates [patch](https://en.wikipedia.org/wiki/Diff) between `oldText` and `newText`. Example: (2) It's the same as `gosh.diff` but it calculates `patch` between compressed strings.

```solidity
string oldText = ...;
string newText = ...;
string patch = gosh.diff(oldText, newText);
```

## applyPatch, applyPatchQ, applyZipPatch, applyZipPatchQ, applyZipBinPatch and applyZipBinPatchQ

```solidity
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

(1) Applies `patch` to the `oldText`. If it's impossible (bad patch), `gosh.applyPatch` throws an exception with type check error code (-8) but`gosh.applyPatchQ` returns `null`. Example: (2) These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to binary arrays. (3) These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to compressed strings. (4) These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to compressed binary arrays.

```solidity
string oldText = ...;
string patch = ...;
string newText = gosh.applyPatch(oldText, patch);
```

## zip and unzip

```solidity
gosh.zip(string text) returns (bytes zip)
gosh.unzip(bytes zip) returns (optional(string) text)
```

`gosh.zip` converts the `text` to compressed `bytes`. `gosh.unzip` reverts such compression.