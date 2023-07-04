# Integers

``int`` / ``uint``: Signed and unsigned integers of various sizes. Keywords ``uintN`` and ``intN``
where ``N`` is a number from ``1``  to ``256`` in steps of 1 denotes the number of bits. ``uint`` and ``int``
are aliases for ``uint256`` and ``int256``, respectively.

Operators:

* Comparison: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (evaluate to ``bool``)
* Bit operators: ``&``, ``|``, ``^`` (bitwise exclusive or), ``~`` (bitwise negation)
* Shift operators: ``<<`` (left shift), ``>>`` (right shift)
* Arithmetic operators: ``+``, ``-``, unary ``-``, ``*``, ``/``, ``%`` (modulo), ``**`` (exponentiation)

## bitSize() and uBitSize()

```solidity
bitSize(int x) returns (uint16)
uBitSize(uint x) returns (uint16)
```

`bitSize` computes the smallest `c` ≥ 0 such that `x` fits into a `c`-bit signed integer
(−2<sup>c−1</sup> ≤ x < 2<sup>c−1</sup>).

`uBitSize` computes the smallest `c` ≥ 0 such that `x` fits into a `c`-bit unsigned integer
(0 ≤ x < 2<sup>c</sup>).

Example:

```solidity
uint16 s = bitSize(12); // s == 5
uint16 s = bitSize(1); // s == 2
uint16 s = bitSize(-1); // s == 1
uint16 s = bitSize(0); // s == 0

uint16 s = uBitSize(10); // s == 4
uint16 s = uBitSize(1); // s == 1
uint16 s = uBitSize(0); // s == 0
```