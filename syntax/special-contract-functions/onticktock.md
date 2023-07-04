# onTickTock

`onTickTock` function is executed on tick/tock transaction.
These transactions are automatically generated for certain special accounts.
For tick transactions **isTock** is false, for tock transactions - true.

Prototype:

```solidity
onTickTock(bool isTock) external {
    /*...*/
}
```