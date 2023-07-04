# Fallback

`fallback` function is called on receiving an inbound internal/external message in such cases:

1. The message contains a function id that the contract doesn't contain.
2. Bit-length of the message is from 1 to 31 (including).
3. Bit-length of the message is equal to zero, but the message contains reference(s).

**Note**: if the message has correct function id but invalid encoded function parameters then
the transaction fail with an exception (e.g. [cell underflow exception](#tvm-exception-codes)).

If in the contract there is no fallback function then contract has implicit fallback function
that throws [exception](#solidity-runtime-errors).

Example:

```solidity
// file ContractA.sol
contract ContractA {
    uint public counter = 0;

    function f(uint a, uint b) public pure { /*...*/ }

    fallback() external {
        ++counter;
    }

}

// file ContractB.sol
import "ContractA.sol";
import "ContractAnother.sol";

contract ContractB {
    // Let's condider that `addr` is ContractA's address. 4 messages are send. ContractA's fallback
    // function will handle that messages except the last one.
    function g(address addr) public pure {
        tvm.accept();
        // The message contains a function id that the contract doesn't contain.
        // There is wrong casting to ContractAnother. `addr` is ContractA's address.
        ContractAnother(addr).sum{value: 1 ton}(2, 2);

        {
            TvmBuilder b;
            b.storeUnsigned(1, 1);
            // Bit-length of the message is equal to 20 bits.
            addr.transfer({value: 2 ton, body: b.toCell()});
        }

        {
            TvmBuilder b;
            b.storeRef(b);
            // Bit-length of the message is equal to zero but the message contains one reference.
            addr.transfer({value: 1 ton, body: b.toCell()});
        }

        TvmBuilder b;
        uint32 id = tvm.functionId(ContractA.f);
        b.store(id, uint(2));
        // ContractA's fallback function won't be called because the message body doesn't contain
        // the second ContractA.f's parameter. It will cause cell underflow exception in ContractA.
        addr.transfer({value: 1 ton, body: b.toCell()});
    }
}
```