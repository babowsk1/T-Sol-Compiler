# Compiler version

T-Sol Compiler add its current version to the generated code. This version can be obtained:

1) using [tvm_linker](https://github.com/tonlabs/TVM-linker#2-decoding-of-boc-messages-prepared-externally) from a `*.tvc` file:

    ```bash
    tvm_linker decode --tvm <tvc-file>
    ```

2) using [tonos-cli](https://github.com/tonlabs/tonos-cli#48-decode-commands) from a `*.boc` file, a `*.tvc` file, or a network account:

```bash
tonos-cli decode tvc [--tvc] [--boc] <input>
```