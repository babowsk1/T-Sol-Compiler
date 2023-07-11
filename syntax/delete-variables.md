# Delete variables

As in classic Solidity `delete` operation assigns the initial value for the type to a variable.
Delete operation can be applied not only to variables itself, but to its fields or index values.

Example:

```solidity
int a = 5;
delete a; // a == 0

uint[] arr;
arr.push(11);
arr.push(22);
arr.push(33);

delete arr[1];
// arr[0] == 11
// arr[1] == 0
// arr[2] == 33

delete arr; // arr.length == 0

mapping(uint => uint) l_map;
l_map[1] = 2;
delete l_map[1];
bool e = l_map.exists(1); // e == false
l_map[1] = 2;
delete l_map;
bool e = l_map.exists(1); // e == false

struct DataStruct {
    uint m_uint;
    bool m_bool;
}

DataStruct l_struct;
l_struct.m_uint = 1;
delete l_struct; // l_struct.m_uint == 0

TvmBuilder b;
uint i = 0x54321;
b.store(i);
TvmCell c = b.toCell();
delete c;
TvmCell empty; // tvm.hash(empty) == tvm.hash(c)
b.store(c);

TvmSlice slice = b.toSlice();
uint16 bits = slice.bits(); // bits == 256
uint8 refs = slice.refs(); // refs == 1
delete slice;
uint16 bits = slice.bits(); // bits == 256
uint8 refs = slice.refs(); // refs == 1


uint16 bits = b.bits(); // bits == 256
uint8 refs = b.refs(); // refs == 1
delete b;
uint16 bits = b.bits(); // bits == 256
uint8 refs = b.refs(); // refs == 1
```