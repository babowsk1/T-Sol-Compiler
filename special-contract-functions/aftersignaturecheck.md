# afterSignatureCheck

```solidity
function afterSignatureCheck(TvmSlice body, TvmCell message) private inline returns (TvmSlice) {
    /*...*/
}
```

`afterSignatureCheck` function is used to define custom replay protection function instead of the
default one.

NB: Do not use [tvm.commit()](#tvmcommit) or [tvm.accept()](#tvmaccept) in this function as it called before the constructor call.
**time** and **expire** header fields can be used for replay protection and, if set, they should be read in `afterSignatureCheck`.
See also: [Contract execution](#contract-execution).
See an example of how to define this function:

* [Custom replay protection](https://github.com/tonlabs/samples/blob/master/solidity/14_CustomReplayProtection.sol)