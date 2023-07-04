# Pragmas

`pragma` keyword is used to enable certain compiler features or checks.
A pragma directive is always local to a source file, so you have to add
the pragma to all your files if you want to enable it in your whole project.
If you import another file, the pragma from that file is not
automatically applied to the importing file.

## pragma ton-solidity

```solidity
pragma ton-solidity >= 0.35.5;      // Check if the compiler version is greater or equal than 0.35.5
pragma ton-solidity ^ 0.35.5;       // Check if the compiler version is greater or equal than 0.35.5 and less than 0.36.0
pragma ton-solidity < 0.35.5;       // Check if the compiler version is less than 0.35.5
pragma ton-solidity >= 0.35.5 < 0.35.7; // Check if the compiler version is equal to either 0.35.5 or 0.35.6
```

Used to restrict source file compilation to the particular compiler versions.

## pragma-copyleft

```solidity
pragma copyleft <type>, <wallet_address>; 
```

It is an experimental feature available only in certain blockchain deployments.

Parameters: 
 * `<type>` (`uint8`) - copyleft type. 
 * `<wallet_address>` (`uint256`) - author's wallet address in masterchain.

If contract has the `copyleft` pragma, it means that after each transaction some part of validator's fee
is transferred to `<wallet_address>` according to the `<type>` rule.

For example:

```solidity
pragma copyleft 0, 0x2cfbdc31c9c4478b61472c72615182e9567595b857b1bba9e0c31cd9942f6ca41;
```

## pragma ignoreIntOverflow

```solidity
pragma ignoreIntOverflow;
```

Turns off binary operation result overflow check.

## pragma AbiHeader

```solidity
pragma AbiHeader notime;
pragma AbiHeader pubkey;
pragma AbiHeader expire;
```

Defines headers that are used in external messages:

* `notime` - disables `time` abi header, which is enabled by default. Abi header `time` – `uint64` local time when message was created, used for replay protection
* `pubkey` (`uint256`) - optional public key that the message can be signed with.
* `expire` (`uint32`)  - time when the message should be meant as expired.

**Note:**

Defined headers are listed in `*.abi.json` file in `header` section.

See also: [Contract execution](#contract-execution), [afterSignatureCheck](#aftersignaturecheck),
[msg.pubkey()](#msgpubkey) and [tvm.pubkey()](#tvmpubkey).
To read more about these fields and ABI follow this [link](https://docs.ton.dev/86757ecb2/p/40ba94-abi-specification-v2).
Here is example of [message expiration time](https://docs.ton.dev/86757ecb2/p/88321a-message-expiration-time) usage.

## pragma msgValue

```solidity
pragma msgValue <value>;
```

Allows specifying default value in nanotons attached to the
internal outbound messages used to call another contract. If it's not
specified, this value is set to 10 000 000 nanotons.

Example:

```solidity
pragma msgValue 123456789;
pragma msgValue 1e8;
pragma msgValue 10 ton;
pragma msgValue 10_000_000_123;
```

## pragma upgrade func/oldsol

```solidity
pragma upgrade func;
pragma upgrade oldsol;
```

Defines that code is compiled with special selector that is needed to upgrade func/solidity contracts.