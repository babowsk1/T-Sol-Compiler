# API functions and memebers

## Type information

The expression `type(T)` can be used to retrieve information about the type T.

The following properties are available for an integer, variable integer and enum type `T`:

* `type(T).min` - the smallest value representable by type `T`.
* `type(T).max` - the largest value representable by type `T`.

## **msg** namespace

### msg.sender

```solidity
msg.sender (address)
```

Returns:

* sender of the message for internal message.
* address(0) for external message.
* address(0) for tick/tock transaction.

### msg.value

```solidity
msg.value (uint128)
```

Returns:

* Balance of the inbound message in nanotons for internal message.
* 0 for external message.
* Undefined value for tick/tock transaction.

### msg.currencies

```solidity
msg.currencies (ExtraCurrencyCollection)
```

Collections of arbitrary currencies contained in the balance of the inbound message.

### msg.pubkey()

```solidity
msg.pubkey() returns (uint256);
```

Returns public key that is used to check the message signature. If the message isn't signed then it's equal to `0`. See also: [Contract execution](api-functions-and-members.md#contract-execution), [pragma AbiHeader](api-functions-and-members.md#pragma-abiheader).

### msg.isInternal, msg.isExternal and msg.isTickTock

Returns flag whether the contract is called by internal message, external message or by tick/tock transactions.

### msg.createdAt

```solidity
msg.createdAt (uint32)
```

Returns the field **created\_at** of the external inbound message.

### msg.data

```solidity
msg.data (TvmSlice)
```

Returns the payload of an inbound message.

### msg.hasStateInit

```solidity
msg.hasStateInit (bool)
```

Whether the internal/external inbound message contains field `stateInit`. Returns undefined value for tick/tock transaction. See \[TL-B scheme]\[3] of `Message X`.

## **tvm** namespace

### TVM instructions

### tvm.accept()

```solidity
tvm.accept();
```

Executes TVM instruction "ACCEPT" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.2). This instruction sets current gas limit to its maximal allowed value. This action is required to process external messages that bring no value.

See example of how to use this function:

* [accumulator](https://github.com/tonlabs/samples/blob/master/solidity/1\_Accumulator.sol)

### tvm.setGasLimit()

```solidity
tvm.setGasLimit(uint g);
```

Executes TVM instruction "SETGASLIMIT" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.2). Sets current gas limit **gl** to the minimum of **g** and **gm**, and resets the gas credit **gc** to zero. If the gas consumed so far (including the present instruction) exceeds the resulting value of **gl**, an (unhandled) out of gas exception is thrown before setting new gas limits. Notice that `tvm.setGasLimit(...)` with an argument **g** ≥ 263 - 1 is equivalent to `tvm.accept()`. `tvm.setGasLimit()` is similar to `tvm.accept()`. `tvm.accept()` sets gas limit **gl** to the maximum possible value (depends on the network configuration parameters, usually is equal to 1\_000\_000 units of gas). `tvm.setGasLimit()` is generally used for accepting external messages and restricting max possible gas consumption. It may be used to protect from flood by "bad" owner in a contract that is used by multiple users. Let's consider some scenario:

1. Check whether msg.pubkey() != 0 and msg.pubkey() belongs to the list of trusted public keys;
2. Check whether `m_floodCounter[msg.pubkey()] < 5` where **m\_floodCounter** is count of pending operations of `msg.pubkey()` user.
3. `tvm.setGasLimit(75_000);` accept external message and set gas limit to 75\_000.
4. `++m_floodCounter[msg.pubkey()];` increase count of pending operations for current users.
5. `tvm.commit();` save current state if it needs
6. Do other things.

So if some user's public key will be stolen, then a hacker can spam with external messages and burn at most `5 * 75_000` units of gas instead of `5 * 1_000_000`, because we use `tvm.setGasLimit()` instead of `tvm.accept()`.

### tvm.buyGas()

```solidity
tvm.buyGas(uint value);
```

