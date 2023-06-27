# Import

T-Sol Compiler allows user to import remote files using link starting with `http`.
If import file name starts with `http`, then compiler tries to download the file using this
link and saves it to the folder `.solc_imports`. If compiler fails to create this folder or
to download the file, then an error is emitted.

**Note**: to import file from GitHub, one should use link to the raw version of the file.

Example:

```TVMSolidity
pragma ton-solidity >= 0.35.0;
pragma AbiHeader expire;
pragma AbiHeader pubkey;

import "https://github.com/tonlabs/debots/raw/9c6ca72b648fa51962224ec0d7ce91df2a0068c1/Debot.sol";
import "https://github.com/tonlabs/debots/raw/9c6ca72b648fa51962224ec0d7ce91df2a0068c1/Terminal.sol";

contract HelloDebot is Debot {
  ...
}
```