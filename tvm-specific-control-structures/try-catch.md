# try-catch

It is an experimental feature available only in certain blockchain networks.

The `try` statement allows you to define a block of code to be tested for errors while it is executed. The 
`catch` statement allows you to define a block of code to be executed, if an error occurs in the try block.
`catch` block gets two parameters of type variant and uint16, which contain exception argument and code respectively.
Example:

```Solidity
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