Computes the amount of gas that can be bought for `value` nanotons, and sets **gl**\
accordingly in the same way as [tvm.setGasLimit()](api-functions-and-members.md#tvmsetgaslimit).

### tvm.commit()

```solidity
tvm.commit();
```

Creates a "check point" of the state variables (by copying them from c7 to c4) and register c5. If the contract throws an exception at the computing phase then the state variables and register c5 will roll back to the "check point", and the computing phase will be considered "successful". If contract doesn't throw an exception, it has no effect.

### tvm.rawCommit()

```solidity
tvm.rawCommit();
```

Same as [tvm.commit()](api-functions-and-members.md#tvmcommit) but doesn't copy the state variables from c7 to c4. It's a wrapper for opcode `COMMIT`. See [TVM](https://broxus.gitbook.io/threaded-virtual-machine/).

**Note**: Don't use `tvm.rawCommit()` after `tvm.accept()` in processing external messages because you don't save from c7 to c4 the hidden state variable `timestamp` that is used for replay protection.

### tvm.getData()

```solidity
tvm.getData() returns (TvmCell);
```

**Note:** Function is experimental.

A dual of the `tvm.setData()`function. It returns value of the `c4` register. Obtaining a raw storage cell can be useful when upgrading a new version of the contract that introduces an altered data layout.

Manipulation with a raw storage cell requires understanding of the way the compiler stores the data. Refer to the description of `tvm.setData()` below to get more details.

**Note:** state variables and replay protection timestamp stored in the data cell have the same values that were before the transaction. See [tvm.commit()](api-functions-and-members.md#tvmcommit) to learn about register `c4` update.

### tvm.setData()

```solidity
tvm.setData(TvmCell data);
```

**Note:** Function is experimental.

Stores cell `data` in the register `c4`. Mind that after returning from a public function all state variables from `c7` are copied to `c4` and `tvm.setData` will have no effect. Example hint, how to set `c4`:

```solidity
TvmCell data = ...;
tvm.setData(data); // set register c4
tvm.rawCommit();   // save register c4 and c5
revert(200);       // throw the exception to terminate the transaction
```

Be careful with the hidden state variable `timestamp` and think about possibility of external messages replaying.

### tvm.log()

```solidity
tvm.log(string log);
logtvm(string log);
```

Dumps `log` string. This function is a wrapper for TVM instructions `PRINTSTR` (for constant literal strings shorter than 16 symbols) and `STRDUMP` (for other strings). `logtvm` is an alias for `tvm.log(string)`. Example:

```solidity
tvm.log("Hello, world!");
logtvm("99_Bottles");

string s = "Some_text";
tvm.log(s);
```

**Note:** For long strings dumps only the first 127 symbols.

### tvm.hexdump() and tvm.bindump()

```solidity
tvm.hexdump(T a);
tvm.bindump(T a);
```

Dumps cell data or integer. Note that for cells this function dumps data only from the first cell. `T` must be an integer type or TvmCell.

Example:

```solidity
TvmBuilder b;
b.storeUnsigned(0x9876543210, 40);
TvmCell c = b.toCell();
tvm.hexdump(c);
tvm.bindump(c);
uint a = 123;
tvm.hexdump(a);
tvm.bindump(a);
int b = -333;
tvm.hexdump(b);
tvm.bindump(b);
```

Expected output for the example:

```solidityLog
CS<9876543210>(0..40)
CS<10011000011101100101010000110010000100001>(0..40)
7B
1111011
-14D
-101001101
```

### tvm.setcode()

```solidity
tvm.setcode(TvmCell newCode);
```

This command creates an output action that would change this smart contract code to that given by the `TvmCell` **newCode** (this change will take effect only after the successful termination of the current run of the smart contract).

See example of how to use this function:

* [old contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_BadContract.sol)
* [new contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_NewVersion.sol)

### tvm.configParam()

```solidity
tvm.configParam(uint8 paramNumber) returns (TypeA a, TypeB b, ...);
```

Executes TVM instruction "CONFIGPARAM" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.4. - F832). This command returns the value of the global configuration parameter with integer index **paramNumber**. Argument should be an integer literal. Supported **paramNumbers**: 1, 15, 17, 34.

### tvm.rawConfigParam()

```solidity
tvm.rawConfigParam(uint8 paramNumber) returns (TvmCell cell, bool status);
```

Executes TVM instruction "CONFIGPARAM" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.4. - F832). Returns the value of the global configuration parameter with integer index **paramNumber** as a `TvmCell` and a boolean status.

### tvm.rawReserve()

```solidity
tvm.rawReserve(uint value, uint8 flag);
tvm.rawReserve(uint value, ExtraCurrencyCollection currency, uint8 flag);
```

Creates an output action that reserves **reserve** nanotons. It is roughly equivalent to create an outbound message carrying **reserve** nanotons to oneself, so that the subsequent output actions would not be able to spend more money than the remainder. It's a wrapper for opcodes "RAWRESERVE" and "RAWRESERVEX". See [TVM](https://broxus.gitbook.io/threaded-virtual-machine/).

Let's denote:

* `original_balance` is balance of the contract before the computing phase that is equal to balance of the contract before the transaction minus storage fee. Note: `original_balance` doesn't include `msg.value` and `original_balance` is not equal to `address(this).balance`.
* `remaining_balance` is contract's current remaining balance at the action phase after some handled actions and before handing the "rawReserve" action.

Let's consider how much nanotons (**reserve**) are reserved in all cases of **flag**:

* 0 -> `reserve = value` nanotons.
* 1 -> `reserve = remaining_balance - value` nanotons.
* 2 -> `reserve = min(value, remaining_balance)` nanotons.
* 3 = 2 + 1 -> `reserve = remaining_balance - min(value, remaining_balance)` nanotons.
* 4 -> `reserve = original_balance + value` nanotons.
* 5 = 4 + 1 -> `reserve = remaining_balance - (original_balance + value)` nanotons.
* 6 = 4 + 2 -> `reserve = min(original_balance + value, remaining_balance) = remaining_balance` nanotons.
* 7 = 4 + 2 + 1 -> `reserve = remaining_balance - min(original_balance + value, remaining_balance)` nanotons.
* 12 = 8 + 4 -> `reserve = original_balance - value` nanotons.
* 13 = 8 + 4 + 1 -> `reserve = remaining_balance - (original_balance - value)` nanotons.
* 14 = 8 + 4 + 2 -> `reserve = min(original_balance - value, remaining_balance)` nanotons.
* 15 = 8 + 4 + 2 + 1 -> `reserve = remaining_balance - min(original_balance - value, remaining_balance)` nanotons.

All other values of `flag` are invalid.

To make it clear, let's consider the order of `reserve` calculation:

1. if `flag` has bit `+8` then `value = -value`.
2. if `flag` has bit `+4` then `value += original_balance`.
3. Check `value >= 0`.
4. if `flag` has bit `+2` then `value = min(value, remaining_balance)`.
5. if `flag` has bit `+1` then `value = remaining_balance - value`.
6. `reserve = value`.
7. Check `0 <= reserve <= remaining_balance`.

Example:

```solidity
tvm.rawReserve(1 ton, 4 + 8);
```

See also: [23\_rawReserve.sol](https://github.com/tonlabs/samples/blob/master/solidity/23\_rawReserve.sol)

### tvm.initCodeHash()

```solidity
tvm.initCodeHash() returns (uint256 hash)
```

Returns the initial code hash that contract had when it was deployed.

### Hashing and cryptography

### tvm.hash()

```solidity
tvm.hash(TvmCell cellTree) returns (uint256);
tvm.hash(string data) returns (uint256);
tvm.hash(bytes data) returns (uint256);
tvm.hash(TvmSlice data) returns (uint256);
```

Executes TVM instruction "HASHCU" or "HASHSU" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.6. - F900). It computes the representation hash of a given argument and returns it as a 256-bit unsigned integer. For `string` and `bytes` it computes hash of the tree of cells that contains data but not data itself. See [sha256](api-functions-and-members.md#sha256) to count hash of data.

Example:

```solidity
uint256 hash = tvm.hash(TvmCell cellTree);
uint256 hash = tvm.hash(string);
uint256 hash = tvm.hash(bytes);
```

### tvm.checkSign()

```solidity
tvm.checkSign(uint256 hash, uint256 SignHighPart, uint256 SignLowPart, uint256 pubkey) returns (bool);
tvm.checkSign(uint256 hash, TvmSlice signature, uint256 pubkey) returns (bool);
tvm.checkSign(TvmSlice data, TvmSlice signature, uint256 pubkey) returns (bool);
```

Executes TVM instruction "CHKSIGNU" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.6. - F910) for variants 1 and 2. This command checks the Ed25519-signature of the **hash** using public key **pubkey**. Signature is represented by two uint256 **SignHighPart** and **SignLowPart** in the first variant and by the slice **signature** in the second variant. In the third variant executes TVM instruction "CHKSIGNS" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.6. - F911). This command checks Ed25519-signature of the **data** using public key **pubkey**. Signature is represented by the slice **signature**.

