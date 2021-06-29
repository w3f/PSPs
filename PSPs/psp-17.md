# PSP-17 Token Standard in Ink!

- **PSP Number:** 17
- **Authors:** Green Baneling <green.baneling@supercolony.net>, Markian <markian@supercolony.net>, Pierre Ossun <pierre.ossun@supercolony.net>, Sven <sven.seven@supercolony.net>
- **Status:** Draft
- **Created:** 2021-06-19
- **Reference Implementation:** [OpenBrush](https://github.com/Supercolony-net/openbrush-contracts/blob/main/contracts/token/psp20/impls.rs)


## Summary

A standard interface for Ink! tokens.

This proposal aims to define the standard token in ink! smart contracts, in the same way of EIP-20 for Ethereum ecosystem (https://github.com/ethereum/EIPs/edit/master/EIPS/eip-20.md).

## Implementation

Example implementation:

- [OpenBrush](https://github.com/Supercolony-net/openbrush-contracts/blob/main/contracts/token/psp20/impls.rs)

## Motivation

A standard interface allows any Ink! tokens on Polkadot/Kusama to be re-used by other applications: from wallets to decentralized exchanges.


## Motivation for having a standard separate from ERC20

Due to different nature of ink!'s the Token Standard should be have ink! specific rules and methods. 
Therefore different name - PSP-20.


## Specification

The main motivation for this proposal is to have one **trait** that shares the same **trait naming** between all implementations,
as naming of trait affects the identifiers of functions in this trait.
The second motivation is to define an exhaustive method list in this trait. Unlike ERC20, we suggest including `increase_allowance` & `decrease_allowance`
as a part of standard proposal and extract metadata fields to separate trait.
Another suggestion is to have `PSP17Receiver` and call `on_received` method at the end of transfer if the recipient is a contract.

### Traits

```rust
/// PSP17 is a trait of Fungible Token Standard
pub trait PSP17 {
 /// Returns the total token supply.
 fn total_supply(&self) -> Balance;

 /// Returns the account balance for the specified `owner`.
 /// Returns `0` if the account is non-existent.
 fn balance_of(&self, owner: AccountId) -> Balance;

 /// Returns the amount which `spender` is still allowed to withdraw from `owner`.
 /// Returns `0` if no allowance has been set.
 fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance;

 /// Transfers `value` amount of tokens from the caller's account to account `to`
 /// with additional `data` with no specified format.
 /// Emits a `Transfer` event on success.
 /// 
 /// This method also calls [on_received](#on_received) method on `to`.
 /// 
 /// **Errors**
 /// * Panics with `InsufficientBalance` error if there are not enough tokens on
 /// the caller's account balance.
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 fn transfer(&mut self, to: AccountId, value: Balance, data: Vec<u8>);

 /// Transfers `value` tokens on behalf of `from` to the account `to` 
 /// with additional `data` with no specified format.
 /// This can be used to allow a contract to transfer tokens on ones behalf and/or 
 /// to charge fees in sub-currencies, for example.
 /// Emits `Transfer` and `Approval` events on success.
 /// 
 /// This method also calls [on_received](#on_received) method on `to`.
 /// 
 /// **Errors**
 /// * Panics with `InsufficientAllowance` error if there are not enough tokens allowed for the caller to withdraw from `from`.
 /// * Panics with `InsufficientBalance` error if there are not enough tokens on the account balance of `from`.
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 fn transfer_from(&mut self, from: AccountId, to: AccountId, value: Balance, data: Vec<u8>);

 /// Allows `spender` to withdraw from the caller's account multiple times, up to the `value` amount.
 /// If this function is called again it overwrites the current allowance with `value`.
 /// Emits `Approval` event.
 /// 
 /// **Errors**
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 fn approve(&mut self, spender: AccountId, value: Balance);

 /// Atomically increases the allowance granted to `spender` by the caller by `delta_value`.
 /// Emits `Approval` event.
 /// 
 /// **Errors**
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 fn increase_allowance(&mut self, spender: AccountId, delta_value: Balance);

 /// Atomically decreases the allowance granted to `spender` by the caller by `delta_value`.
 /// Emits `Approval` event.
 /// 
 /// **Errors**
 /// * Panics with `InsufficientAllowance` error if there are not enough tokens allowed by owner for `spender`.
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 fn decrease_allowance(&mut self, spender: AccountId, delta_value: Balance);
}

/// PSP17Metadata is an optional trait of metadata for Fungible Token Standard
pub trait PSP17Metadata {
 /// Returns the token name.
 fn name(&self) -> Option<String>;

 /// Returns the token symbol.
 fn symbol(&self) -> Option<String>;

 /// Returns the token decimals.
 fn decimals(&self) -> u8;
}

/// PSP17Receiver is a trait for any contract that wants to support safe transfers
/// from PSP17 token smart contracts to avoid unexpected tokens on balance of contract.
pub trait PSP17Receiver {
 /// Handle the receipt of a PSP17 token by a smart contract.
 /// Returns `Ok(())` if the contract has accepted the token(s) and `Err(PSP17ReceiverError::TransferRejected(String))` otherwise.
 /// 
 /// This method will get called on every transfer to check whether the recipient in `transfer` is a contract, and if it is,
 /// does it accept tokens. This is done to prevent contracts from locking tokens forever.
 /// 
 /// This method does not throw. Returns `PSP17ReceiverError` if the contract does not accept the tokens.
 fn on_received(&mut self, operator: AccountId, from: AccountId, value: Balance, data: Vec<u8>) -> Result<(), PSP17ReceiverError>;
}
```

### Types
```rust
// AccountId is 32 bytes array, like in substrate-based blockchains.
type AccountId = [u8; 32];
// u128 must be enough to cover most of the use cases of standard token.
type Balance = u128;
```

### Events
The identifiers of events must be based on the name of the trait. At the moment, ink! doesn't support it,
but it must be fixed with this [issue](https://github.com/paritytech/ink/issues/809). 

#### Transfer 
Must be emitted when a token transfer occurs.
When a contract creates (mints) new tokens, `from` will be `None`
When a contract deletes (burns) tokens, `to` will be `None`
```rust
struct Transfer {
 from: Option<AccountId>,
 to: Option<AccountId>,
 value: Balance
}
```

#### Approval
Must be emitted when an approval occurs that `spender` is allowed to withdraw up to the amount of `value` tokens from `owner`.
```rust
struct Approval {
 owner: AccountId,
 spender: AccountId,
 value: Balance
}
```

### Errors
Suggested methods don't return `Result` (except `on_received`). Instead, they panic.
This panic can contain one of the following messages:

```rust
pub enum PSP17Error {
 /// Unknown error type for cases if writer of traits added own restrictions
 Unknown(String),
 /// Returned if not enough balance to fulfill a request is available.
 InsufficientBalance,
 /// Returned if not enough allowance to fulfill a request is available.
 InsufficientAllowance,
 /// Returned if recipient's address is zero.
 ZeroRecipientAddress,
 /// Returned if sender's address is zero.
 ZeroSenderAddress,
 /// Returned if safe transfer check fails (see _do_safe_transfer_check() in PSP17 trait)
 SafeTransferCheckFailed(String),
}

pub enum PSP17ReceiverError {
 /// Returned if a transfer is rejected.
 TransferRejected(String),
}
```
## Copyright

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).

