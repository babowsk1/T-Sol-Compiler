# Optional type

The template optional type manages an optional contained value, i.e., a value that may or may not be present.

## Constructing an optional

There are many ways to set a value:

```solidity
optional(uint) opt;
opt.set(11); // just sets value
opt = 22; // just sets value, too
opt.get() = 33; // if 'opt' has value then set value. Otherwise throws an exception.

optional(uint) another = ...;
opt = another;
```

## hasValue()

```solidity
<optional(Type)>.hasValue() returns (bool);
```

Checks whether the `optional` contains a value.

## get()

```solidity
<optional(Type)>.get() returns (Type);
```

Returns the contained value, if the `optional` contains one. Otherwise, throws an exception.

## set()

```solidity
<optional(Type)>.set(Type value);
```

Replaces content of the `optional` with **value**.

## reset()

```solidity
<optional(Type)>.reset();
```

Deletes content of the `optional`.

## Keyword `null`

Keyword `null` is a constant that is used to indicate an optional type with an uninitialized state. Example:

```solidity
optional(uint) x = 123;
x = null; // reset value
```
