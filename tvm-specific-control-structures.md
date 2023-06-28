# TVM specific control structures

## Range-based for loop

Executes a `for` loop over a range. Used as a more readable equivalent to the traditional `for` loop
operating over a range of values, such as all elements in an array or mapping.

```TVMSolidity
for ( range_declaration : range_expression ) loop_statement
```

`range_expression` is calculated only once, and the result is copied. Then iteration goes over the copy of
the array or mapping.

```TVMSolidity
uint[] arr = ...;
uint sum = 0;
for (uint val : arr) { // iteration over the array
    sum += val;
}


bytes byteArray = "Hello!";
for (byte b : byteArray) { // iteration over the byte array

}

mapping(uint32 => uint) map = ...;
uint keySum = 0;
uint valueSum = 0;
for ((uint32 key, uint value) : map) { // iteration over the mapping
    keySum += key;
    valueSum += value;
}
```

Key or value can be omitted if you iterate over a mapping:

```TVMSolidity
mapping(uint32 => uint) map = ...;

uint keySum = 0;
for ((uint32 key, ) : map) { // value is omitted
    keySum += key;
}

uint valueSum = 0;
for ((, uint value) : map) { // key is omitted
    valueSum += value;
}
```

## repeat

Allows repeating block of code several times.
A `repeat` loop evaluates the expression only one time.
This expression must have an unsigned integer type.

```TVMSolidity
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

## try-catch

It is an experimental feature available only in certain blockchain networks.

The `try` statement allows you to define a block of code to be tested for errors while it is executed. The 
`catch` statement allows you to define a block of code to be executed, if an error occurs in the try block.
`catch` block gets two parameters of type variant and uint16, which contain exception argument and code respectively.
Example:

```TVMSolidity
TvmBuilder builder;
uint c = 0;
try {
    c = a + b;
    require(c != 42, 100, 22);
    require(c != 43, 100, 33);
    builder.store(c);
} catch (variant value, uint16 errorCode) {
    uint errorValue;
    if (value.isUint()) {
        errorValue = value.toUint();
    }

    if (errorCode == 100) {
        if (errorValue == 22) {
            // it was line: `require(c != 42, 100, 22);`
        } else if (errorValue == 33) {
            // it was line: `require(c != 43, 100, 33);`
        }
    } else if (errorCode == 8) {
        // Cell overflow
        // It was line: `builder.store(c);`
    } else if (errorCode == 4) {
        // Integer overflow
        // It was line: `c = a + b;`
    }
}
```