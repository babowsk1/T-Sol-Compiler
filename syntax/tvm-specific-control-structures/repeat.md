# Repeat

Allows repeating block of code several times.
A `repeat` loop evaluates the expression only one time.
This expression must have an unsigned integer type.

```solidity
uint a = 0;
repeat(10) {
    a ++;
}
// a == 10

// Despite a is changed in the cycle, code block will be repeated 10 times (10 is the initial value of a)
repeat(a) {
    a += 2;
}
// a == 30

a = 11;
repeat(a - 1) {
    a -= 1;
}
// a == 1
```