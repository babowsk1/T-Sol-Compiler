# Function mutability: pure, view and default

Function mutability shows how this function treats state variables.
Possible values of the function mutability:

* `pure` - function neither reads nor assigns state variables;
* `view` - function may read but doesn't assign state variables;
* default - function may read and/or assign state variables.

Example:

```solidity
contract Test {

    uint a;

    event MyEvent(uint val);

    // pure mutability
    function f() public pure {
        emit MyEvent(123);
    }

    // view mutability
    function g() public view {
        emit MyEvent(a);
    }

    // default mutability (not set)
    function e(uint newA) public {
        a = newA;
    }
}
```