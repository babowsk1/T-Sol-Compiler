# Exponentiation

Exponentiation `**` is only available for unsigned types in the exponent. The resulting type of an exponentiation is always equal to the type of the base. Please take care that it is large enough to hold the result and prepare for potential assertion failures or wrapping behaviour.

Note that `0**0` throws an exception.

Example:

```solidity
uint b = 3;
uint32 p = 4;
uint res = b ** p; // res == 81
```