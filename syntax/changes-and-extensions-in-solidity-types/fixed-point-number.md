# Fixed point number

`fixed` / `ufixed`: Signed and unsigned fixed point number of various sizes. Keywords `ufixedMxN`
and `fixedMxN`, where `M` represents the number of bits taken by the type and `N` represents how
many decimal points are available. `M` must be divisible by 8 and goes from 8 to 256 bits. `N` must
be between 0 and 80, inclusive. `ufixed` and `fixed` are aliases for `ufixed128x18` and
`fixed128x18`, respectively.

Operators:

* Comparison: `<=`, `<`, `==`, `!=`, `>=`, `>` (evaluate to `bool`)
* Arithmetic operators: `+`, `-`, unary `-`, `*`, `/`, `%` (modulo)
* Math operations: [math.min(), math.max()](#mathmin-mathmax), [math.minmax()](#mathminmax),
[math.abs()](#mathabs), [math.divr(), math.divc()](#mathdivr-mathdivc)