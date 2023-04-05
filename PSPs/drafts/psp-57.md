# Title

- **PSP Number: 57**
- **Authors:** Shunsuke Watanabe (Github: @shunsukew)
- **Status:** Draft
- **Created:** 2023-04-03
- **Reference Implementation** https://github.com/swanky-dapps/meta-transaction

## Summary

A standard for Meta transactions at the contract level running on Substrate's [contracts-pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts), thereby facilitating seamless user experiences free from gas costs. Notably, this standard does not require any modifications at the protocol level (pallet-contracts); instead, Forwarder contracts incorporate an AccountId to call inputs as additional bytes, while Recipient contracts retrieve the effective caller from `self._caller()` as opposed to `self.env().caller()`. The ultimate goal of this proposition is to align the Meta transaction framework for WASM smart contracts with the EVM ecosystem's [ERC2771](https://eips.ethereum.org/EIPS/eip-2771), guaranteeing the compatibility of Forwarder and Recipient contracts.


## Motivation

The motivation behind proposing a meta transaction standard for Substrate-based WASM smart contracts is to improve the user experience, reduce the barrier of entry for new users, and promote widespread adoption of the WASM contracts ecosystem. By implementing a standard for meta transactions at the contract level, developers can enable gasless interactions with smart contracts, creating a more inclusive and accessible environment for users.


## Specification

### Definitions

Terminology definitions are same as ERC2771.

Quotes
> `Transaction Signer`: Signs & sends transactions to a Gas Relay
>
> `Gas Relay`: Receives signed requests off-chain from Transaction Signers and pays gas to turn it into a valid transaction that goes through a Trusted Forwarder
>
> `Trusted Forwarder`: A contract trusted by the Recipient to correctly verify signatures and nonces before forwarding the request from Transaction Signers
>
> `Recipient`: A contract that accepts meta-transactions through a Trusted Forwarder


### Workflow

![](/src/psp-57/workflow.jpg)


### Extract Transaction Signer's AccountId

Trusted Forwarder contracts bear the responsibility of appending an additional 32 bytes of Call input, containing the Transactions Signer's AccountId, to inform Recipient contracts of the original signer of the transaction.

```
ExecutionInput::new(Selector::new(ink::selector_bytes!("example")))
    .push_arg(CallInput(&example_input))
    .push_arg(signer.encode()) // Append SCALE encoded AccountId as extra 32bytes
```

In order for Recipient contracts to extract the Transaction Signer's AccountId, they must follow the following steps:

1. Check if the contract is called by a Trusted Forwarder.
2. The last 32 bytes of the call input must be extracted to retrieve the Transaction Signer's AccountId.
3. This AccountId is then used as the original caller of the transaction, as opposed to the default self.env().caller(). In cases where the caller is not a Trusted Forwarder, `self.env().caller()` is used as it is.

Recipient contracts must ensure extra AccountId bytes are appended by Trusted Forwarder, otherwise contracts' message get executed with a forged AccountId as the original caller.

## Copyright

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
