# msgValue

```solidity
pragma msgValue <value>;
```

Allows specifying default value in nanotons attached to the
internal outbound messages used to call another contract. If it's not
specified, this value is set to 10 000 000 nanotons.

Example:

```solidity
pragma msgValue 123456789;
pragma msgValue 1e8;
pragma msgValue 10 ton;
pragma msgValue 10_000_000_123;
```