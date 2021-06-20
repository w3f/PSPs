# PSP-20 Token Standard in Ink!

- **PSP Number:** 17
- **Authors:** TODO
- **Status:** Draft
- **Created:** 2021 06 19
- **Reference Implementation** NA

## Summary

This proposal aims to define the standard token in ink! smart contracts, in the same way of EIP-20 for ethereum ecosystem (https://eips.ethereum.org/EIPS/eip-20).

## Motivation

Due to some ink! specificities, that differ from solidity smart contract development, the Token Standard should be adapted to ink!.
Also calling it PSP-20 makes more sense as the implementation differs from the solidity ERC20 standards.

The goal is to build, as OpenZeppelin for ethereum ecosystem, a set of standards for commonly used contracts in ink! called OpenBrush.

The main motivation for this proposal is to have one **trait** that shares the same **trait naming** between all implementations.
Because naming of trait affects the identifiers of functions in this trait. It is why the name of the trait must be the same across all implementations.
The second motivation is to define the exhaustive method list in this trait. Unlike ERC-20, we suggest include `increase_allowance` & `decrease_allowance`
as part of the standard proposal and extract metadata fields to separate trait.

## Specification

### Types
```rust
// AccountId is 32 bytes array like in substrate-based blockchains.
type AccountId = [u8; 32];
// u128 must be enough to cover most of the use cases of standard token.
type Balance = u128;
```

### Traits

```rust
pub trait IPSP20 {
 fn total_supply(&self) -> Balance;

 fn balance_of(&self, owner: AccountId) -> Balance;

 fn transfer(&mut self, to: AccountId, value: Balance);

 fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance;

 fn transfer_from(&mut self, from: AccountId, to: AccountId, value: Balance);

 fn approve(&mut self, spender: AccountId, value: Balance);

 fn increase_allowance(&mut self, spender: AccountId, delta_value: Balance);

 fn decrease_allowance(&mut self, spender: AccountId, delta_value: Balance);
}

pub trait IPSP20Metadata {
 fn token_name(&self) -> Option<String>;

 fn token_symbol(&self) -> Option<String>;

 fn token_decimals(&self) -> u8;
}
```
### Events
The identifiers of events must be based on name of trait. At the moment ink! doesn't support it,
but it must be fixed during this [issue](https://github.com/paritytech/ink/issues/809). 

#### Transfer 
Must ber emitted when a token transfer occurs.
When contract creates (mint) new tokens, `from` will be `None`
When contract deletes (burn) tokens, `to` will be `None`
```rust
Transfer
from: Option<AccountId>,
to: Option<AccountId>,
value: Balance,
```

#### Approval
Must be emitted when an approval occurs that `spender` is allowed to withdraw up to the amount of `value` tokens from `owner`.
```rust
Approval
owner: AccountId,
spender: AccountId,
value: Balance,
```

### Errors
Suggested methods doesn't return `Result`, Suggested methods don't return `Result`, instead of this, it throws a panic.
But this panic can contain one of the following messages.

```rust
pub enum Erc20Error {
 /// Unknown error type for cases if writer of traits added own restrictions
 Unknown(&'static str),
 /// Returned if not enough balance to fulfill a request is available.
 InsufficientBalance,
 /// Returned if not enough allowance to fulfill a request is available.
 InsufficientAllowance,
 /// Returned if recipient's address is zero.
 ZeroRecipientAddress,
 /// Returned if sender's address is zero.
 ZeroSenderAddress,
}
```

### Methods

#### token_name
Returns the token name.
```rust
fn token_name(&self) -> Option<String>;
```

#### token_symbol
Returns the token symbol.
```rust
fn token_symbol(&self) -> Option<String>;
```

#### token_decimals
Returns the token decimals.
```rust
fn token_decimals(&self) -> u8;
```

#### total_supply
Returns the total token supply.
```rust
fn total_supply(&self) -> Balance;
```

#### balance_of
Returns the account balance for the specified `owner`.
Returns `0` if the account is non-existent.
```rust
fn balance_of(&self, owner: AccountId) -> Balance;
```

#### transfer
Transfers `value` amount of tokens from the caller's account to account `to`.
 On success a `Transfer` event is emitted.

**Errors**
* Panics `InsufficientBalance` error if there are not enough tokens on
the caller's account Balance.
* Panics `ZeroSenderAddress` error if sender's address is zero.
* Panics `ZeroRecipientAddress` error if recipient's address is zero.
```rust
fn transfer(&mut self, to: AccountId, value: Balance);
```

#### allowance
Returns the amount which `spender` is still allowed to withdraw from `owner`.
Returns `0` if no allowance has been set `0`.
```rust
fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance;
```

#### transfer_from
Transfers `value` tokens on the behalf of `from` to the account `to`.
This can be used to allow a contract to transfer tokens on ones behalf and/or to charge fees in sub-currencies,for example.
On success a `Transfer` and `Approval` events are emitted.

**Errors**
* Panics `InsufficientAllowance` error if there are not enough tokens allowed for the caller to withdraw from `from`.
* Panics `InsufficientBalance` error if there are not enough tokens on the the account Balance of `from`.
* Panics `ZeroSenderAddress` error if sender's address is zero.
* Panics `ZeroRecipientAddress` error if recipient's address is zero.
```rust
fn transfer_from(&mut self, from: AccountId, to: AccountId, value: Balance);
```

#### approve
Allows `spender` to withdraw from the caller's account multiple times, up to the `value` amount.
If this function is called again it overwrites the current allowance with `value`.
An `Approval` event is emitted.

**Errors**
* Panics `ZeroSenderAddress` error if sender's address is zero.
* Panics `ZeroRecipientAddress` error if recipient's address is zero.
```rust
fn approve(&mut self, spender: AccountId, value: Balance);
```

#### increase_allowance
Atomically increases the allowance granted to `spender` by the caller on `delta_value`.
An `Approval` event is emitted.

**Errors**
* Panics `ZeroSenderAddress` error if sender's address is zero.
* Panics `ZeroRecipientAddress` error if recipient's address is zero.
```rust
fn increase_allowance(&mut self, spender: AccountId, delta_value: Balance);
```

#### decrease_allowance
Atomically decreases the allowance granted to `spender` by the caller on `delta_value`.
An `Approval` event is emitted.

**Errors**
* Panics `InsufficientAllowance` error if there are not enough tokens allowed by owner for `spender`.
* Panics `ZeroSenderAddress` error if sender's address is zero.
* Panics `ZeroRecipientAddress` error if recipient's address is zero.
```rust
fn decrease_allowance(&mut self, spender: AccountId, delta_value: Balance);
```


## Copyright

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).

