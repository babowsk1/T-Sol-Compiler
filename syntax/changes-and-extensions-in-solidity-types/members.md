# Members

## wid

```solidity
<address>.wid returns (int8);
```

Returns the workchain id of **addr\_std** or **addr\_var**. Throws "range check error" [exception](../../troubleshooting/tvm-exception-codes.md) for other `address` types.

## value

```solidity
<address>.value returns (uint);
```

Returns the `address` value of **addr\_std** or **addr\_var** if **addr\_var** has 256-bit `address` value. Throws "range check error" [exception](../../troubleshooting/tvm-exception-codes.md) for other `address` types.

## balance

```solidity
address(this).balance returns (uint128);
```

Returns balance of the current contract account in nanotons.

## currencies

```solidity
address(this).currencies returns (ExtraCurrencyCollection);
```

Returns currencies on the balance of the current contract account.
