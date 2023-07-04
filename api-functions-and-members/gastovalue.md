# GasToValue

```solidity
gasToValue(uint128 gas) returns (uint128 value)
gasToValue(uint128 gas, int8 wid) returns (uint128 value)
```

Returns worth of **gas** in workchain **wid**. Throws an exception if **wid** is not equal to `0` or `-1`. If `wid` is omitted than used the contract's `wid`.
