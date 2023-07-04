# address

`address` represents different types of TVM addresses: **addr_none**, **addr_extern**,
**addr_std** and **addr_var**. T-Sol Compiler expands `address` type with the following
members and functions:

## Object creating

### constructor()

```solidity
uint address_value;
address addrStd = address(address_value);
```

Constructs an `address` of type **addr_std** with zero workchain id and given address value.

### makeAddrStd()

```solidity
int8 wid;
uint address;
address addrStd = address.makeAddrStd(wid, address);
```

Constructs an `address` of type **addr_std** with given workchain id **wid** and value **address_value**.

### makeAddrNone()

```solidity
address addrNone = address.makeAddrNone();
```

Constructs an `address` of type **addr_none**.

### makeAddrExtern()

```solidity
uint addrNumber;
uint bitCnt;
address addrExtern = address.makeAddrExtern(addrNumber, bitCnt);
```

Constructs an `address` of type **addr_extern** with given **value** with **bitCnt** bit-length.