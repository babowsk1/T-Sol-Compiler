# Libraries

Libraries are similar to contracts, but they can't have state variables
and can't inherit nor be inherited. Libraries can be seen as implicit
base contracts of the contracts that use them. They will not be
explicitly visible in the inheritance hierarchy, but calls of library
functions look just like calls of functions of explicit base contracts
(using qualified access like `LibName.func(a, b, c)`). There is also
another way to call library function: `obj.func(b, c)`.
For now libraries are stored as a part of the code of the contact that
uses libraries. In the future, it can be changed.

## Function call via library name

Example of using library in the manner `LibName.func(a, b, c)`:

```solidity
// file MathHelper.sol
pragma solidity >= 0.6.0;

// Library declaration
library MathHelper {
    // State variables are forbidden in library but constants are not
    uint constant MAX_VALUE = 300;

    // Library function
    function sum(uint a, uint b) internal pure returns (uint) {
        uint c = a + b;
        require(c < MAX_VALUE);
        return c;
    }
}


// file MyContract.sol
pragma solidity >= 0.6.0;

import "MathHelper.sol";

contract MyContract {
    uint s;

    function addValue(uint value) public returns (uint) {
        s = MathHelper.sum(s, value);
        return s;
    }
}
```

## Function call via object

In TON solidity **arguments of a function call passed by value not by
reference**. It's effective for numbers and even for huge arrays.
See ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.2.3.2).
**But if a library function is called like `obj.func(b, c)` then the
first argument `obj` is passed by reference.**  It's similar to
the `self` variable in Python.
The directive `using A for B;` can be used to attach library functions
(from the library `A`) to any type (`B`) in the context of the contract.
These functions will receive the object they were called for as their
first parameter.
The effect of `using A for *;` is that the functions from the library
`A` are attached to all types.

Example of using library in the manner `obj.func(b, c)`:

```solidity
// file ArrayHelper.sol
pragma solidity >= 0.6.0;

library ArrayHelper {
    // Delete value from the `array` at `index` position
    function del(uint[] array, uint index) internal pure {
        for (uint i = index; i + 1 < array.length; ++i){
            array[i] = array[i + 1];
        }
        array.pop();
    }
}


// file MyContract.sol
pragma solidity >= 0.6.0;

import "ArrayHelper.sol";

contract MyContract {
    // Attach library function `del` to the type `uint[]`
    using ArrayHelper for uint[];

    uint[] array;

    constructor() public {
        array = [uint(100), 200, 300];
    }

    function deleteElement(uint index) public {
        // Library function call via object.
        // Note: library function `del` has 2 arguments:
        // array is passed by reference and index is passed by value
        array.del(index);
    }
}
```

### Import

T-Sol Compiler allows user to import remote files using link starting with `http`.
If import file name starts with `http`, then compiler tries to download the file using this
link and saves it to the folder `.solc_imports`. If compiler fails to create this folder or
to download the file, then an error is emitted.

**Note**: to import file from GitHub, one should use link to the raw version of the file.

Example:

```solidity
pragma ton-solidity >= 0.35.0;
pragma AbiHeader expire;
pragma AbiHeader time;
pragma AbiHeader pubkey;

import "https://github.com/tonlabs/debots/raw/9c6ca72b648fa51962224ec0d7ce91df2a0068c1/Debot.sol";
import "https://github.com/tonlabs/debots/raw/9c6ca72b648fa51962224ec0d7ce91df2a0068c1/Terminal.sol";

contract HelloDebot is Debot {
  ...
}
```