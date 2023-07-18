# Tvm namespace

## TVM instructions

### accept()

```solidity
tvm.accept();
```

Executes TVM instruction "ACCEPT" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.2). This instruction sets current gas limit to its maximal allowed value. This action is required to process external messages that bring no value.

See example of how to use this function:

* [accumulator](https://github.com/tonlabs/samples/blob/master/solidity/1\_Accumulator.sol)

### setGasLimit()

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

### buyGas()

```solidity
tvm.buyGas(uint value);
```

Computes the amount of gas that can be bought for `value` nanotons, and sets **gl**\
accordingly in the same way as [tvm.setGasLimit()](tvm-namespace.md#setgaslimit).

### commit()

```solidity
tvm.commit();
```

Creates a "check point" of the state variables (by copying them from c7 to c4) and register c5. If the contract throws an exception at the computing phase then the state variables and register c5 will roll back to the "check point", and the computing phase will be considered "successful". If contract doesn't throw an exception, it has no effect.

### rawCommit()

```solidity
tvm.rawCommit();
```

Same as [tvm.commit()](tvm-namespace.md#commit) but doesn't copy the state variables from c7 to c4. It's a wrapper for opcode `COMMIT`. See [TVM](https://broxus.gitbook.io/threaded-virtual-machine/).

**Note**: Don't use `tvm.rawCommit()` after `tvm.accept()` in processing external messages because you don't save from c7 to c4 the hidden state variable `timestamp` that is used for replay protection.

### getData()

```solidity
tvm.getData() returns (TvmCell);
```

**Note:** Function is experimental.

A dual of the `tvm.setData()`function. It returns value of the `c4` register. Obtaining a raw storage cell can be useful when upgrading a new version of the contract that introduces an altered data layout.

Manipulation with a raw storage cell requires understanding of the way the compiler stores the data. Refer to the description of `tvm.setData()` below to get more details.

**Note:** state variables and replay protection timestamp stored in the data cell have the same values that were before the transaction. See [tvm.commit()](tvm-namespace.md#commit) to learn about register `c4` update.

### setData()

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

### log()

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

### hexdump() and tvm.bindump()

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

### setcode()

```solidity
tvm.setcode(TvmCell newCode);
```

This command creates an output action that would change this smart contract code to that given by the `TvmCell` **newCode** (this change will take effect only after the successful termination of the current run of the smart contract).

See example of how to use this function:

* [old contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_BadContract.sol)
* [new contract](https://github.com/tonlabs/samples/blob/master/solidity/12\_NewVersion.sol)

### configParam()

```solidity
tvm.configParam(uint8 paramNumber) returns (TypeA a, TypeB b, ...);
```

Executes TVM instruction "CONFIGPARAM" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.4. - F832). This command returns the value of the global configuration parameter with integer index **paramNumber**. Argument should be an integer literal. Supported **paramNumbers**: 1, 15, 17, 34.

### rawConfigParam()

```solidity
tvm.rawConfigParam(uint8 paramNumber) returns (TvmCell cell, bool status);
```

Executes TVM instruction "CONFIGPARAM" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.4. - F832). Returns the value of the global configuration parameter with integer index **paramNumber** as a `TvmCell` and a boolean status.

{% hint style="warning" %}
The behavior of the **`tvm.rawConfigParam(...)`** method has been changed since **v0.68.0**: it now returns an optional `TvmCell` instead of a tuple **`(TvmCell, bool)`**. If you use an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

### rawReserve()

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

### initCodeHash()

```solidity
tvm.initCodeHash() returns (uint256 hash)
```

Returns the initial code hash that contract had when it was deployed.
