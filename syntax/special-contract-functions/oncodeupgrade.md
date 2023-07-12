# OnCodeUpgrade

`onCodeUpgrade` function can have an arbitrary set of arguments and should be executed after [tvm.setcode()](../api-functions-and-members/tvm-namespace.md#setcode) function call. In this function [tvm.resetStorage()](../api-functions-and-members/misc-functions-from-tvm.md#resetstorage) should be called if the set of state variables is changed in the new version of the contract. This function implicitly calls [tvm.commit()](../api-functions-and-members/tvm-namespace.md#commit). `onCodeUpgrade` function does not return value. `onCodeUpgrade` function finishes TVM execution with exit code 0.

Prototype:

```solidity
function onCodeUpgrade(...) private {
    /*...*/
}
```

Function `onCodeUpgrade` had function id = 2 (for compiler <= 0.65.0). Now, it has another id, but you can set `functionID(2)` in new contracts for the `onCodeUpgrade` to upgrade old ones.

See example of how to upgrade code of the contract:

* [old contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_BadContract.sol)
* [new contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_NewVersion.sol)

It's good to pass `TvmCell cell` to the public function that calls `onCodeUpgrade(TvmCell cell, ...)` function. `TvmCell cell` may contain some data that may be useful for the new contract.

```solidity
// old contract
// Public function that changes the code and takes some cell
function updateCode(TvmCell newcode, TvmCell cell) public pure checkPubkeyAndAccept {
    tvm.setcode(newcode);
    tvm.setCurrentCode(newcode);
    // pass cell to new contract
    onCodeUpgrade(cell);
}

function onCodeUpgrade(TvmCell cell) private pure {
}


// new contract
function onCodeUpgrade(TvmCell cell) private pure {
    // new code can use cell that was passed from the old version of the contract
}
```
