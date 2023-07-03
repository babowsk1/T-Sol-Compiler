# Range-based for loop

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