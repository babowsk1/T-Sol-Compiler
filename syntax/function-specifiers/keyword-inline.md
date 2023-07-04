# Keyword inline

`inline` specifier instructs the compiler to insert a copy of the private function
body into each place where the function is called.
Keyword can be used only for private and internal functions.

Example:

```solidity
// This function is called as a usual function.
function getSum(uint a, uint b) public returns (uint) {
    return sum(a, b);
}

// Code of this function is inserted in place of the call site.
function sum(uint a, uint b) private inline returns (uint) {
    return a + b;
}
```