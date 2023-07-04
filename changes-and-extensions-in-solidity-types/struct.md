# struct

Structs are custom defined types that can group several variables.

## struct constructor

```solidity
struct Stakes {
    uint total;
    mapping(uint => uint) stakes;
}

// create struct with default values of its members
Stakes stakes;

// create struct using parameters
mapping(uint => uint) values;
values[100] = 200;
Stakes stakes = Stakes(200, values);

// create struct using named parameters
Stakes stakes = Stakes({stakes: values, total: 200});
```

## unpack()

```solidity
<struct>.unpack() returns (TypeA /*a*/, TypeB /*b*/, ...);
```

Unpacks all members stored in the `struct`.

Example:

```solidity
struct MyStruct {
    uint a;
    int b;
    address c;
}

function f() pure public {
    MyStruct s = MyStruct(1, -1, address(2));
    (uint a, int b, address c) = s.unpack();
}
```