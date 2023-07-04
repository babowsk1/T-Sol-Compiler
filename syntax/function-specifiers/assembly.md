# Assembly

To make inline assembler you should mark free function as `assembly`. Function body must contain lines of assembler code separated by commas.

It is up to user to set correct mutability (`pure`, `view` or default), return parameters of the function and so on. 

```solidity
function checkOverflow(uint a, uint b) assembly pure returns (bool) {
    "QADD",
    "ISNAN",
    "NOT",
}

contract Contract {
    function f(uint a, uint b) private {
        bool ok =  checkOverflow(a, b);
        if (ok) {
            uint c = a + b;
        }
    }
}
```

You can use inline assembler to support new opcodes in experimental or another implementations of TVM. 

```solidity
function incomingValue() assembly pure returns (uint) {
    ".blob xF82B", // it's opcode INCOMINGVALUE 
}
```