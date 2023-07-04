# Copyleft

{% hint style="warning" %}
It is an experimental feature available only in certain blockchain deployments.
{% endhint %}

If a contract has the `copyleft` pragma, it means that after each transaction, some part of the validator's fee is transferred to `<wallet_address>` according to the `<type>` rule.

```solidity
pragma copyleft <type>, <wallet_address>; 
```

Parameters:

* `<type>` (`uint8`) - copyleft type.
* `<wallet_address>` (`uint256`) - author's wallet address in masterchain.

For example:

```solidity
pragma copyleft 0, 0:2cfbdc31c9c4478b61472c72615182e9567595b857b1bba9e0c31cd9942f6ca41;
```
