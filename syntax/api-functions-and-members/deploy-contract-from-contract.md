# Deploy contract from contract

## insertPubkey()

```solidity
tvm.insertPubkey(TvmCell stateInit, uint256 pubkey) returns (TvmCell);
```

Inserts a public key into the `stateInit` data field. If the `stateInit` has wrong format then throws an exception.

## buildStateInit()

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
* `varInit` (`initializer list`) - used to set [static](/syntax/state-variables/keyword-static.md) variables of the contract. Conflicts with `data` and requires `contr` to be set. Can be omitted.
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

## buildDataInit()

```solidity
tvm.buildDataInit({pubkey: uint256 pubkey, contr: contract Contract, varInit: {VarName0: varValue0, ...}});
```

Generates `data` field of the `StateInit` ([Everscale documentation](https://docs.everscale.network/arch/accounts#smart-contract-storage-stateinit)). Parameters are the same as in [tvm.buildStateInit()](deploy-contract-from-contract.md/#builddatainit).

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

## stateInitHash()

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

**Note**: Address of the new account is calculated as a hash of the `stateInit`. Constructor function parameters don't influence the address. See [New contract address problem](deploy-contract-from-contract.md/#new-contract-address-problem).

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
* `varInit` (`initializer list`) - used to set [static](/syntax/state-variables/keyword-static.md) variables of the new contract.
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
* `flag` (`uint16`) - flag used to send the outbound internal message. Defaults to `0`. Possible values of the `flag` are described here: [\<address>.transfer()](../changes-and-extensions-in-solidity-types/functions.md#transfer).

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

You can also deploy the contract via [\<address>.transfer()](../changes-and-extensions-in-solidity-types/functions.md#transfer). Just set the option `stateInit`.

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
