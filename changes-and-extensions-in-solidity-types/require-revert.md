# Require, revert

In case of exception, state variables of the contract are reverted to the state before [tvm.commit()](require-revert.md#tvmcommit) or to the state of the contract before it was called. Use error codes that are greater than 100 because other error codes can be [reserved](require-revert.md#solidity-runtime-errors).&#x20;

{% hint style="info" %}
If a nonconstant error code is passed as the function argument and the error code is less than 2, then the error code will be set to 100.
{% endhint %}

## require

```solidity
require(bool condition, [uint errorCode = 100, [Type exceptionArgument]]);
```

**require** function can be used to check the condition and throw an exception if the condition is not met. The function takes condition and optional parameters: error code (unsigned integer) and the object of any type.

Example:

```solidity
uint a = 5;

require(a == 5); // ok
require(a == 6); // throws an exception with code 100
require(a == 6, 101); // throws an exception with code 101
require(a == 6, 101, "a is not equal to six"); // throws an exception with code 101 and string
require(a == 6, 101, a); // throws an exception with code 101 and number a
```

## revert

```solidity
revert(uint errorCode = 100, [Type exceptionArgument]);
```

**revert** function can be used to throw exceptions. The function takes an optional error code (unsigned integer) and the object of any type.

Example:

```solidity
uint a = 5;
revert(); // throw exception 100
revert(101); // throw exception 101
revert(102, "We have a some problem"); // throw exception 102 and string
revert(101, a); // throw exception 101 and number a
```
