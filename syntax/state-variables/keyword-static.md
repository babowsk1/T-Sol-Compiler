# Keyword `static`

Static state variables are used in the contract initial state generation.
Such variables can be set while deploying contract from contract
(onchain) or by tvm-linker (offchain). Example:

```solidity
contract C {
    uint static a; // ok
    uint static b = 123; // error
}
```

See also:

* [`code` option usage](/api-functions-and-members.md#code-option-usage)
* [New contract address problem](/api-functions-and-members.md#new-contract-address-problem)