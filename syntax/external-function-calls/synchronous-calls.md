# Synchronous calls

T-Sol compiler allows user to perform synchronous calls. To do it user should call a remote contract
function with `.await` suffix. Example:

```solidity
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