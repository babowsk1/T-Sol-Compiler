# Function specifiers

## Function mutability: pure, view and default

Function mutability shows how this function treats state variables.
Possible values of the function mutability:

* `pure` - function neither reads nor assigns state variables;
* `view` - function may read but doesn't assign state variables;
* default - function may read and/or assign state variables.

Example:

```TVMSolidity
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

## Keyword inline

`inline` specifier instructs the compiler to insert a copy of the private function
body into each place where the function is called.
Keyword can be used only for private and internal functions.

Example:

```TVMSolidity
// This function is called as a usual function.
function getSum(uint a, uint b) public returns (uint) {
    return sum(a, b);
}

// Code of this function is inserted in place of the call site.
function sum(uint a, uint b) private inline returns (uint) {
    return a + b;
}
```

## Assembly

To make inline assembler you should mark free function as `assembly`. Function body must contain lines of assembler code separated by commas.

It is up to user to set correct mutability (`pure`, `view` or default), return parameters of the function and so on. 

```TVMSolidity
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

```TVMSolidity
function incomingValue() assembly pure returns (uint) {
    ".blob xF82B", // it's opcode INCOMINGVALUE 
}
```

## functionID()

`functionID` keyword allows assigning function identifier explicitly.
Each public function has a unique 32-bit identifier (id). id 0 is reserved for [receive](#receive) function.
In case `functionID` is not defined explicitly, it is calculated as a hash of the function signature.
In general, there is no purpose to set the function id manually.

Example:

```TVMSolidity
function f() public pure functionID(123) {
    /*...*/
}
 ```

## externalMsg and internalMsg

Keywords `externalMsg` and `internalMsg` specify which messages the function can handle.
If the function marked by keyword `externalMsg` is called by internal message, the function throws an
exception with code 71.
If the function marked by keyword `internalMsg` is called by external message, the function throws
an exception with code 72.

Example:

```TVMSolidity
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