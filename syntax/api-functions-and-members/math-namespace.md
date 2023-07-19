# Math namespace

## min() max()

```solidity
math.min(T a, T b, ...) returns (T);
math.max(T a, T b, ...) returns (T);
```

Returns the minimal (maximal) value of the passed arguments. `T` should be an integer or fixed point type.

## minmax()

```solidity
math.minmax(T a, T b) returns (T /*min*/, T /*max*/);
```

Returns minimal and maximal values of the passed arguments.`T` should be an integer or fixed point type.

Example:

```solidity
(uint a, uint b) = math.minmax(20, 10); // (a, b) == (10, 20)
```

## abs()

```solidity
math.abs(intM val) returns (intM);
math.abs(fixedMxN val) returns (fixedMxN);
```

Computes the absolute value of the given integer.

Example:

```solidity
int a = math.abs(-4123); // 4123
int b = -333;
int c = math.abs(b); // 333
```

## modpow2()

```solidity
math.modpow2(uint value, uint power) returns (uint);
```

Computes the `value mod 2^power`. Note: `power` should be a constant integer.

Example:

```solidity
uint constant pow = 12;
uint val = 12313;
uint a = math.modpow2(val, 10);
uint b = math.modpow2(val, pow);
```

## divr() math.divc()

```solidity
math.divc(T a, T b) returns (T);
math.divr(T a, T b) returns (T);
```

Returns result of the division of two integers. `T` should be an integer or fixed point type.
The return value is rounded. **ceiling** and **nearest** modes are used for `divc` and `divr`
respectively.       
See also: [Division and rounding](../division-and-rounding.md).

Example:

```solidity
int c = math.divc(10, 3); // c == 4
int c = math.divr(10, 3); // c == 3

fixed32x2 a = 0.25;
fixed32x2 res = math.divc(a, 2); // res == 0.13
```

## muldiv() muldivr() muldivc()

```solidity
math.muldiv(T a, T b, T c) returns (T);
math.muldivr(T a, T b, T c) returns (T);
math.muldivc(T a, T b, T c) returns (T);
```

Multiplies two values and then divides the result by a third value. `T` is integer type.
The return value is rounded. **floor**, **ceiling** and **nearest** modes are used for `muldiv`,
`muldivc` and `muldivr` respectively.
See also: [Division and rounding](../division-and-rounding.md).

Example:

```solidity
uint res = math.muldiv(3, 7, 2); // res == 10
uint res = math.muldivr(3, 7, 2); // res == 11
uint res = math.muldivc(3, 7, 2); // res == 11
```

## muldivmod()

```solidity
math.muldivmod(T a, T b, T c) returns (T /*result*/, T /*remainder*/);
```

This instruction multiplies first two arguments, divides the result by third argument and returns the result and the remainder. Intermediate result is stored in the 514 bit buffer, and the final result is rounded to the floor.

Example:

```solidity
uint a = 3;
uint b = 2;
uint c = 5;
(uint d, uint r) = math.muldivmod(a, b, c); // (1, 1)
int e = -1;
int f = 3;
int g = 2;
(int h, int p) = math.muldivmod(e, f, g); // (-2, 1)
```

## divmod()

```solidity
math.divmod(T a, T b) returns (T /*result*/, T /*remainder*/);
```

This function divides the first number by the second and returns the result and the remainder. Result is rounded to the floor. `T` must be an integer type.

Example:

```solidity
uint a = 3;
uint b = 2;
(uint d, uint r) = math.divmod(a, b); //

int e = -11;
int f = 3;
(int h, int p) = math.divmod(e, f); //
```

## sign()

```solidity
math.sign(int val) returns (int8);
```

Returns the number in case of the sign of the argument value `val`:

* \-1 if `val` is negative;
* 0 if `val` is zero;
* 1 if `val` is positive.

Example:

```solidity
int8 sign = math.sign(-100); // sign == -1
int8 sign = math.sign(100); // sign == 1
int8 sign = math.sign(0); // sign == 0
```
