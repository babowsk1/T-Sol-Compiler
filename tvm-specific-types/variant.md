# Variant

The `variant` type acts like a union for the most common solidity data types. Supported only `uint` so far.

## isUint()

```TVMSolidity
<variant>.isUint() returns (bool)
```

Checks whether `<variant>` holds `uint` type.

## toUint()

Converts `<variant>` to `uint` type if it's possible. Otherwise, it throws an exception with the code `77`.