Example:

```solidity
uint256 hash;
uint256 SignHighPart;
uint256 SignLowPart;
uint256 pubkey;
bool signatureIsValid = tvm.checkSign(hash, SignHighPart, SignLowPart, pubkey);  // 1 variant

uint256 hash;
TvmSlice signature;
uint256 pubkey;
bool signatureIsValid = tvm.checkSign(hash, signature, pubkey);  // 2 variant

TvmSlice data;
TvmSlice signature;
uint256 pubkey;
bool signatureIsValid = tvm.checkSign(hash, signature, pubkey);  // 3 variant
```

### Deploy contract from contract

### tvm.insertPubkey()

```solidity
tvm.insertPubkey(TvmCell stateInit, uint256 pubkey) returns (TvmCell);
```

Inserts a public key into the `stateInit` data field. If the `stateInit` has wrong format then throws an exception.

### tvm.buildStateInit()

```solidity
// 1)
tvm.buildStateInit(TvmCell code, TvmCell data) returns (TvmCell stateInit);
// 2)
tvm.buildStateInit(TvmCell code, TvmCell data, uint8 splitDepth) returns (TvmCell stateInit);
// 3)
tvm.buildStateInit({code: TvmCell code, data: TvmCell data, splitDepth: uint8 splitDepth,
    pubkey: uint256 pubkey, contr: contract Contract, varInit: {VarName0: varValue0, ...}});
```

