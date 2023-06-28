# External function calls

TON Solidity compiler allows specifying different parameters of the outbound internal message that
is sent via external function call. Note, all external function calls are asynchronous, so
callee function will be called after termination of the current transaction.
`value`, `currencies`, `bounce` or `flag` options can be set. See [\<address\>.transfer()](#addresstransfer)
where these options are described.
**Note:** if `value` isn't set, then the default value is equal to 0.01 ton, or 10^7 nanoton. It's equal
to 10_000 units of gas in workchain.
If the callee function returns some value and marked as `responsible` then `callback` option must be set.
This callback function will be called by another contract. Remote function will pass its return
values as function arguments for the callback function. That's why types of return values of the
callee function must be equal to function arguments of the callback function.
If the function marked as `responsible` then field `answerId` appears in the list of input parameters of the
function in `*abi.json` file. `answerId` is function id that will be called.

Example of the external call of the function that returns nothing:

```TVMSolidity
interface IContract {
    function f(uint a) external;
}

contract Caller {
    function callExt(address addr) public {
        IContract(addr).f(123); // attached default value: 0.01 ton
        IContract(addr).f{value: 10 ton}(123);
        IContract(addr).f{value: 10 ton, flag: 3}(123);
        IContract(addr).f{value: 10 ton, bounce: true}(123);
        IContract(addr).f{value: 1 micro, bounce: false, flag: 128}(123);
        ExtraCurrencyCollection cc;
        cc[12] = 1000;
        IContract(addr).f{value: 10 ton, currencies:cc}(123);
    }
}
```

Example of the external call of the function that returns some values:

```TVMSolidity
contract RemoteContract {
    // Note this function is marked as responsible to call callback function
    function getCost(uint x) public pure responsible returns (uint) {
        uint cost = x == 0 ? 111 : 222;
        // return cost and set option for outbound internal message.
        return{value: 0, bounce: false, flag: 64} cost;
    }
}

contract Caller {
    function test(address addr, uint x) public pure {
        // `getCost` returns result to `onGetCost`
        RemoteContract(addr).getCost{value: 1 ton, callback: Caller.onGetCost}(x);
    }

    function onGetCost(uint cost) public {
        // Check if msg.sender is expected address
        // we get cost value, we can handle this value
    }
}
```

`*.abi.json` for `responsible` function `getCost`:

```json
{
    "name": "getCost",
    "inputs": [
        {"name":"answerId","type":"uint32"},
        {"name":"x","type":"uint256"}
    ],
    "outputs": [
        {"name":"value0","type":"uint256"}
    ]
}
```

See also:

* Example of callback usage: [24_SquareProvider](https://github.com/tonlabs/samples/blob/master/solidity/24_SquareProvider.sol)
* Example of callback usage: [4.1_CentralBank](https://github.com/tonlabs/samples/blob/master/solidity/4.1_CentralBank.sol)
and [4.1_CurrencyExchange.sol](https://github.com/tonlabs/samples/blob/master/solidity/4.1_CurrencyExchange.sol)
* [return](#return)

## Synchronous calls

TON Solidity compiler allows user to perform synchronous calls. To do it user should call a remote contract
function with `.await` suffix. Example:

```TVMSolidity
interface IContract {
    function getNum(uint a) external responsible returns (uint) ;
}

contract Caller {
    function call(address addr) public pure {
        ...
        uint res = IContract(addr).getNum(123).await;
        require(res == 124, 101);
        ...
    }
}
```

When function `call` is called this code:

1) Executes code before the `.await` call;
2) Generates and sends an internal message to **IContract** contract with address **addr**;
3) Saves current state of the contract including continuation, that is currently executed;
4) Waits for the answer from the remote contract;
5) If an internal message is received from the required address, contract unpacks the state and continues function execution.

Such await calls can be used everywhere in the code (e.g. inside a cycle), but be aware that usage of such
construction means that your contract function could possibly execute in more than one transaction.
Should be mentioned, that execution after the await call will spend money, that were attached to the responce
message. It means, that for correct work the receiver contract should make an accept after the await call,
or be sure that remote contract attaches enough currency for further execution.

**Note**: This feature was designed to be used in debots, in usual contracts use it at your own risk.