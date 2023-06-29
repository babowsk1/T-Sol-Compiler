# Vector

`vector(Type)` is a template container type capable of storing an arbitrary set of values of a single type, pretty much like dynamic-sized array. Two major differences are that `vector(Type)`:

1. is much more efficient than a dynamic-sized array;
2. has a lifespan of a smart-contract execution, so it can't be neither passed nor returned as an external function call parameter, nor stored in a state variable.

**Note:** `vector` implementation based on `TVM Tuple` type, and it has a limited length of 255 \* 255 = 65025 values.

## push()

```solidity
<vector(Type)>.push(Type obj);
```

Appends **obj** to the `vector`.

```solidity
vector(uint) vect;
uint a = 11;
vect.push(a);
vect.push(111);
```

## pop()

```solidity
<vector(Type)>.pop() returns (Type);
```

Pops the last value from the `vector` and returns it.

```solidity
vector(uint) vect;
...
uint a = vect.pop();
```

## length()

```solidity
<vector(Type)>.length() returns (uint8);
```

Returns length of the `vector`.

```solidity
vector(uint) vect;
...
uint8 len = vect.length();
```

## empty()

```solidity
<vector(Type)>.empty() returns (bool);
```

Checks whether the `vector` is empty.

```solidity
vector(uint) vect;
...
bool is_empty = vect.empty();
```
