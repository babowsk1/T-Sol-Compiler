# VarInt and varUint

`varInt`/`varInt16`/`varInt32`/`varUint`/`varUint16`/`varUint32` are kinds of [Integer](integers.md) types. But they are serialized/deserialized according to [their TLB schemes](https://github.com/ton-blockchain/ton/blob/master/crypto/block/block.tlb#L112). These schemes are effective if you want to store or send integers, and they usually have small size. Use these types only if you are sure. `varInt` is equal to `varInt32`, `varInt32` - `int248` , `varInt16` - `int120`. `varUint` is equal to `varUint32`, `varUint32` - `uint248` , `varUint16` - `uint120`. Example:

```solidity
mapping(uint => varInt) m_map; // use `varInt` as mapping value only if values have small size
m_map[10] = 15;
```