Generates a `StateInit` ([Everscale documentation](https://docs.everscale.network/arch/accounts#smart-contract-storage-stateinit)) from `code` and `data` `TvmCell`s. Member `splitDepth` of the tree of cell `StateInit`:

1. is not set. Has no value.
2. is set. `0 <= splitDepth <= 31`
3. Arguments can also be set with names. List of possible names:

* `code` (`TvmCell`) - defines the code field of the `StateInit`. Must be specified.
* `data` (`TvmCell`) - defines the data field of the `StateInit`. Conflicts with `pubkey` and `varInit`. Can be omitted, in this case data field would be built from `pubkey` and `varInit`.
* `splitDepth` (`uint8`) - splitting depth. `0 <= splitDepth <= 31`. Can be omitted. By default, it has no value.
* `pubkey` (`uint256`) - defines the public key of the new contract. Conflicts with `data`. Can be omitted, default value is 0.
* `varInit` (`initializer list`) - used to set [static](api-functions-and-members.md#keyword-static) variables of the contract. Conflicts with `data` and requires `contr` to be set. Can be omitted.
* `contr` (`contract`) - defines the contract whose `StateInit` is being built. Mandatory to be set if the option `varInit` is specified.

Examples of this function usage:

```solidity
contract A {
    uint static var0;
    address static var1;
}

contract C {

    function f() public pure {
        TvmCell code;
        TvmCell data;
        uint8 depth;
        TvmCell stateInit = tvm.buildStateInit(code, data);
        stateInit = tvm.buildStateInit(code, data, depth);
    }

    function f1() public pure {
        TvmCell code;
        TvmCell data;
        uint8 depth;
        uint pubkey;
        uint var0;
        address var1;

        TvmCell stateInit1 = tvm.buildStateInit({code: code, data: data, splitDepth: depth});
        stateInit1 = tvm.buildStateInit({code: code, splitDepth: depth, varInit: {var0: var0, var1: var1}, pubkey: pubkey, contr: A});
        stateInit1 = tvm.buildStateInit({varInit: {var0: var0, var1: var1}, pubkey: pubkey, contr: A, code: code, splitDepth: depth});
        stateInit1 = tvm.buildStateInit({contr: A, varInit: {var0: var0, var1: var1}, pubkey: pubkey, code: code, splitDepth: depth});
        stateInit1 = tvm.buildStateInit({contr: A, varInit: {var0: var0, var1: var1}, pubkey: pubkey, code: code});
        stateInit1 = tvm.buildStateInit({contr: A, varInit: {var0: var0, var1: var1}, code: code, splitDepth: depth});
        stateInit1 = tvm.buildStateInit({pubkey: pubkey, code: code, splitDepth: depth});
        stateInit1 = tvm.buildStateInit({code: code, splitDepth: depth});
        stateInit1 = tvm.buildStateInit({code: code});
    }
}
```

### tvm.buildDataInit()

```solidity
tvm.buildDataInit({pubkey: uint256 pubkey, contr: contract Contract, varInit: {VarName0: varValue0, ...}});
```

Generates `data` field of the `StateInit` ([Everscale documentation](https://docs.everscale.network/arch/accounts#smart-contract-storage-stateinit)). Parameters are the same as in [tvm.buildStateInit()](api-functions-and-members.md#tvmbuildstateinit).

```solidity
// SimpleWallet.sol
contract SimpleWallet {
    uint static value;
    // ...
}

// Usage
TvmCell data = tvm.buildDataInit({
    contr: SimpleWallet,
    varInit: {mulRes: 12345},
    pubkey: 0x3f82435f2bd40915c28f56d3c2f07af4108931ae8bf1ca9403dcf77d96250827
});
TvmCell code = ...;
TvmCell stateInit = tvm.buildStateInit({code: code, data: data});

// Note, the code above can be simplified to:
TvmCell stateInit = tvm.buildStateInit({
    code: code,
    contr: SimpleWallet,
    varInit: {mulRes: 12345},
    pubkey: 0x3f82435f2bd40915c28f56d3c2f07af4108931ae8bf1ca9403dcf77d96250827
});
```

### tvm.stateInitHash()

```solidity
tvm.stateInitHash(uint256 codeHash, uint256 dataHash, uint16 codeDepth, uint16 dataDepth) returns (uint256);
```

Calculates hash of the stateInit for given code and data specifications.

Example:

```solidity
TvmCell code = ...;
TvmCell data = ...;
uint codeHash = tvm.hash(code);
uint dataHash = tvm.hash(data);
uint16 codeDepth = code.depth();
uint16 dataDepth = data.depth();
uint256 hash = tvm.stateInitHash(codeHash, dataHash, codeDepth, dataDepth);
```

See also [internal doc](https://github.com/tonlabs/TON-Solidity-Compiler/blob/master/docs/internal/stateInit\_hash.md) to read more about this function mechanics.

### Deploy via new

Either `code` or `stateInit` option must be set when you deploy a contract from contract via keyword `new`. `stateInit` is a tree of cells that contains original state of the contract. `stateInit` contains `data`, `code` and another members.

See also [Everscale documentation](https://docs.everscale.network/arch/accounts#smart-contract-storage-stateinit) to read about `stateInit`.

Use `stateInit` option if you have the created account state (maybe offchain or onchain) and use `code` if you want to create account state in the `new` expression.

**Note**: Address of the new account is calculated as a hash of the `stateInit`. Constructor function parameters don't influence the address. See [New contract address problem](api-functions-and-members.md#new-contract-address-problem).

[Step-by-step description how to deploy contracts from the contract here](https://github.com/tonlabs/samples/blob/master/solidity/17\_ContractProducer.md).

Examples:

* [WalletProducer](https://github.com/tonlabs/samples/blob/master/solidity/17\_ContractProducer.sol).
* [SelfDeployer](https://github.com/tonlabs/samples/blob/master/solidity/21\_self\_deploy.sol).

### `stateInit` option usage

`stateInit` defines the origin state of the new account.

```solidity
TvmCell stateInit = ...;
address newWallet = new SimpleWallet{value: 1 ton, stateInit: stateInit}(arg0, arg1, ...);
```

### `code` option usage

`code` option defines the code of the new contract.

```solidity
TvmCell code = ...;
address newWallet = new SimpleWallet{value: 1 ton, code: code}(arg0, arg1, ...);
```

The following options can only be used with the `code` option:

* `pubkey` (`uint256`) - defines the public key of the new contract.
* `varInit` (`initializer list`) - used to set [static](api-functions-and-members.md#keyword-static) variables of the new contract.
* `splitDepth` (`uint8`) - splitting depth. `0 <= splitDepth <= 31`. By default, it has no value.

Example of these options usage:

```solidity
// file SimpleWallet.sol
...
contract SimpleWallet {
    address static m_owner;
    uint static m_value;
    ...
}

// file containing a contract that deploys a SimpleWallet
TvmCell code = ...;
address newWallet = new SimpleWallet{
    value: 1 ton,
    code: code,
    pubkey: 0xe8b1d839abe27b2abb9d4a2943a9143a9c7e2ae06799bd24dec1d7a8891ae5dd,
    splitDepth: 15,
    varInit: {m_owner: address(this), m_value: 15}
}(arg0, arg1, ...);
```

### Other deploy options

The following options can be used with both `stateInit` and `code`:

* `value` (`uint128`) - funds attached to the outbound internal message, that creates new account. This value must be set.
* `currencies` (`ExtraCurrencyCollection`) - currencies attached to the outbound internal message. Defaults to an empty set.
* `bounce` (`bool`) - if it's set and deploy falls (only at the computing phase, not at the action phase!) then funds will be returned. Otherwise, (flag isn't set or deploying terminated successfully) the address accepts the funds. Defaults to `true`.
* `wid` (`uint8`) - workchain id of the new account address. Defaults to `0`.
* `flag` (`uint16`) - flag used to send the outbound internal message. Defaults to `0`. Possible values of the `flag` are described here: [\<address>.transfer()](api-functions-and-members.md#addresstransfer).

```solidity
TvmCell stateInit = ...;
address newWallet = new SimpleWallet{
    stateInit: stateInit,
    value: 1 ton,
    wid: -1,
    flag: 0
}(arg0, arg1, ...);
```

### Deploy via \<address>.transfer()

You can also deploy the contract via [\<address>.transfer()](api-functions-and-members.md#addresstransfer). Just set the option `stateInit`.

* [Example of usage](https://github.com/tonlabs/samples/blob/master/solidity/11\_ContractDeployer.sol)
* [Step-by-step description how to deploy contracts from the contract here](https://github.com/tonlabs/samples/blob/master/solidity/17\_ContractProducer.md).

### New contract address problem

Address of the new account is calculated as a hash of the `stateInit`. Parameters of the constructor don't influence the address. The problem is that hacker can deploy the contract with your `stateInit` before you with malicious constructor parameters.

Let's consider how to protect against this problem:

1. Constructor is called by external message. We must Check if we didn't forget to set the public key in the contract and the inbound message is signed by that key. If hacker doesn't have your private key then he can't sign message to call the constructor. See [constructor of WalletProducer](https://github.com/tonlabs/samples/blob/master/solidity/17\_ContractProducer.sol).
2. Constructor is called by internal message. We should define static variable in the new contract that will contain address of the creator. Address of the creator will be a part of the `stateInit`. And in the constructor we must check address of the message sender. See [function `deployWallet` how to deploy contract](https://github.com/tonlabs/samples/blob/master/solidity/17\_ContractProducer.sol).\
   See [constructor of SimpleWallet](https://github.com/tonlabs/samples/blob/master/solidity/17\_SimpleWallet.sol).\
   If some contract should deploy plenty of contracts (with some contract's public key) then it's a good idea to declare static variable in the deployed contract. This variable can contain some sequence number. It will allow each new contact to have unique `stateInit`. See [SimpleWallet](https://github.com/tonlabs/samples/blob/master/solidity/17\_SimpleWallet.sol).\
   **Note**: contract's public key (`tvm.pubkey()`) is a part of `stateInit`.

### Misc functions from `tvm`

### tvm.code()

```solidity
tvm.code() returns (TvmCell);
```

Returns contract's code.

See [SelfDeployer](https://github.com/tonlabs/samples/blob/master/solidity/21\_self\_deploy.sol).

### tvm.codeSalt()

```solidity
tvm.codeSalt(TvmCell code) returns (optional(TvmCell) optSalt);
```

If **code** contains salt then **optSalt** contains one. Otherwise, **optSalt** doesn't contain any value.

### tvm.setCodeSalt()

```solidity
tvm.setCodeSalt(TvmCell code, TvmCell salt) returns (TvmCell newCode);
```

Inserts **salt** into **code** and returns new code **newCode**.

### tvm.pubkey()

```solidity
tvm.pubkey() returns (uint256);
```

Returns contract's public key, stored in contract data. If key is not set, function returns 0.

### tvm.setPubkey()

```solidity
tvm.setPubkey(uint256 newPubkey);
```

Set new contract's public key. Contract's public key can be obtained from `tvm.pubkey`.

### tvm.setCurrentCode()

```solidity
tvm.setCurrentCode(TvmCell newCode);
```

Changes this smart contract current code to that given by Cell **newCode**. Unlike [tvm.setcode()](api-functions-and-members.md#tvmsetcode) this function changes code of the smart contract only for current TVM execution, but has no effect after termination of the current run of the smart contract.

See example of how to use this function:

* [old contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_BadContract.sol)
* [new contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_NewVersion.sol)

### tvm.resetStorage()

```solidity
tvm.resetStorage();
```

Resets all state variables to their default values.

### tvm.functionId()

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

### tvm.encodeBody()

```solidity
tvm.encodeBody(function, arg0, arg1, arg2, ...) returns (TvmCell);
tvm.encodeBody(function, callbackFunction, arg0, arg1, arg2, ...) returns (TvmCell);
tvm.encodeBody(contract, arg0, arg1, arg2, ...) returns (TvmCell);
```

Constructs a message body for the function call. Body can be used as a payload for [\<address>.transfer()](api-functions-and-members.md#addresstransfer). If the **function** is `responsible`, **callbackFunction** must be set.

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

* [External function calls](api-functions-and-members.md#external-function-calls)
* [\<TvmSlice>.loadFunctionParams()](api-functions-and-members.md#tvmsliceloadfunctionparams)
* [tvm.buildIntMsg()](api-functions-and-members.md#tvmbuildintmsg)

### tvm.exit() and tvm.exit1()

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

### tvm.buildExtMsg()

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

### tvm.buildIntMsg()

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

Generates an internal outbound message that contains a function call. The result `TvmCell` can be used to send a message using [tvm.sendrawmsg()](api-functions-and-members.md#tvmsendrawmsg). If the `function` is `responsible` then `callbackFunction` parameter must be set.

`dest`, `value` and `call` parameters are mandatory. Another parameters can be omitted. See [\<address>.transfer()](api-functions-and-members.md#addresstransfer) where these options and their default values are described.

See also:

* sample [22\_sender.sol](https://github.com/tonlabs/samples/blob/master/solidity/22\_sender.sol)
* [tvm.encodeBody()](api-functions-and-members.md#tvmencodebody)

## tvm.sendrawmsg()

```solidity
tvm.sendrawmsg(TvmCell msg, uint8 flag);
```

Send the internal/external message `msg` with `flag`. It's a wrapper for opcode `SENDRAWMSG` ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.10). Internal message `msg` can be generated by [tvm.buildIntMsg()](api-functions-and-members.md#tvmbuildintmsg). Possible values of `flag` are described here: [\<address>.transfer()](api-functions-and-members.md#addresstransfer).

**Note:** make sure that `msg` has a correct format and follows the \[TL-B scheme]\[3] of `Message X`. For example:

```solidity
TvmCell msg = ...
tvm.sendrawmsg(msg, 2);
```

If the function is called by external message and `msg` has a wrong format (for example, the field `init` of `Message X` is not valid) then the transaction will be replayed despite the usage of flag 2. It will happen because the transaction will fail at the action phase.

## **math** namespace

`T` is an integer, [variable integer](api-functions-and-members.md#varint-and-varuint) or fixed point type in the `math.*` functions where applicable. Fixed point type is not applicable for `math.modpow2()`, `math.muldiv[r|c]()`, `math.muldivmod()` and `math.divmod()`.

### math.min() math.max()

```solidity
math.min(T a, T b, ...) returns (T);
math.max(T a, T b, ...) returns (T);
```

Returns the minimal (maximal) value of the passed arguments.

### math.minmax()

```solidity
math.minmax(T a, T b) returns (T /*min*/, T /*max*/);
```

Returns minimal and maximal values of the passed arguments.

Example:

```solidity
(uint a, uint b) = math.minmax(20, 10); // (a, b) == (10, 20)
```

### math.abs()

```solidity
math.abs(T val) returns (T);
```

Computes the absolute value of the given integer.

Example:

```solidity
int a = math.abs(-100); // a == 100
int b = -100;
int c = math.abs(b); // c == 100
```

### math.modpow2()

```solidity
math.modpow2(T value, uint power) returns (T);
```

Computes the `value mod 2^power`. Note: `power` should be a constant integer.

Example:

```solidity
uint constant pow = 10;
uint val = 1026;
uint a = math.modpow2(21, 4); // a == 5
uint b = math.modpow2(val, pow); // b == 2
```

### math.divr() math.divc()

```solidity
math.divc(T a, T b) returns (T);
math.divr(T a, T b) returns (T);
```

Returns result of the division of two integers. The return value is rounded. **ceiling** and **nearest** modes are used for `divc` and `divr` respectively. See also: [Division and rounding](api-functions-and-members.md#division-and-rounding).

Example:

```solidity
int c = math.divc(10, 3); // c == 4
int c = math.divr(10, 3); // c == 3

fixed32x2 a = 0.25;
fixed32x2 res = math.divc(a, 2); // res == 0.13
```

### math.muldiv() math.muldivr() math.muldivc()

```solidity
math.muldiv(T a, T b, T c) returns (T);
math.muldivr(T a, T b, T c) returns (T);
math.muldivc(T a, T b, T c) returns (T);
```

Multiplies two values and then divides the result by a third value. The return value is rounded. **floor**, **ceiling** and **nearest** modes are used for `muldiv`, `muldivc` and `muldivr` respectively. See also: [Division and rounding](api-functions-and-members.md#division-and-rounding).

Example:

```solidity
uint res = math.muldiv(3, 7, 2); // res == 10
uint res = math.muldivr(3, 7, 2); // res == 11
uint res = math.muldivc(3, 7, 2); // res == 11
```

### math.muldivmod()

```solidity
math.muldivmod(T a, T b, T c) returns (T /*result*/, T /*remainder*/);
```

This instruction multiplies first two arguments, divides the result by third argument and returns the result and the remainder. Intermediate result is stored in the 514 bit buffer, and the final result is rounded to the floor.

Example:

```solidity
uint a = 3;
uint b = 2;
uint c = 5;
(uint d, uint r) = math.muldivmod(a, b, c); // (d, r) == (1, 1)
int e = -1;
int f = 3;
int g = 2;
(int h, int p) = math.muldivmod(e, f, g); // (h, p) == (-2, 1)
```

### math.divmod()

```solidity
math.divmod(T a, T b) returns (T /*result*/, T /*remainder*/);
```

This function divides the first number by the second and returns the result and the remainder. Result is rounded to the floor.

Example:

```solidity
uint a = 11;
uint b = 3;
(uint d, uint r) = math.divmod(a, b); // (d, r) == (3, 2)

int e = -11;
int f = 3;
(int h, int p) = math.divmod(e, f); // (h, p) == (-3, 2)
```

### math.sign()

```solidity
math.sign(T val) returns (int2);
```

Returns:

* \-1 if `val` is negative;
* 0 if `val` is zero;
* 1 if `val` is positive.

Example:

```solidity
int8 sign = math.sign(-100); // sign == -1
int8 sign = math.sign(100); // sign == 1
int8 sign = math.sign(0); // sign == 0
```

### **tx** namespace

### tx.logicaltime

```solidity
tx.logicaltime returns (uint64);
```

Returns the logical time of the current transaction.

### tx.storageFee

It's an experimental feature and is available only in certain blockchain networks.

```solidity
tx.storageFee returns (uint120);
```

Returns the storage fee paid in the current transaction.

### **block** namespace

### block.timestamp

```solidity
block.timestamp returns (uint32);
```

Returns the current Unix time. Unix time is the same for the all transactions from one block.

{% hint style="warning" %}
The behavior of the method has been changed since **v. 0.67.0**: it has replaced the keyword `now ,`which is now considered deprecated, and now binds to opcode `NOW`, which returns a Unix timestamp.

If you use an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

### block.logicaltime

```solidity
block.logicaltime returns (uint64);
```

Returns the starting logical time of the current block.

### **rnd** namespace

The pseudorandom number generator uses the random seed. The initial value of the random seed before a smart contract execution in Everscale Blockchain is a hash of the smart contract address and the global block random seed. If there are several runs of the same smart contract inside a block, then all of these runs will have the same random seed. This can be fixed, for example, by running `rnd.shuffle()` (without parameters) each time before using the pseudorandom number generator.

### rnd.next

```solidity
rnd.next([Type limit]) returns (Type);
```

Generates a new pseudo-random number.

1. Returns `uint256` number.
2. If the first argument `limit > 0` then function returns the value in the range `0..limit-1`. Else if `limit < 0` then the returned value lies in range `limit..-1`. Else if `limit == 0` then it returns `0`.

Example:

```solidity
// 1)
uint256 r0 = rnd.next(); // 0..2^256-1
// 2)
uint8 r1 = rnd.next(100);  // 0..99
int8 r2 = rnd.next(int8(100));  // 0..99
int8 r3 = rnd.next(int8(-100)); // -100..-1
```

### rnd.getSeed

```solidity
rnd.getSeed() returns (uint256);
```

Returns the current random seed.

### rnd.setSeed

```solidity
rnd.setSeed(uint256 x);
```

Sets the random seed to `x`.

### rnd.shuffle

```solidity
(1)
rnd.shuffle(uint someNumber);
(2)
rnd.shuffle();
```

Randomizes the random seed. (1) Mixes the random seed and `someNumber`. The result is set as the random seed. (2) Mixes the random seed and the logical time of the current transaction. The result is set as the random seed.

Example:

```solidity
// (1)
uint256 someNumber = ...;
rnd.shuffle(someNumber);
// (2)
rnd.shuffle();
```

## abi namespace

### abi.encode(), abi.decode()

```solidity
(1)
abi.encode(TypeA a, TypeB b, ...) returns (TvmCell /*cell*/);
(2)
abi.decode(TvmCell cell, (TypeA, TypeB, ...)) returns (TypeA /*a*/, TypeB /*b*/, ...);
```

`abi.encode` creates `cell` from the values. `abi.decode` decodes the `cell` and returns the values. Note: all types must be set in `abi.decode`. Otherwise, `abi.decode` throws an exception.

Example:

```solidity
// pack the values to the cell
TvmCell cell = abi.encode(uint(1), uint(2), uint(3), uint(4));
// unpack the cell
(uint a, uint b, uint c, uint d) = abi.decode(cell, (uint, uint, uint, uint));
// a == 1
// b == 2
// c == 3
// d == 4
```

## **gosh** namespace

All `gosh.*` functions are experimental features and are available only in certain blockchain networks.

## gosh.diff and gosh.zipDiff

```solidity
(1)
gosh.diff(string oldText, string newText) returns (string patch)
(2)
gosh.zipDiff(bytes oldText, bytes newText) returns (bytes patch)
```

(1) Calculates [patch](https://en.wikipedia.org/wiki/Diff) between `oldText` and `newText`. Example: (2) It's the same as `gosh.diff` but it calculates `patch` between compressed strings.

```solidity
string oldText = ...;
string newText = ...;
string patch = gosh.diff(oldText, newText);
```

## gosh.applyPatch, gosh.applyPatchQ, gosh.applyZipPatch, gosh.applyZipPatchQ, gosh.applyZipBinPatch and gosh.applyZipBinPatchQ

```solidity
(1)
gosh.applyPatch(string oldText, string patch) returns (string newText)
gosh.applyPatchQ(string oldText, string patch) returns (optional(string) newText)
(2)
gosh.applyBinPatch(bytes oldText, bytes patch) returns (bytes newText)
gosh.applyBinPatchQ(bytes oldText, bytes patch) returns (optional(bytes) newText)
(3)
gosh.applyZipPatch(bytes oldText, bytes patch) returns (bytes newText)
gosh.applyZipPatchQ(bytes oldText, bytes patch) returns (optional(bytes) newText)
(4)
gosh.applyZipBinPatch(bytes oldText, bytes patch) returns (bytes newText)
gosh.applyZipBinPatchQ(bytes oldText, bytes patch) returns (optional(bytes) newText)
```

(1) Applies `patch` to the `oldText`. If it's impossible (bad patch), `gosh.applyPatch` throws an exception with type check error code (-8) but`gosh.applyPatchQ` returns `null`. Example: (2) These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to binary arrays. (3) These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to compressed strings. (4) These are the same as `gosh.applyPatch`/`gosh.applyPatchQ` but these functions are applied to compressed binary arrays.

```solidity
string oldText = ...;
string patch = ...;
string newText = gosh.applyPatch(oldText, patch);
```

## gosh.zip and gosh.unzip

```solidity
gosh.zip(string text) returns (bytes zip)
gosh.unzip(bytes zip) returns (optional(string) text)
```

`gosh.zip` converts the `text` to compressed `bytes`. `gosh.unzip` reverts such compression.

## Exponentiation

Exponentiation `**` is only available for unsigned types in the exponent. The resulting type of an exponentiation is always equal to the type of the base. Please take care that it is large enough to hold the result and prepare for potential assertion failures or wrapping behaviour.

Note that `0**0` throws an exception.

Example:

```solidity
uint b = 3;
uint32 p = 4;
uint res = b ** p; // res == 81
```

## selfdestruct

```solidity
selfdestruct(address dest_addr);
```

Creates and sends the message that carries all the remaining balance of the current smart contract and destroys the current account.

See example of how to use the `selfdestruct` function:

* [Kamikaze](https://github.com/tonlabs/samples/blob/master/solidity/8\_Kamikaze.sol)

## sha256

```solidity
// (1)
sha256(TvmSlice slice) returns (uint256)
// (2)
sha256(bytes b) returns (uint256)
// (3)
sha256(string str) returns (uint256)
```

1. Computes the SHA-256 hash. If the bit-length of `slice` is not divisible by eight, throws a cell underflow [exception](api-functions-and-members.md#tvm-exception-codes). References of `slice` are not used to compute the hash. Only data bits located in the root cell of `slice` are used.
2. Computes the SHA-256 hash only for the first 127 bytes. If `bytes.length > 127` then `b[128], b[129], b[130] ...` elements are ignored.
3. Same as for `bytes`: only the first 127 bytes are taken into account.

See also [tvm.hash()](api-functions-and-members.md#tvmhash) to compute representation hash of the whole tree of cells.

## gasToValue

```solidity
gasToValue(uint128 gas) returns (uint128 value)
gasToValue(uint128 gas, int8 wid) returns (uint128 value)
```

Returns worth of **gas** in workchain **wid**. Throws an exception if **wid** is not equal to `0` or `-1`. If `wid` is omitted than used the contract's `wid`.

## valueToGas

```solidity
valueToGas(uint128 value) returns (uint128 gas)
valueToGas(uint128 value, int8 wid) returns (uint128 gas)
```

Counts how much **gas** could be bought on **value** nanotons in workchain **wid**. Throws an exception if **wid** is not equal to `0` or `-1`. If `wid` is omitted than used the contract's `wid`.
