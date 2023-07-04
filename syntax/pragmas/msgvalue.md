# MsgValue

Allows specifying default value in nanoevers attached to the internal outbound messages used to call another contract. This value is set to 10 000 000 nanoevers if it's not specified.

```solidity
pragma msgValue <value>;
```

Example:

```solidity
pragma msgValue 123456789;
pragma msgValue 1e8;
pragma msgValue 10 ton;
pragma msgValue 10_000_000_123;
```
