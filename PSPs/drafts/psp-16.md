# PSP-20 Token Standard in Ink!

- **PSP Number:** 16
- **Authors:** TODO
- **Status:** Draft
- **Created:** 2021 06 19
- **Reference Implementation** NA

## Summary

This proposal aims to standarize the Token standard in Ink! smart contracts, in the same way of EIP-20 for ethereum ecosystem (https://eips.ethereum.org/EIPS/eip-20).

## Motivation

Due to some Ink! specificities, that differ from solidity smart contract development, the Token Standard should be adapted to Ink!.
Also calling it PSP-20 makes more sense as the implemntation differs from the solidity ERC20 standards.
The goal is to build, as Openzeppelin for ethereum ecosystem, a set of standards for commonly used contracts in Ink! called OpenBrush.

## Specification

### PSP-20 Contract

## Events

#### Transfer 
Must ber emitted when a token transfer occurs.
When contract creates new token, *from* takes the zero account argument AccountId::from([0x0; 32]).
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

## Errors
#### Insufficient Balance
Returned if not enough balance to fulfill a request is available.
```rust
InsufficientBalance
```

#### Insufficient Allowance
Returned if not enough allowance to fulfill a request is available.
```rust
InsufficientAllowance
```

#### Zero Recipient Address
Returned if recipient's address is zero.
```rust
ZeroRecipientAddress
```

#### Zero Sender Address
Returned if sender's address is zero.
```rust
ZeroSenderAddress
```

## Traits

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

