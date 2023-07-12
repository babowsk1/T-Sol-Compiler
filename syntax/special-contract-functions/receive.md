# Receive

`receive` function is called in two cases:

1. [msg.data](../api-functions-and-members/msg-namespace.md#data) (or message body) is empty.
2. [msg.data](../api-functions-and-members/msg-namespace.md#data) starts with 32-bit zero. Then message body may contain data, for example [string](../changes-and-extensions-in-solidity-types/string.md) with comment.

If in the contract there is no `receive` function then the contract has an implicit empty `receive` function.

```solidity
// file sink.sol
contract Sink {
    uint public counter = 0;
    uint public msgWithPayload = 0;
    receive() external {
        ++counter;
        // if the inbound internal message has payload then we can get it using `msg.data`
        TvmSlice s = msg.data;
        if (!s.empty()) {
            ++msgWithPayload;
        }
    }
}

// file bomber.sol
contract Bomber {
    // This function send tons 3 times to the Sink contract. Sink's function receive will handle 
    // that messages.
    function f(address addr) pure public {
        tvm.accept();
        addr.transfer({value: 1 ton}); // message's body is empty

        TvmBuilder b;
        addr.transfer({value: 1 ton, body: b.toCell()}); // message's body is empty, too

        b.store(uint32(0), "Thank you for the coffee!");
        // body of the message contains 32-bit zero number and the string
        addr.transfer({value: 20 ton, body: b.toCell()});
    }
}
```
