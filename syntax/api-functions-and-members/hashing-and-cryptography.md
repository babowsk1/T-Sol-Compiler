# Hashing and cryptography

## hash()

```solidity
tvm.hash(TvmCell cellTree) returns (uint256);
tvm.hash(string data) returns (uint256);
tvm.hash(bytes data) returns (uint256);
tvm.hash(TvmSlice data) returns (uint256);
```

Executes TVM instruction "HASHCU" or "HASHSU" ([TVM](https://broxus.gitbook.io/threaded-virtual-machine/) - A.11.6. - F900). It computes the representation hash of a given argument and returns it as a 256-bit unsigned integer. For `string` and `bytes` it computes hash of the tree of cells that contains data but not data itself. See [sha256](../../api-functions-and-members/api-functions-and-members.md#sha256) to count hash of data.

Example:

```solidity
uint256 hash = tvm.hash(TvmCell cellTree);
uint256 hash = tvm.hash(string);
uint256 hash = tvm.hash(bytes);
```

## checkSign()

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
