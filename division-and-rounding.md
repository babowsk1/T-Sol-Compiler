# Division and rounding

Let consider we have `x` and `y` and we want to divide `x` by `y`. Compute the quotient `q` and the
remainder `r` of the division of `x` by `y`: `x = y*q + r` where `|r| < |y|`.

In TVM there are 3 variants of rounding:

* **floor** - quotient `q` is rounded to −∞. `q = ⌊x/y⌋`, `r` has the same sign as `y`. This
rounding variant is used for operator `/`.
Example:

```solidity
int res = int(-2) / 3; // res == -1
int res = int(-5) / 10; // res == -1
int res = int(5) / 10; // res == 0
int res = int(15) / 10; // res == 1
```

* **ceiling** - quotient `q` is rounded to +∞. `q = ⌈x/y⌉`, `r` and `y` have opposite signs.
Example:

```solidity
int res = math.divc(-2, 3); // res == 0
int res = math.divc(-5, 10); // res == 0
int res = math.divc(5, 10); // res == 1
int res = math.divc(15, 10); // res == 2
```

* **nearest** - quotient `q` is rounded to the nearest number. `q = ⌊x/y + 1/2⌋` and `|r| ≤ |y|/2`.
Example:

```solidity
int res = math.divr(-2, 3); // res == -1
int res = math.divr(-5, 10); // res == 0
int res = math.divr(5, 10); // res == 1
int res = math.divr(15, 10); // res == 2
```