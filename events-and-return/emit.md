# emit

`emit` statement sends an external outbound message. Use `{dest: ...}`to set destination address.
The address must be of **addr_extern** type.

If option `dest` is not used, the destination address is set to **addr_none**.

**Note:** fee for creating the external outbound message is withdrawn from the contract's
balance even if the contract has been called by internal inbound message.

Example:

```solidity
event SomethingIsReceived(uint a, uint b, uint sum);
...
address addr = address.makeAddrExtern(...);
emit SomethingIsReceived{dest: addr}(2, 8, 10); // dest address is set
emit SomethingIsReceived(10, 15, 25); // dest address == addr_none
```