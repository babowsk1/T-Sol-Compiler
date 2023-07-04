# Ever-solidity

Used to restrict source file compilation to the particular compiler versions.

```solidity
pragma ever-solidity >= 0.35.5;      // Check if the compiler version is greater or equal than 0.35.5
pragma ever-solidity ^ 0.35.5;       // Check if the compiler version is greater or equal than 0.35.5 and less than 0.36.0
pragma ever-solidity < 0.35.5;       // Check if the compiler version is less than 0.35.5
pragma ever-solidity >= 0.35.5 < 0.35.7; // Check if the compiler version is equal to either 0.35.5 or 0.35.6
```
