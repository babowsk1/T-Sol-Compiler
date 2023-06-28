# Events and return

## emit

`emit` statement sends an external outbound message. Use `{dest: ...}`to set destination address.
The address must be of **addr_extern** type.

If option `dest` is not used, the destination address is set to **addr_none**.

**Note:** fee for creating the external outbound message is withdrawn from the contract's
balance even if the contract has been called by internal inbound message.

Example:

```TVMSolidity
event SomethingIsReceived(uint a, uint b, uint sum);
...
address addr = address.makeAddrExtern(...);
emit SomethingIsReceived{dest: addr}(2, 8, 10); // dest address is set
emit SomethingIsReceived(10, 15, 25); // dest address == addr_none
```

## return

`return` statement has different effects depending on:

* visibility of the function;
* type of the inbound message;
* presence of the `responsible` modifier.

In `private` or `internal` visible functions, `return` assigns the return variables to the specified parameters;
In `public` or `external` visible functions, `return` statement:

* on an external inbound message, generates an external outbound message with a destination address set to
the source address of the inbound (external) message. All arguments of the return statement are ignored.
* on an external inbound message:
  * if the function is marked as `responsible`, generates an internal outbound message;
  * otherwise has no effect.

`value`, `bounce`, `flag` and `currencies` options are used to create the message. Some options can
be omitted. See [\<address\>.transfer()](#addresstransfer) where these options and their default
values are described.

**Note**: if the function `f` below is called with `n = 5` and internal/external message must be
generated then only one message is sent with result `120` (120 = 5 * 4 * 3 * 2 * 1).

**Hint:** Use `{value: 0, bounce: false, flag: 64}` for `responsible` function.
Because `flag: 0` is used by default.

See also: [External function calls](#external-function-calls).

```TVMSolidity
function f(uint n) public pure {
    return n <= 1 ? 1 : n * f(n - 1);
}

function f(uint n) public responsible pure {
    return{value: 0, bounce: false, flag: 64} n <= 1 ? 1 : n * f(n - 1);
}
```

