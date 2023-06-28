# State variables

## Decoding state variables

You can decode state variables using tonos-cli. See `tonos-cli decode account --help`.

See also: [\<TvmSlice\>.loadStateVars()](#tvmsliceloadstatevars).

## Keyword `constant`

For `constant` variables, the value has to be a compile time constant and this value is
substituted where the variable is used. The value has to be assigned where the variable is declared.
Example:

```TVMSolidity
contract MyContract {
    uint constant cost = 100;
    uint constant cost2 = cost + 200;
    address constant dest = address.makeAddrStd(-1, 0x89abcde);
}
```

## Keyword `static`

Static state variables are used in the contract initial state generation.
Such variables can be set while deploying contract from contract
(onchain) or by tvm-linker (offchain). Example:

```TVMSolidity
contract C {
    uint static a; // ok
    uint static b = 123; // error
}
```

See also:

* [`code` option usage](#code-option-usage)
* [New contract address problem](#new-contract-address-problem)

## Keyword `public`

For each public state variable, a getter function is generated. Generated
function has the same name and return type as the public variable. This
function can be called only locally. Public state variables are useful,
because you don't need to write functions that return a particular state variable manually.

Example:

```TVMSolidity
contract C {
    uint public a;
    uint public static b; // it's ok. Variable is both public and static.
}
```
