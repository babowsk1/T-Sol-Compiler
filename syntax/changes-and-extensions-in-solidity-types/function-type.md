# Function type

Function types are the types of functions. Variables of function type can be assigned from functions
and function parameters of function type can be used to pass functions to and return functions from
function calls.

If unassigned variable of function type is called then exception with code 65 is thrown.

```solidity
function getSum(int a, int b) internal pure returns (int) {
    return a + b;
}

function getSub(int a, int b) internal pure returns (int) {
    return a - b;
}

function process(int a, int b, uint8 mode) public returns (int) {
    function (int, int) returns (int) fun;
    if (mode == 0) {
        fun = getSum;
    } else if (mode == 1) {
        fun = getSub;
    }
    return fun(a, b); // if `fun` isn't initialized then exception is thrown
}
```