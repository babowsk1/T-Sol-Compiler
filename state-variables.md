# State variables

 Decoding state variables

You can decode state variables using tonos-cli. See `tonos-cli decode account --help`.

See also: [\<TvmSlice\>.loadStateVars()](#tvmsliceloadstatevars).

 Keyword `constant`

For `constant` variables, the value has to be a compile time constant and this value is
substituted where the variable is used. The value has to be assigned where the variable is declared.
Example:

```TVMSolidity
contract MyContract {
    uint constant cost = 100;
    uint constant cost2 = cost + 200;
    address constant dest = address.makeAddrStd(-1, 0x89abcde);
}
```

 Keyword `static`

Static state variables are used in the contract initial state generation.
Such variables can be set while deploying contract from contract
(onchain) or by tvm-linker (offchain). Example:

```TVMSolidity
contract C {
    uint static a; // ok
    uint static b = 123; // error
}
```

See also:

* [`code` option usage](#code-option-usage)
* [New contract address problem](#new-contract-address-problem)

 Keyword `public`

For each public state variable, a getter function is generated. Generated
function has the same name and return type as the public variable. This
function can be called only locally. Public state variables are useful,
because you don't need to write functions that return a particular state variable manually.

Example:

```TVMSolidity
contract C {
    uint public a;
    uint public static b; // it's ok. Variable is both public and static.
}
```

## Special contract functions

 receive

`receive` function is called in two cases:

1. [msg.data](#msgdata) (or message body) is empty.
2. [msg.data](#msgdata) starts with 32-bit zero. Then message body may contain data,
for example [string](#string) with comment.

If in the contract there is no `receive` function then the contract has an implicit empty `receive`
function.

```TVMSolidity
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

 fallback

`fallback` function is called on receiving an inbound internal/external message in such cases:

1. The message contains a function id that the contract doesn't contain.
2. Bit-length of the message is from 1 to 31 (including).
3. Bit-length of the message is equal to zero, but the message contains reference(s).

**Note**: if the message has correct function id but invalid encoded function parameters then
the transaction fail with an exception (e.g. [cell underflow exception](#tvm-exception-codes)).

If in the contract there is no fallback function then contract has implicit fallback function
that throws [exception](#solidity-runtime-errors).

Example:

```TVMSolidity
// file ContractA.sol
contract ContractA {
    uint public counter = 0;

    function f(uint a, uint b) public pure { /*...*/ }

    fallback() external {
        ++counter;
    }

}

// file ContractB.sol
import "ContractA.sol";
import "ContractAnother.sol";

contract ContractB {
    // Let's condider that `addr` is ContractA's address. 4 messages are send. ContractA's fallback
    // function will handle that messages except the last one.
    function g(address addr) public pure {
        tvm.accept();
        // The message contains a function id that the contract doesn't contain.
        // There is wrong casting to ContractAnother. `addr` is ContractA's address.
        ContractAnother(addr).sum{value: 1 ton}(2, 2);

        {
            TvmBuilder b;
            b.storeUnsigned(1, 1);
            // Bit-length of the message is equal to 20 bits.
            addr.transfer({value: 2 ton, body: b.toCell()});
        }

        {
            TvmBuilder b;
            b.storeRef(b);
            // Bit-length of the message is equal to zero but the message contains one reference.
            addr.transfer({value: 1 ton, body: b.toCell()});
        }

        TvmBuilder b;
        uint32 id = tvm.functionId(ContractA.f);
        b.store(id, uint(2));
        // ContractA's fallback function won't be called because the message body doesn't contain
        // the second ContractA.f's parameter. It will cause cell underflow exception in ContractA.
        addr.transfer({value: 1 ton, body: b.toCell()});
    }
}
```

 onBounce

```TVMSolidity
onBounce(TvmSlice body) external {
    /*...*/
}
```

`onBounce` function is executed when contract receives a bounced inbound internal message.
The message is generated by the network if the contract sends an internal message with `bounce: true` and either

* called contract doesn't exist;
* called contract fails at the storage/credit/computing phase (not at the action phase!)

The message is generated only if the remaining message value is enough for sending one back.

`body` is empty or contains at most **256** data bits of the original message (without references).
The function id takes **32** bits and parameters can take at most **224** bits.
It depends on the network config. If `onBounce` function is not defined then the contract does
nothing on receiving a bounced inbound internal message.

If the `onBounce` function throws an exception then another bounced messages are not generated.

Example of how to use `onBounce` function:

* [onBounceHandler](https://github.com/tonlabs/samples/blob/master/solidity/16_onBounceHandler.sol)

 onTickTock

`onTickTock` function is executed on tick/tock transaction.
These transactions are automatically generated for certain special accounts.
See ([TBLKCH][2] - 4.2.4.)
For tick transactions **isTock** is false, for tock transactions - true.

Prototype:

```TVMSolidity
onTickTock(bool isTock) external {
    /*...*/
}
```

 onCodeUpgrade

`onCodeUpgrade` function can have an arbitrary set of arguments and should be
executed after [tvm.setcode()](#tvmsetcode) function call. In this function
[tvm.resetStorage()](#tvmresetstorage) should be called if the set of state
variables is changed in the new version of the contract. This function implicitly
calls [tvm.commit()](#tvmcommit). `onCodeUpgrade` function does not return value. `onCodeUpgrade` function
finishes TVM execution with exit code 0.

Prototype:

```TVMSolidity
function onCodeUpgrade(...) private {
    /*...*/
}
```

Function `onCodeUpgrade` had function id = 2 (for compiler <= 0.65.0). Now, it has another id, but you can set 
`functionID(2)` in new contracts for the `onCodeUpgrade` to upgrade old ones.

See example of how to upgrade code of the contract:

* [old contract](https://github.com/tonlabs/samples/blob/master/solidity/12_BadContract.sol)
* [new contract](https://github.com/tonlabs/samples/blob/master/solidity/12_NewVersion.sol)

It's good to pass `TvmCell cell` to the public function that calls `onCodeUpgrade(TvmCell cell, ...)`
function. `TvmCell cell` may contain some data that may be useful for the new contract.

```TVMSolidity
// old contract
// Public function that changes the code and takes some cell
function updateCode(TvmCell newcode, TvmCell cell) public pure checkPubkeyAndAccept {
    tvm.setcode(newcode);
    tvm.setCurrentCode(newcode);
    // pass cell to new contract
    onCodeUpgrade(cell);
}

function onCodeUpgrade(TvmCell cell) private pure {
}


// new contract
function onCodeUpgrade(TvmCell cell) private pure {
    // new code can use cell that was passed from the old version of the contract
}
```

 afterSignatureCheck

```TVMSolidity
function afterSignatureCheck(TvmSlice body, TvmCell message) private inline returns (TvmSlice) {
    /*...*/
}
```

`afterSignatureCheck` function is used to define custom replay protection function instead of the
default one.

NB: Do not use [tvm.commit()](#tvmcommit) or [tvm.accept()](#tvmaccept) in this function as it called before the constructor call.
**time** and **expire** header fields can be used for replay protection and, if set, they should be read in `afterSignatureCheck`.
See also: [Contract execution](#contract-execution).
See an example of how to define this function:

* [Custom replay protection](https://github.com/tonlabs/samples/blob/master/solidity/14_CustomReplayProtection.sol)