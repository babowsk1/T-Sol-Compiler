# ValueToGas

Counts how much **gas** could be bought on **value** nanotons in workchain **wid**. Throws an exception if **wid** is not equal to `0` or `-1`. If `wid` is omitted than used the contract's `wid`.

```solidity
valueToGas(uint128 value) returns (uint128 gas)
valueToGas(uint128 value, int8 wid) returns (uint128 gas)
```
