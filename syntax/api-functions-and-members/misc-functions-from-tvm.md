# Misc functions from tvm

## code()

```solidity
tvm.code() returns (TvmCell);
```

Returns contract's code.

See [SelfDeployer](https://github.com/tonlabs/samples/blob/master/solidity/21\_self\_deploy.sol).

## codeSalt()

```solidity
tvm.codeSalt(TvmCell code) returns (optional(TvmCell) optSalt);
```

If **code** contains salt then **optSalt** contains one. Otherwise, **optSalt** doesn't contain any value.

## setCodeSalt()

```solidity
tvm.setCodeSalt(TvmCell code, TvmCell salt) returns (TvmCell newCode);
```

Inserts **salt** into **code** and returns new code **newCode**.

## pubkey()

```solidity
tvm.pubkey() returns (uint256);
```

Returns contract's public key, stored in contract data. If key is not set, function returns 0.

## setPubkey()

```solidity
tvm.setPubkey(uint256 newPubkey);
```

Set new contract's public key. Contract's public key can be obtained from `tvm.pubkey`.

## setCurrentCode()

```solidity
tvm.setCurrentCode(TvmCell newCode);
```

Changes this smart contract current code to that given by Cell **newCode**. Unlike [tvm.setcode()](tvm-namespace.md#setcode) this function changes code of the smart contract only for current TVM execution, but has no effect after termination of the current run of the smart contract.

See example of how to use this function:

* [old contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_BadContract.sol)
* [new contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_NewVersion.sol)

## resetStorage()

```solidity
tvm.resetStorage();
```

Resets all state variables to their default values.

## functionId()

```solidity
// id of public function
tvm.functionId(functionName) returns (uint32);

// id of public constructor
tvm.functionId(ContractName) returns (uint32);
```

Returns the function id (uint32) of a public/external function or constructor.

Example:

```solidity
contract MyContract {
    constructor(uint a) public {
    }
        /*...*/
    }

    function f() public pure returns (uint) {
        /*...*/
    }

    function getConstructorID() public pure returns (uint32) {
        uint32 functionId = tvm.functionId(MyContract);
        return functionId;
    }

    function getFuncID() public pure returns (uint32) {
        uint32 functionId = tvm.functionId(f);
        return functionId;
    }
}
```

See example of how to use this function:

* [onBounceHandler](https://github.com/tonlabs/samples/blob/master/solidity/16\_onBounceHandler.sol)

## encodeBody()

```solidity
tvm.encodeBody(function, arg0, arg1, arg2, ...) returns (TvmCell);
tvm.encodeBody(function, callbackFunction, arg0, arg1, arg2, ...) returns (TvmCell);
tvm.encodeBody(contract, arg0, arg1, arg2, ...) returns (TvmCell);
```

Constructs a message body for the function call. Body can be used as a payload for [\<address>.transfer()](../changes-and-extensions-in-solidity-types/functions.md#transfer). If the **function** is `responsible`, **callbackFunction** must be set.

Example:

```solidity
contract Remote {
    constructor(uint x, uint y, uint z) public { /* */ }
    function func(uint256 num, int64 num2) public pure { /* */ }
    function getCost(uint256 num) public responsible pure returns (uint128) { /* */ }
}

// deploy the contract
TvmCell body = tvm.encodeBody(Remote, 100, 200, 300);
addr.transfer({value: 10 ton, body: body, stateInit: stateInit });

// call the function
TvmCell body = tvm.encodeBody(Remote.func, 123, -654);
addr.transfer({value: 10 ton, body: body});

// call the responsible function
TvmCell body = tvm.encodeBody(Remote.getCost, onGetCost, 105);
addr.transfer({value: 10 ton, body: body});
```

See also:

* [External function calls](../external-function-calls/)
* [\<TvmSlice>.loadFunctionParams()](../tvm-specific-types/tvmslice.md#loadfunctionparams)
* [tvm.buildIntMsg()](misc-functions-from-tvm.md#buildintmsg)

## exit() and tvm.exit1()

```solidity
tvm.exit();
tvm.exit1();
```

Functions are used to save state variables and quickly terminate execution of the smart contract. Exit codes are equal to zero and one for `tvm.exit` and `tvm.exit1` respectively.

Example:

```solidity
function g0(uint a) private {
    if (a == 0) {
        tvm.exit();
    }
    //...
}

function g1(uint a) private {
    if (a == 0) {
        tvm.exit1();
    }
    //...
}
```

## buildExtMsg()

```solidity
tvm.buildExtMsg({
    dest: address,
    time: uint64,
    expire: uint32,
    call: {functionIdentifier [, list of function arguments]},
    sign: bool,
    pubkey: optional(uint256),
    callbackId: (uint32 | functionIdentifier),
    onErrorId: (uint32 | functionIdentifier),
    stateInit: TvmCell,
    signBoxHandle: optional(uint32),
    abiVer: uint8,
    flags: uint8
})
returns (TvmCell);
```

Function should be used only offchain and intended to be used only in debot contracts. Allows creating an external inbound message, that calls the **func** function of the contract on address **destination** with specified function arguments.

Mandatory parameters that are used to form a src field that is used for debots:

* `callbackId` - identifier of the callback function.
* `onErrorId` - identifier of the function that is called in case of an error.
* `signBoxHandle` - handle of the sign box entity, that engine will use to sign the message.

These parameters are stored in addr\_extern and placed to the src field of the message. Message is of type [ext\_in\_msg\_info](https://github.com/ton-blockchain/ton/blob/24dc184a2ea67f9c47042b4104bbb4d82289fac1/crypto/block/block.tlb#L127) and src address is of type [addr\_extern](https://github.com/ton-blockchain/ton/blob/24dc184a2ea67f9c47042b4104bbb4d82289fac1/crypto/block/block.tlb#L101) but stores special data:

* callback id - 32 bits;
* on error id - 32 bits;
* abi version - 8 bits; Can be specified manually and contain full abi version in little endian half bytes (e.g. version = "2.3" -> abiVer: 0x32)
* header mask - 3 bits in such order: time, expire, pubkey;
* optional value signBoxHandle - 1 bit (whether value is present) + \[32 bits];
* control flags byte - 8 bits. Currently used bits: 1 - override time (dengine will replace time value with current time) 2 - override exp (dengine will replace time value with actual expire value) 4 - async call (dengine must send message and don't wait for the result)

Other function parameters define fields of the message:

* `time` - message creation timestamp. Used for replay attack protection, encoded as 64 bit Unix time in milliseconds.
* `expire` - Unix time (in seconds, 32 bit) after that message should not be processed by contract.
* `pubkey` - public key from key pair used for signing the message body. This parameter is optional and can be omitted.
* `sign` - constant bool flag that shows whether message should contain signature. If set to **true**, message is generated with signature field filled with zeroes. This parameter is optional and can be omitted (in this case is equal to **false**).

User can also attach stateInit to the message using `stateInit` parameter.

Function throws an exception with code 64 if function is called with wrong parameters (pubkey is set and has value, but sign is false or omitted).

Example:

```solidity

interface Foo {
    function bar(uint a, uint b) external pure;
}

contract Test {

    TvmCell public m_cell;

    function generate0() public {
        tvm.accept();
        address addr = address.makeAddrStd(0, 0x0123456789012345678901234567890123456789012345678901234567890123);
        m_cell = tvm.buildExtMsg({callbackId: 0, onErrorId: 0, dest: addr, time: 0x123, expire: 0x12345, call: {Foo.bar, 111, 88}});
    }

    function generate1() public {
        tvm.accept();
        optional(uint) pubkey;
        address addr = address.makeAddrStd(0, 0x0123456789012345678901234567890123456789012345678901234567890123);
        m_cell = tvm.buildExtMsg({callbackId: 0, onErrorId: 0, dest: addr, time: 0x123, expire: 0x12345, call: {Foo.bar, 111, 88}, pubkey: pubkey});
    }

    function generate2() public {
        tvm.accept();
        optional(uint) pubkey;
        pubkey.set(0x95c06aa743d1f9000dd64b75498f106af4b7e7444234d7de67ea26988f6181df);
        address addr = address.makeAddrStd(0, 0x0123456789012345678901234567890123456789012345678901234567890123);
        optional(uint32) signBox;
        signBox.set(0x12333112);
        m_cell = tvm.buildExtMsg({callbackId: 0, onErrorId: 0, dest: addr, time: 0x1771c58ef9a, expire: 0x600741e4, call: {Foo.bar, 111, 88}, pubkey: pubkey, sign: true, signBoxHandle: signBox});
    }

}
```

External inbound message can also be built and sent with construction similar to remote contract call. It requires suffix ".extMsg" and call options similar to `buildExtMsg` function call. **Note**: this type of call should be used only offchain in debot contracts.

```solidity
interface Foo {
    function bar(uint a, uint b) external pure;
}

contract Test {

    function test7() public {
        address addr = address.makeAddrStd(0, 0x0123456789012345678901234567890123456789012345678901234567890123);
        Foo(addr).bar{expire: 0x12345, time: 0x123}(123, 45).extMsg;
        optional(uint) pubkey;
        optional(uint32) signBox;
        Foo(addr).bar{expire: 0x12345, time: 0x123, pubkey: pubkey}(123, 45).extMsg;
        Foo(addr).bar{expire: 0x12345, time: 0x123, pubkey: pubkey, sign: true}(123, 45).extMsg;
        pubkey.set(0x95c06aa743d1f9000dd64b75498f106af4b7e7444234d7de67ea26988f6181df);
        Foo(addr).bar{expire: 0x12345, time: 0x123, pubkey: pubkey, sign: true}(123, 45).extMsg;
        Foo(addr).bar{expire: 0x12345, time: 0x123, sign: true, signBoxHandle: signBox}(123, 45).extMsg;
        Foo(addr).bar{expire: 0x12345, time: 0x123, sign: true, signBoxHandle: signBox, abiVer: 0x32, flags: 0x07}(123, 45).extMsg;
    }
}
```

## buildIntMsg()

```solidity
tvm.buildIntMsg({
    dest: address,
    value: uint128,
    call: {function, [callbackFunction,] arg0, arg1, arg2, ...},
    bounce: bool,
    currencies: ExtraCurrencyCollection
    stateInit: TvmCell
})
returns (TvmCell);
```

Generates an internal outbound message that contains a function call. The result `TvmCell` can be used to send a message using [tvm.sendrawmsg()](misc-functions-from-tvm.md#sendrawmsg). If the `function` is `responsible` then `callbackFunction` parameter must be set.

`dest`, `value` and `call` parameters are mandatory. Another parameters can be omitted. See [\<address>.transfer()](../changes-and-extensions-in-solidity-types/functions.md#transfer) where these options and their default values are described.

See also:

* sample [22\_sender.sol](https://github.com/tonlabs/samples/blob/master/solidity/22\_sender.sol)
* [tvm.encodeBody()](misc-functions-from-tvm.md#encodebody)

## sendrawmsg()

```solidity
tvm.sendrawmsg(TvmCell msg, uint8 flag);
```

Send the internal/external message `msg` with `flag`. It's a wrapper for opcode `SENDRAWMSG` ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.10). Internal message `msg` can be generated by [tvm.buildIntMsg()](misc-functions-from-tvm.md#buildintmsg). Possible values of `flag` are described here: [\<address>.transfer()](../changes-and-extensions-in-solidity-types/functions.md#transfer).

**Note:** make sure that `msg` has a correct format and follows the \[TL-B scheme]\[3] of `Message X`. For example:

```solidity
TvmCell msg = ...
tvm.sendrawmsg(msg, 2);
```

If the function is called by external message and `msg` has a wrong format (for example, the field `init` of `Message X` is not valid) then the transaction will be replayed despite the usage of flag 2. It will happen because the transaction will fail at the action phase.
