# Fungible Token Standard for Substrate's `contracts` pallet

- **PSP Number:** 62
- **Authors:** Konrad Wierzbik <konrad.wierzbik@gmail.com>,
- **Status:** Draft
- **Created:** 2023-07-20
- **Reference Implementation:** None


## Summary

A standard for a fungible token interface for WebAssembly smart contracts which run on Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts).

This proposal aims to extend and improve the [PSP22 token](https://github.com/w3f/PSPs/blob/master/PSPs/psp-22.md) by adding only one method transfer_from_with_signature(from: AccountId, to: AccountId, value: Balamce, signature: \[u8\], data: \[u8\] ) which allows transferring "value" of tokens from account "from" to account "to" without the need of account "to" to have an allowance from account "from" but with the need to provide the appropriate message signed by the account "from".

With the transfer_from_with_signature method, one will be able to construct a DeFi application that will be more convenient and secure for every user.


## Motivation

The PSP22 and ERC20 are very inconvenient and somehow unsecure for every user. These problems come from the mechanism of giving other accounts an allowance to spend tokens.

A lot of users' transactions include interaction with smart contracts (like Dexes, Lending Protocols, Yield Optimizers, Bridges, etc.) that want to spend users' tokens. In most of the DeFi platforms users before interaction with the smart contract need to give the smart contract allowance, so it is allowed to access the users' funds. 

Such a design creates inconvenience and unnecessary costs for DeFi users because they need to send two transactions.

Moreover, it puts additional responsibility and unnecessary risk on the user. Until very recently, when giving allowance to the smart contract most platforms and wallets by default vere set U256_MAX as the "amount to be allowed" parameter. This is of course, very convenient for the user if he wants to interact with the smart contract often but it also creates the risk in the case of the smart contract upgrade. If the smart contract logic is maliciously upgraded then the user is at risk of losing all of his funds. Thus we can see that the design we have right now (PSP22 and ERC20) put users at play between convenience and risk.

Here I propose a modified PSP22 token that solves these problems. The idea is to add an additional method transfer_from_with_signature(from: AccountId, to: AccountId, value: Balamce, signature: \[u8\], data: \[u8\] ) which allows transferring "value" of tokens from account "from" to account "to" without the need of account "to" to have an allowance from account "from" but with the need to provide the appropriate message signed by the account "from". The signed message should be Encoded struct {from: AccountId, to: AccountId, value: Balamce, data: \[u8\], nonce: u64 }.

## Implementations
None

# This standard is at ABI level

Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts) can execute any WebAssembly contract that implements its defined API; we do not want to restrain this standard to only Rust and [the ink! language](https://github.com/paritytech/ink), but make it possible to be implemented by any language/framework that compiles to WebAssembly.

## Specification
1. [Interfaces](#Interfaces)
2. [Events](#Events)
3. [Types](#Types)
4. [Errors](#Errors)

### Interfaces

#### PSP-22 Interface

This section defines the required interface for this standard.

##### **total_supply()** ➔ Balance
Selector: `0x6fafb584` - first 4 bytes of `blake2b_256("PSP62::total_supply")`
```json
{
  "args": [],
  "docs": [
    "Returns the total token supply."
  ],
  "mutates": false,
  "name": [
    "PSP62",
    "total_supply"
  ],
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": "Balance"
  },
  "selector": "0x6fafb584"
}
```

##### **balance_of**(owner: AccountId) ➔ Balance
Selector: `0x9455ef9c` - first 4 bytes of `blake2b_256("PSP62::balance_of")`
```json
{
  "args": [
    {
      "name": "owner",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    }
  ],
  "docs": [
    "Returns the account balance for the specified `owner`.",
    "",
    "Returns `0` if the account is non-existent."
  ],
  "mutates": false,
  "name": [
    "PSP62",
    "balance_of"
  ],
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": "Balance"
  },
  "selector": "0x9455ef9c"
}
```

##### **allowance**(owner: AccountId, spender: AccountId) ➔ Balance
Selector: `0x7046304b` - first 4 bytes of `blake2b_256("PSP62::allowance")`
```json
{
  "args": [
    {
      "name": "owner",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "spender",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    }
  ],
  "docs": [
    "Returns the amount which `spender` is still allowed to withdraw from `owner`.",
    "",
    "Returns `0` if no allowance has been set."
  ],
  "mutates": false,
  "name": [
    "PSP62",
    "allowance"
  ],
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": "Balance"
  },
  "selector": "0x7046304b"
}
```

##### **transfer**(to: AccountId, value: Balance, data: [u8]) ➔ Result<(), PSP62Error>
Selector: `0xfa47a20e` - first 4 bytes of `blake2b_256("PSP62::transfer")`
```json
{
  "args": [
    {
      "name": "to",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    },
    {
      "name": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [
    "Transfers `value` amount of tokens from the caller's account to account `to`",
    "with additional `data` in unspecified format.",
    "",
    "On success a `Transfer` event is emitted.",
    "",
    "# Errors",
    "",
    "Reverts with error `InsufficientBalance` if there are not enough tokens on",
    "the caller's account Balance.",
    "",
    "Reverts with error `ZeroSenderAddress` if sender's address is zero.",
    "",
    "Reverts with error `ZeroRecipientAddress` if recipient's address is zero."
    "Reverts with error `SafeTransferCheckFailed` if the recipient is a contract and rejected the transfer."
  ],
  "mutates": true,
  "name": [
    "PSP62",
    "transfer"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0xfa47a20e"
}
```

##### **transfer_from**(from: AccountId, to: AccountId, value: Balance, data: [u8]) ➔ Result<(), PSP62Error>
Selector: `0x7d3184e8` - first 4 bytes of `blake2b_256("PSP62::transfer_from")`
```json
{
  "args": [
    {
      "name": "from",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "to",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    },
    {
      "name": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [
    "Transfers `value` tokens on the behalf of `from` to the account `to`",
    "with additional `data` in unspecified format.",
    "",
    "This can be used to allow a contract to transfer tokens on ones behalf and/or",
    "to charge fees in sub-currencies, for example.",
    "",
    "On success a `Transfer` and `Approval` events are emitted.",
    "",
    "# Errors",
    "",
    "Reverts with error `InsufficientAllowance` if there are not enough tokens allowed",
    "for the caller to withdraw from `from`.",
    "",
    "Reverts with error `InsufficientBalance` if there are not enough tokens on",
    "the the account Balance of `from`.",
    "",
    "Reverts with error `ZeroSenderAddress` if sender's address is zero.",
    "",
    "Reverts with error `ZeroRecipientAddress` if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP62",
    "transfer_from"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x7d3184e8"
}
```

##### **transfer_from_with_signature**(from: AccountId, to: AccountId, signature: [u8], value: Balance, data: [u8]) ➔ Result<(), PSP62Error>
Selector: `0xd93eb9e9` - first 4 bytes of `blake2b_256("PSP62::transfer_from_with_signature")`
```json
{
  "args": [
    {
      "name": "from",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "to",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    },
    {
      "name": "signature",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    },
    {
      "name": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [
    "Transfers `value` tokens on the behalf of `from` to the account `to`",
    "with additional `data` in unspecified format.",
    "",
    "This can be used to allow a contract to transfer tokens on ones behalf and/or",
    "to charge fees in sub-currencies, for example.",
    "",
    "On success a `Transfer` event is emitted.",
    "",
    "# Errors",
    "",
    "Reverts with error `IncorrectSignature` if the provided signature is wrong.",
    "",
    "Reverts with error `InsufficientBalance` if there are not enough tokens on",
    "the the account Balance of `from`.",
    "",
    "Reverts with error `ZeroSenderAddress` if sender's address is zero.",
    "",
    "Reverts with error `ZeroRecipientAddress` if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP62",
    "transfer_from"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0xd93eb9e9"
}
```

##### **approve**(spender: AccountId, value: Balance) ➔ Result<(), PSP62Error>
Selector: `0x813987e2` - first 4 bytes of `blake2b_256("PSP62::approve")`
```json
{
  "args": [
    {
      "name": "spender",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    }
  ],
  "docs": [
    "Allows `spender` to withdraw from the caller's account multiple times, up to",
    "the `value` amount.",
    "",
    "If this function is called again it overwrites the current allowance with `value`.",
    "",
    "An `Approval` event is emitted.",
    "",
    "# Errors",
    "",
    "Reverts with error `ZeroSenderAddress` if sender's address is zero.",
    "",
    "Reverts with error `ZeroRecipientAddress` if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP62",
    "approve"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x813987e2"
}

```

##### **increase_allowance**(spender: AccountId, delta_value: Balance) ➔ Result<(), PSP62Error>
Selector: `0x6dc933e9` - first 4 bytes of `blake2b_256("PSP62::increase_allowance")`
```json
{
  "args": [
    {
      "name": "spender",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "delta_value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    }
  ],
  "docs": [
    "Atomically increases the allowance granted to `spender` by the caller.",
    "",
    "An `Approval` event is emitted.",
    "",
    "# Errors",
    "",
    "Reverts with error `ZeroSenderAddress` if sender's address is zero.",
    "",
    "Reverts with error `ZeroRecipientAddress` if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP62",
    "increase_allowance"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x6dc933e9"
}
```

##### **decrease_allowance**(spender: AccountId, delta_value: Balance) ➔ Result<(), PSP62Error>
Selector: `0x51cd44c5` - first 4 bytes of `blake2b_256("PSP62::decrease_allowance")`
```json
{
  "args": [
    {
      "name": "spender",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "name": "delta_value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    }
  ],
  "docs": [
    "Atomically decreases the allowance granted to `spender` by the caller.",
    "",
    "An `Approval` event is emitted.",
    "",
    "# Errors",
    "",
    "Reverts with error `InsufficientAllowance` if there are not enough tokens allowed",
    "by owner for `spender`.",
    "",
    "Reverts with error `ZeroSenderAddress` if sender's address is zero.",
    "",
    "Reverts with error `ZeroRecipientAddress` if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP62",
    "decrease_allowance"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x51cd44c5"
}
```

#### PSP62Metadata 

`PSP62Metadata` is an optional interface for metadata for this fungible token standard.

##### **token_name**() ➔ Option<String>
Selector: `0x0b2a8b00` - first 4 bytes of `blake2b_256("PSP62Metadata::token_name")`
```json
{
  "args": [],
  "docs": [
    "Returns the token name."
  ],
  "mutates": false,
  "name": [
    "PSP62Metadata",
    "token_name"
  ],
  "returnType": {
    "displayName": [
      "Option"
    ],
    "type": "Option<string>"
  },
  "selector": "0x0b2a8b00"
}
```

##### **token_symbol**() ➔ Option<String>
Selector: `0xe3e783e1` - first 4 bytes of `blake2b_256("PSP62Metadata::token_symbol")`
```json
{
  "args": [],
  "docs": [
    "Returns the token symbol."
  ],
  "mutates": false,
  "name": [
    "PSP62Metadata",
    "token_symbol"
  ],
  "returnType": {
    "displayName": [
      "Option"
    ],
    "type": "Option<string>"
  },
  "selector": "0xe3e783e1"
}
```

##### **token_decimals**() ➔ u8
Selector: `0x548402b8` - first 4 bytes of `blake2b_256("PSP62Metadata::token_decimals")`
```json
{
  "args": [],
  "docs": [
    "Returns the token decimals."
  ],
  "mutates": false,
  "name": [
    "PSP62Metadata",
    "token_decimals"
  ],
  "returnType": {
    "displayName": [
      "u8"
    ],
    "type": "u8"
  },
  "selector": "0x548402b8"
}
```

### Events

#### Transfer 
When a contract creates (mints) new tokens, `from` will be `None`.
When a contract deletes (burns) tokens, `to` will be `None`.
```json
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
        "type": "Option<AccountId>"
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
        "type": "Option<AccountId>"
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
        "type": "Balance"
      }
    }
  ],
  "docs": [
    "Event emitted when a token transfer occurs."
  ],
  "name": "Transfer"
}
```

### Approval
```json
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
        "type": "AccountId"
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
        "type": "AccountId"
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
        "type": "Balance"
      }
    }
  ],
  "docs": [
    "Event emitted when an approval occurs that `spender` is allowed to withdraw",
    "up to the amount of `value` tokens from `owner`."
  ],
  "name": "Approval"
}
```

### Types
```rust
// AccountId is a 32 bytes Array, like in Substrate-based blockchains.
type AccountId = [u8; 32];

// `u128` must be enough to cover most of the use-cases of standard tokens.
type Balance = u128;
```

#### Return types
```json
{
  "types": {
    "1": {
      "def": {
        "variant": {
          "variants": [
            {
              "fields": [
                {
                  "type": {
                    "def": {
                      "tuple": []
                    }
                  }
                }
              ],
              "name": "Ok"
            },
            {
              "fields": [
                {
                  "type": {
                    "def": {
                      "variant": {
                        "variants": [
                          {
                            "fields": [
                              {
                                "type": "string"
                              }
                            ],
                            "name": "Custom"
                          },
                          {
                            "name": "InsufficientBalance"
                          },
                          {
                            "name": "InsufficientAllowance"
                          },
                          {
                            "name": "ZeroRecipientAddress"
                          },
                          {
                            "name": "ZeroSenderAddress"
                          },
                          {
                            "fields": [
                              {
                                "type": "string"
                              }
                            ],
                            "name": "SafeTransferCheckFailed"
                          }
                        ]
                      }
                    },
                    "path": [
                      "PSP62Error"
                    ]
                  }
                }
              ],
              "name": "Err"
            }
          ]
        }
      }
    }
  }
}
```

### Errors
The suggested methods revert the transaction and return a [SCALE-encoded](https://github.com/paritytech/parity-scale-codec) `Result` type with one of the following `Error` enum variants:

```rust
enum PSP62Error {
    /// Custom error type for cases in which an implementation adds its own restrictions.
    Custom(String),
    /// Returned if not enough balance to fulfill a request is available.
    InsufficientBalance,
    /// Returned if not enough allowance to fulfill a request is available.
    InsufficientAllowance,
    /// Returned if signature is incorrect.
    IncorrectSignature,
    /// Returned if recipient's address is zero.
    ZeroRecipientAddress,
    /// Returned if sender's address is zero.
    ZeroSenderAddress,
    /// Returned if a safe transfer check fails (e.g. if the receiving contract does not accept tokens).
    SafeTransferCheckFailed(String),
}
```
