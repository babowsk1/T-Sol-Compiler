# ExternalMsg and internalMsg

Keywords `externalMsg` and `internalMsg` specify which messages the function can handle.
If the function marked by keyword `externalMsg` is called by internal message, the function throws an
exception with code 71.
If the function marked by keyword `internalMsg` is called by external message, the function throws
an exception with code 72.

Example:

```solidity
function f() public externalMsg { // this function receives only external messages
    /*...*/
}

// Note: keyword `external` specifies function visibility
function ff() external externalMsg { // this function also receives only external messages
    /*...*/
}

function g() public internalMsg { // this function receives only internal messages
    /*...*/
}

// These function receives both internal and external messages.
function fun() public { /*...*/ }
```