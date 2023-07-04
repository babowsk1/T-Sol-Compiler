# Keyword `public`

For each public state variable, a getter function is generated. Generated
function has the same name and return type as the public variable. This
function can be called only locally. Public state variables are useful,
because you don't need to write functions that return a particular state variable manually.

Example:

```solidity
contract C {
    uint public a;
    uint public static b; // it's ok. Variable is both public and static.
}
```