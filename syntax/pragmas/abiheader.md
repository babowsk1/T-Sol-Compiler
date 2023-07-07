# AbiHeader

Defines headers that are used in external messages:

* `notime` - disables `time` abi header, which is enabled by default. Abi header `time` – `uint64` local time when message was created, used for replay protection
* `pubkey` (`uint256`) - optional public key that the message can be signed with.
* `expire` (`uint32`) - time when the message should be meant as expired.

```solidity
pragma AbiHeader notime;
pragma AbiHeader pubkey;
pragma AbiHeader expire;
```

### **Note**

Defined headers are listed in `*.abi.json` file in `header` section.

See also: [Contract execution](abiheader.md#contract-execution), [afterSignatureCheck](abiheader.md#aftersignaturecheck), [msg.pubkey()](abiheader.md#msgpubkey) and [tvm.pubkey()](abiheader.md#tvmpubkey). To read more about these fields and ABI follow this [link](https://docs.ton.dev/86757ecb2/p/40ba94-abi-specification-v2). Here is example of [message expiration time](https://docs.ton.dev/86757ecb2/p/88321a-message-expiration-time) usage.