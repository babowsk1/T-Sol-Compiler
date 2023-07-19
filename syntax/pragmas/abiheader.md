# AbiHeader

```solidity
pragma AbiHeader time;
pragma AbiHeader pubkey;
pragma AbiHeader expire;
```

Defines headers that are used in external messages:

* `pubkey` (`uint256`) - optional public key that the message can be signed with.
* `time` (`uint64`)  - local time when message was created. Used for replay protection.
* `expire` (`uint32`) - time when the message should be meant as expired.

### **Note:**

- `time` presents in external messages if `pragma AbiHeader time` is used OR there is no
`afterSignatureCheck` function defined in the contract.
- `time` doesn't present in external messages if `pragma AbiHeader time` isn't used AND there
is `afterSignatureCheck` function defined in the contract.

Defined headers are listed in `*.abi.json` file in `header` section.

See also: [Contract execution](../../other/contract-execution.md), [afterSignatureCheck](../special-contract-functions/aftersignaturecheck.md), [msg.pubkey()](../api-functions-and-members/msg-namespace.md#pubkey) and [tvm.pubkey()](../api-functions-and-members/misc-functions-from-tvm.md#pubkey). To read more about these fields and ABI follow this [link](https://docs.ton.dev/86757ecb2/p/40ba94-abi-specification-v2). Here is example of [message expiration time](https://docs.ton.dev/86757ecb2/p/88321a-message-expiration-time) usage.
