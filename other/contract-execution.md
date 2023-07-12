# Contract execution

Before executing any contract function special code is executed. In `*.code` file there are two special
functions: `main_internal` and `main_external` that run on internal and external messages
respectively. These functions initialize some internal global variables and call contract
function of special function like `receive`, `fallback`, `onBounce`, `onTickTock`, etc.

Before calling contract's function `main_external` does:

1. Checks the message signature. Let's consider how the signature is checked:
   - If signature is present and `pubkey` header isn't defined then `tvm.pubkey()` is used
   for checking.
   - If signature isn't present and `pubkey` header isn't defined then signature isn't checked.
   - If signature is present, `pubkey` header is defined and `pubkey` isn't present in the
   message then `tvm.pubkey()` is used for checking.
   - If signature is present, `pubkey` header is defined and `pubkey` is present in the
   message then `msg.pubkey()` is used for checking.
   - If signature isn't present, `pubkey` header is defined and `pubkey` is present in the
   message then an [exception with code 58](troubleshooting/solidity-runtime-errors.md) is thrown.
2. Replay protection:
   - `time` is present and there is no `afterSignatureCheck` then the contract checks whether
   `oldTime` < `time` < `now` * 1000 + 30 minutes. If it's true then `oldTime` is updated by new `time`.
   Otherwise, an exception is thrown.
   - there is `afterSignatureCheck` (despite usage of `time`) then make your own replay protection.
3. Message expiration:
   - `expire` is present and there is no `afterSignatureCheck` then the contract checks whether
   `expire` > `now`.
   - there is `afterSignatureCheck` (despite usage of `expire`) then make your own check.

See also: [pragma AbiHeader](syntax/pragmas/abiheader.md), [afterSignatureCheck](syntax/special-contract-functions/aftersignaturecheck.md).