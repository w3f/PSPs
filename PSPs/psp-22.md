# PSP-22 Fungible Token Standard in Ink!

- **PSP Number:** 22
- **Authors:** Green Baneling <green.baneling@supercolony.net>, Markian <markian@supercolony.net>, Pierre <pierre.ossun@supercolony.net>, Sven <sven.seven@supercolony.net>, Varg <varg.vikernes@supercolony.net>
- **Status:** Draft
- **Created:** 2021-06-19
- **Reference Implementation:** [OpenBrush](https://github.com/Supercolony-net/openbrush-contracts/blob/main/contracts/token/psp20/traits.rs)


## Summary

A standard interface for WASM contracts Fungible Tokens.

This proposal aims to define the standard fungible token in WASM smart contracts, just like [EIP-20](https://github.com/ethereum/EIPs/edit/master/EIPS/eip-20.md) for Ethereum ecosystem.

## Importance
Currently, while there is no standard, every contract will have different signature. Thus, no interoperability is possible. This proposal aims to resolve that by having one **interface** that shares the
same **ABI** between all implementations.

## Implementation
Example of Ink! implementation:

- [OpenBrush](https://github.com/Supercolony-net/openbrush-contracts/blob/main/contracts/token/psp20/traits.rs)

## Motivation
A standard interface allows any tokens on Polkadot/Kusama to be re-used by other applications: from wallets to decentralized exchanges.


## Motivation for having a standard separate from ERC20
Due to different nature of WASM smart contracts and the difference between EVM and pallet-contract in substrate, the standard should have specific rules and methods,
therefore PSP-22 differs from ERC-20 in its implementation.

Also, this standard proposal defines an extensive method list in the(interface. Unlike ERC20, it includes `increase_allowance` & `decrease_allowance`, and defines metadata fields as part of a separate interface.
Another difference is that it has `PSP22Receiver` interface, and `on_received` method is called at the end of transfer if the recipient is a contract.

# This standard is at ABI

As contract-pallet in Substrate can execute any WASM contracts, we should not restrain this standard to only Rust & Ink! Framework.
So that it can be use by any language/framework that compile to WASM.
The full ABI JSON for interface and events can be found at the top down of this proposal


## Specification
1. [Interface](#Interface)
2. [Events](#Events)
3. [Types](#Types)
4. [Errors](#Errors)

### Message

```json
/// PSP22 is an interface of Fungible Token Standard.
PSP22 {
 /// Returns the total token supply.
 total_supply() -> Balance;

 /// Returns the account balance for the specified `owner`.
 /// Returns `0` if the account is non-existent.
 balance_of(owner: AccountId) -> Balance;

 /// Returns the amount which `spender` is still allowed to withdraw from `owner`.
 /// Returns `0` if no allowance has been set.
 allowance(owner: AccountId, spender: AccountId) -> Balance;

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
 transfer(to: AccountId, value: Balance, data: Vec<u8>);

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
 transfer_from(from: AccountId, to: AccountId, value: Balance, data: Vec<u8>);

 /// Allows `spender` to withdraw from the caller's account multiple times, up to the `value` amount.
 /// If this function is called again it overwrites the current allowance with `value`.
 /// Emits `Approval` event.
 /// 
 /// **Errors**
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 approve(spender: AccountId, value: Balance);

 /// Atomically increases the allowance granted to `spender` by the caller by `delta_value`.
 /// Emits `Approval` event.
 /// 
 /// **Errors**
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 Increase_allowance(spender: AccountId, delta_value: Balance);

 /// Atomically decreases the allowance granted to `spender` by the caller by `delta_value`.
 /// Emits `Approval` event.
 /// 
 /// **Errors**
 /// * Panics with `InsufficientAllowance` error if there are not enough tokens allowed by owner for `spender`.
 /// * Panics with `ZeroSenderAddress` error if sender's address is zero.
 /// * Panics with `ZeroRecipientAddress` error if recipient's address is zero.
 decrease_allowance(spender: AccountId, delta_value: Balance);
}

/// PSP22Metadata is an optional interface of metadata for Fungible Token Standard.
PSP22Metadata {
 /// Returns the token name.
 fn name() -> Option<String>;

 /// Returns the token symbol.
 fn symbol() -> Option<String>;

 /// Returns the token decimals.
 fn decimals() -> u8;
}

/// PSP22Receiver is an interface for any contract that wants to support safe transfers
/// from PSP22 token smart contracts to avoid unexpected tokens on balance of contract.
PSP22Receiver {
 /// Handle the receipt of a PSP22 token by a smart contract.
 /// Returns `Ok(())` if the contract has accepted the token(s) and `Err(PSP22ReceiverError::TransferRejected(String))` otherwise.
 /// 
 /// This method will get called on every transfer to check whether the recipient in `transfer` is a contract, and if it is,
 /// does it accept tokens. This is done to prevent contracts from locking tokens forever.
 /// 
 /// This method does not throw. Returns `PSP22ReceiverError` if the contract does not accept the tokens.
 on_received(&mut self, operator: AccountId, from: AccountId, value: Balance, data: Vec<u8>) -> Result<(), PSP22ReceiverError>;
}
```

### Events

‼️ Important ‼️

Events are not supported currently due to how ink! currently handles them.  
The identifiers of events must be based on the name of the trait. At the moment, ink! doesn't support it,
but it must be fixed with this [issue](https://github.com/paritytech/ink/issues/809). 

#### Transfer 
Must be emitted when a token transfer occurs.
When a contract creates (mints) new tokens, `from` will be `None`
When a contract deletes (burns) tokens, `to` will be `None`
```json
 Transfer {
 "from": "Option<AccountId>",
 "to": "Option<AccountId>",
 "value": "Balance"
}
```

#### Approval
Must be emitted when an approval occurs that `spender` is allowed to withdraw up to the amount of `value` tokens from `owner`.
```json
Approval {
 "owner": "AccountId",
 "spender": "AccountId",
 "value": "Balance"
}
```

### Types
```json
// AccountId is 32 bytes array, like in substrate-based blockchains.
type AccountId = [u8; 32];
// u128 must be enough to cover most of the use cases of standard token.
type Balance = u128;
```

### Errors
Suggested methods don't return `Result` (except `on_received`). Instead, they panic.
This panic can contain one of the following messages:

```json
PSP22Error {
 /// Custom error type for cases if writer of traits added own restrictions
 Custom(String),
 /// Returned if not enough balance to fulfill a request is available.
 InsufficientBalance,
 /// Returned if not enough allowance to fulfill a request is available.
 InsufficientAllowance,
 /// Returned if recipient's address is zero.
 ZeroRecipientAddress,
 /// Returned if sender's address is zero.
 ZeroSenderAddress,
 /// Returned if safe transfer check fails (see _do_safe_transfer_check() in PSP22 trait)
 SafeTransferCheckFailed(String),
}

PSP22ReceiverError {
 /// Returned if a transfer is rejected.
 TransferRejected(String),
}
```

## ABI

```json
{
    "events": [
      {
        "args": [
          {
            "docs": [],
            "indexed": true,
            "name": "from",
            "type": {
              "displayName": [
                "Option"
              ],
              "type": 15
            }
          },
          {
            "docs": [],
            "indexed": true,
            "name": "to",
            "type": {
              "displayName": [
                "Option"
              ],
              "type": 15
            }
          },
          {
            "docs": [],
            "indexed": false,
            "name": "value",
            "type": {
              "displayName": [
                "Balance"
              ],
              "type": 1
            }
          }
        ],
        "docs": [
          " Event emitted when a token transfer occurs."
        ],
        "name": "Transfer"
      },
      {
        "args": [
          {
            "docs": [],
            "indexed": true,
            "name": "owner",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "docs": [],
            "indexed": true,
            "name": "spender",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "docs": [],
            "indexed": false,
            "name": "value",
            "type": {
              "displayName": [
                "Balance"
              ],
              "type": 1
            }
          }
        ],
        "docs": [
          " Event emitted when an approval occurs that `spender` is allowed to withdraw",
          " up to the amount of `value` tokens from `owner`."
        ],
        "name": "Approval"
      }
    ],
    "messages": [
      {
        "args": [
          {
            "name": "from",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "name": "to",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "name": "value",
            "type": {
              "displayName": [
                "Balance"
              ],
              "type": 1
            }
          },
          {
            "name": "data",
            "type": {
              "displayName": [
                "Vec"
              ],
              "type": 14
            }
          }
        ],
        "docs": [
          " Transfers `value` tokens on the behalf of `from` to the account `to`",
          " with additional `data` in unspecified format.",
          "",
          " This can be used to allow a contract to transfer tokens on ones behalf and/or",
          " to charge fees in sub-currencies, for example.",
          "",
          " On success a `Transfer` and `Approval` events are emitted.",
          "",
          " # Errors",
          "",
          " Panics with `InsufficientAllowance` error if there are not enough tokens allowed",
          " for the caller to withdraw from `from`.",
          "",
          " Panics with `InsufficientBalance` error if there are not enough tokens on",
          " the the account Balance of `from`.",
          "",
          " Panics with `ZeroSenderAddress` error if sender's address is zero.",
          "",
          " Panics with `ZeroRecipientAddress` error if recipient's address is zero."
        ],
        "mutates": true,
        "name": [
          "PSP20",
          "transfer_from"
        ],
        "payable": false,
        "returnType": null,
        "selector": "0x9eb3870e"
      },
      {
        "args": [
          {
            "name": "spender",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "name": "delta_value",
            "type": {
              "displayName": [
                "Balance"
              ],
              "type": 1
            }
          }
        ],
        "docs": [
          " Atomically decreases the allowance granted to `spender` by the caller.",
          "",
          " An `Approval` event is emitted.",
          "",
          " # Errors",
          "",
          " Panics with `InsufficientAllowance` error if there are not enough tokens allowed",
          " by owner for `spender`.",
          "",
          " Panics with `ZeroSenderAddress` error if sender's address is zero.",
          "",
          " Panics with `ZeroRecipientAddress` error if recipient's address is zero."
        ],
        "mutates": true,
        "name": [
          "PSP20",
          "decrease_allowance"
        ],
        "payable": false,
        "returnType": null,
        "selector": "0x1f86d882"
      },
      {
        "args": [
          {
            "name": "spender",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "name": "delta_value",
            "type": {
              "displayName": [
                "Balance"
              ],
              "type": 1
            }
          }
        ],
        "docs": [
          " Atomically increases the allowance granted to `spender` by the caller.",
          "",
          " An `Approval` event is emitted.",
          "",
          " # Errors",
          "",
          " Panics with `ZeroSenderAddress` error if sender's address is zero.",
          "",
          " Panics with `ZeroRecipientAddress` error if recipient's address is zero."
        ],
        "mutates": true,
        "name": [
          "PSP20",
          "increase_allowance"
        ],
        "payable": false,
        "returnType": null,
        "selector": "0xf43f7266"
      },
      {
        "args": [
          {
            "name": "owner",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          }
        ],
        "docs": [
          " Returns the account Balance for the specified `owner`.",
          "",
          " Returns `0` if the account is non-existent."
        ],
        "mutates": false,
        "name": [
          "PSP20",
          "balance_of"
        ],
        "payable": false,
        "returnType": {
          "displayName": [
            "Balance"
          ],
          "type": 1
        },
        "selector": "0x936205de"
      },
      {
        "args": [
          {
            "name": "spender",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "name": "value",
            "type": {
              "displayName": [
                "Balance"
              ],
              "type": 1
            }
          }
        ],
        "docs": [
          " Allows `spender` to withdraw from the caller's account multiple times, up to",
          " the `value` amount.",
          "",
          " If this function is called again it overwrites the current allowance with `value`.",
          "",
          " An `Approval` event is emitted.",
          "",
          " # Errors",
          "",
          " Panics with `ZeroSenderAddress` error if sender's address is zero.",
          "",
          " Panics with `ZeroRecipientAddress` error if recipient's address is zero."
        ],
        "mutates": true,
        "name": [
          "PSP20",
          "approve"
        ],
        "payable": false,
        "returnType": null,
        "selector": "0xf229fa2f"
      },
      {
        "args": [
          {
            "name": "owner",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "name": "spender",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          }
        ],
        "docs": [
          " Returns the amount which `spender` is still allowed to withdraw from `owner`.",
          "",
          " Returns `0` if no allowance has been set `0`."
        ],
        "mutates": false,
        "name": [
          "PSP20",
          "allowance"
        ],
        "payable": false,
        "returnType": {
          "displayName": [
            "Balance"
          ],
          "type": 1
        },
        "selector": "0x467e8bff"
      },
      {
        "args": [],
        "docs": [
          " Returns the total token supply."
        ],
        "mutates": false,
        "name": [
          "PSP20",
          "total_supply"
        ],
        "payable": false,
        "returnType": {
          "displayName": [
            "Balance"
          ],
          "type": 1
        },
        "selector": "0xd1ff92bd"
      },
      {
        "args": [
          {
            "name": "to",
            "type": {
              "displayName": [
                "AccountId"
              ],
              "type": 5
            }
          },
          {
            "name": "value",
            "type": {
              "displayName": [
                "Balance"
              ],
              "type": 1
            }
          },
          {
            "name": "data",
            "type": {
              "displayName": [
                "Vec"
              ],
              "type": 14
            }
          }
        ],
        "docs": [
          " Transfers `value` amount of tokens from the caller's account to account `to`",
          " with additional `data` in unspecified format..",
          "",
          " On success a `Transfer` event is emitted.",
          "",
          " # Errors",
          "",
          " Panics with `InsufficientBalance` error if there are not enough tokens on",
          " the caller's account Balance.",
          "",
          " Panics with `ZeroSenderAddress` error if sender's address is zero.",
          "",
          " Panics with `ZeroRecipientAddress` error if recipient's address is zero."
        ],
        "mutates": true,
        "name": [
          "PSP20",
          "transfer"
        ],
        "payable": false,
        "returnType": null,
        "selector": "0x18532c7c"
      },
      {
        "args": [],
        "docs": [
          " Returns the token decimals."
        ],
        "mutates": false,
        "name": [
          "PSP20Metadata",
          "token_decimals"
        ],
        "payable": false,
        "returnType": {
          "displayName": [
            "u8"
          ],
          "type": 7
        },
        "selector": "0x997ad16c"
      },
      {
        "args": [],
        "docs": [
          " Returns the token name."
        ],
        "mutates": false,
        "name": [
          "PSP20Metadata",
          "token_name"
        ],
        "payable": false,
        "returnType": {
          "displayName": [
            "Option"
          ],
          "type": 13
        },
        "selector": "0x9c994fe4"
      },
      {
        "args": [],
        "docs": [
          " Returns the token symbol."
        ],
        "mutates": false,
        "name": [
          "PSP20Metadata",
          "token_symbol"
        ],
        "payable": false,
        "returnType": {
          "displayName": [
            "Option"
          ],
          "type": 13
        },
        "selector": "0x10972330"
      }
    ]
}
```

## Copyright

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).

