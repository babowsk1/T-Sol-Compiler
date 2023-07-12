# AfterSignatureCheck

```solidity
function afterSignatureCheck(TvmSlice body, TvmCell message) private inline returns (TvmSlice) {
    /*...*/
}
```

`afterSignatureCheck` function is used to define custom replay protection function instead of the default one.

NB: Do not use [tvm.commit()](../api-functions-and-members/tvm-namespace.md#commit) or [tvm.accept()](../api-functions-and-members/tvm-namespace.md#accept) in this function as it called before the constructor call. **time** and **expire** header fields can be used for replay protection and, if set, they should be read in `afterSignatureCheck`. See also: [Contract execution](../../other/contract-execution.md). See an example of how to define this function:

* [Custom replay protection](https://github.com/tonlabs/samples/blob/master/solidity/14\_CustomReplayProtection.sol)
