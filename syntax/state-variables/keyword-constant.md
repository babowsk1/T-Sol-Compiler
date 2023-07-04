# Keyword constant

For `constant` variables, the value has to be a compile-time constant, which is substituted where the variable is used. The value has to be assigned where the variable is declared. Example:

```solidity
contract MyContract {
    uint constant cost = 100;
    uint constant cost2 = cost + 200;
    address constant dest = address.makeAddrStd(-1, 0x89abcde);
}
```
