# Tx namespace

## **tx** namespace

### logicaltime

```solidity
tx.logicaltime returns (uint64);
```

Returns the logical time of the current transaction.

### storageFee

It's an experimental feature and is available only in certain blockchain networks.

```solidity
tx.storageFee returns (uint120);
```

Returns the storage fee paid in the current transaction.

{% hint style="warning" %}
&#x20;The return type of **`tx.storageFee`** has been changed to **`uint120`** (previously **`uint64`**).

The old property is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

## **block** namespace

#### block.timestamp

```solidity
block.timestamp returns (uint32);
```

Returns the current Unix time. Unix time is the same for the all transactions from one block.

{% hint style="warning" %}
The behavior of the method has been changed since **v. 0.67.0**: it has replaced the keyword `now ,`which is now considered deprecated, and now binds to opcode `NOW`, which returns a Unix timestamp.

If you use an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

#### block.logicaltime

```solidity
block.logicaltime returns (uint64);
```

Returns the starting logical time of the current block.

{% hint style="warning" %}
The property **`tx.timestamp`** has been renamed to **`tx.logicaltime`**. The old property is available and marked as deprecated. If you are using an earlier version of the compiler, please refer to the corresponding version of the documentation.
{% endhint %}

## **rnd** namespace

The pseudorandom number generator uses the random seed. The initial value of the random seed before a smart contract execution in Everscale Blockchain is a hash of the smart contract address and the global block random seed. If there are several runs of the same smart contract inside a block, then all of these runs will have the same random seed. This can be fixed, for example, by running `rnd.shuffle()` (without parameters) each time before using the pseudorandom number generator.

#### rnd.next

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

#### rnd.getSeed

```solidity
rnd.getSeed() returns (uint256);
```

Returns the current random seed.

#### rnd.setSeed

```solidity
rnd.setSeed(uint256 x);
```

Sets the random seed to `x`.

#### rnd.shuffle

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
