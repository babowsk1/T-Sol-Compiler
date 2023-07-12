# Solidity runtime errors

Smart-contract written on solidity can throw runtime errors while execution.

Solidity runtime error codes:

* **40** - External inbound message has an invalid signature. See [tvm.pubkey()](../syntax/api-functions-and-members/misc-functions-from-tvm.md#pubkey) and [msg.pubkey()](../syntax/api-functions-and-members/msg-namespace.md#pubkey).
* **50** - Array index or index of [\<mapping>.at()](../syntax/changes-and-extensions-in-solidity-types/mapping.md#at) is out of range.
* **51** - Contract's constructor has already been called.
* **52** - Replay protection exception. See `timestamp` in [pragma AbiHeader](../syntax/pragmas/).
* **53** - See [\<address>.unpack()](../syntax/changes-and-extensions-in-solidity-types/functions.md#unpack).
* **54** - `<array>.pop` call for an empty array.
* **55** - See [tvm.insertPubkey()](../syntax/api-functions-and-members/misc-functions-from-tvm.md#pubkey).
* **57** - External inbound message is expired. See `expire` in [pragma AbiHeader](../syntax/pragmas/abiheader.md).
* **58** - External inbound message has no signature but has public key. See `pubkey` in [pragma AbiHeader](../syntax/pragmas/abiheader.md).
* **60** - Inbound message has wrong function id. In the contract there are no functions with such function id and there is no fallback function that could handle the message. See [fallback](../syntax/special-contract-functions/fallback.md).
* **61** - Deploying `StateInit` has no public key in `data` field.
* **62** - Reserved for internal usage.
* **63** - See [\<optional(Type)>.get()](../syntax/tvm-specific-types/optional-type.md#get).
* **64** - `tvm.buildExtMSg()` call with wrong parameters. See [tvm.buildExtMsg()](../syntax/api-functions-and-members/misc-functions-from-tvm.md#buildextmsg).
* **66** - Convert an integer to a string with width less than number length. See [format()](solidity-runtime-errors.md#format).
* **67** - See [gasToValue](../syntax/api-functions-and-members/gastovalue.md) and [valueToGas](../syntax/api-functions-and-members/valuetogas.md).
* **68** - There is no config parameter 20 or 21.
* **69** - Zero to the power of zero calculation (`0**0` in solidity style or `0^0`).
* **70** - `string` method `substr` was called with substr longer than the whole string.
* **71** - Function marked by `externalMsg` was called by internal message.
* **72** - Function marked by `internalMsg` was called by external message.
* **73** - The value can't be converted to enum type.
* **74** - Await answer message has wrong source address.
* **75** - Await answer message has wrong function id.
* **76** - Public function was called before constructor.
* **77** - It's impossible to convert `variant` type to target type. See [variant.toUint()](../syntax/tvm-specific-types/variant.md#touint).
* **78** - There's no private function with the function id.
* **79** - You are deploying contract that uses [pragma upgrade func/oldsol](../syntax/pragmas/upgrade-func-oldsol.md). Use the contract only for updating another contracts.
