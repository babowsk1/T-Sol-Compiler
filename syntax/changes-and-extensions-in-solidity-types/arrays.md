# Arrays

## Array literals

An array literal is a comma-separated list of one or more expressions, enclosed in square brackets. For example: `[100, 200, 300]`.

Initializing constant state variable:&#x20;

```solidity
uint[] constant fib = [uint(2), 3, 5, 8, 12, 20, 32];
```

## Creating new arrays

```solidity
uint[] arr; // create 0-length array

uint[] arr = new uint[](0); // create 0-length array

uint constant N = 5;
uint[] arr = new uint[](N); // create 5-length array

uint[] arr = new uint[](10); // create 10-length array
```

Note: If `N` is constant expression or integer literal then the complexity of array creation - `O(1)`. Otherwise, `O(N)`.

## empty()

```solidity
<array>.empty() returns (bool);
```

Returns status flag whether the `array` is empty (its length is 0).

Example:

```solidity
uint[] arr;
bool b = arr.empty(); // b == true
arr.push(...);
bool b = arr.empty(); // b == false
```
