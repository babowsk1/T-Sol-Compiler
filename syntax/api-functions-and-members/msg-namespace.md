# Msg namespace

## sender

```solidity
msg.sender (address)
```

Returns:

* sender of the message for internal message.
* address(0) for external message.
* address(0) for tick/tock transaction.

## value

```solidity
msg.value (uint128)
```

Returns:

* Balance of the inbound message in nanotons for internal message.
* 0 for external message.
* Undefined value for tick/tock transaction.

## currencies

```solidity
msg.currencies (ExtraCurrencyCollection)
```

Collections of arbitrary currencies contained in the balance of the inbound message.

## pubkey()

```solidity
msg.pubkey() returns (uint256);
```

Returns public key that is used to check the message signature. If the message isn't signed then it's equal to `0`. See also: [Contract execution](../../api-functions-and-members/api-functions-and-members.md#contract-execution), [pragma AbiHeader](../../api-functions-and-members/api-functions-and-members.md#pragma-abiheader).

## isInternal, isExternal and isTickTock

Returns flag whether the contract is called by internal message, external message or by tick/tock transactions.

## createdAt

```solidity
msg.createdAt (uint32)
```

Returns the field **created\_at** of the external inbound message.

## data

```solidity
msg.data (TvmSlice)
```

Returns the payload of an inbound message.

## hasStateInit

```solidity
msg.hasStateInit (bool)
```

Whether the internal/external inbound message contains field `stateInit`. Returns undefined value for tick/tock transaction. See \[TL-B scheme]\[3] of `Message X`.
