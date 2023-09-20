# Multi Token Standard for Substrate's `contracts` pallet

- **PSP Number:** 37
- **Authors:** Pierre Ossun <pierre.ossun@supercolony.net>, Green Baneling <green.baneling@supercolony.net>, Markian <markian@supercolony.net>
- **Status:** Published
- **Created:** 2022-03-01
- **Reference Implementation** [OpenBrush](https://github.com/Brushfam/openbrush-contracts/blob/main/contracts/src/token/psp37/psp37.rs)

## Summary

A standard for a Multi Token interface for WebAssembly smart contracts which run on Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts).

This proposal aims to define the standard Multi Token interface for WebAssembly smart contracts, just like [EIP-1155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md) for the Ethereum ecosystem.

## Motivation

Without a standard interface for Multi Token every contract will have different signature and types. Hence, no interoperability is possible.
This proposal aims to resolve that by defining one **interface** that shares the same **ABI** of **permissionless** methods between all implementations.

The goal is to have a standard contract interface that allows tokens deployed on Substrate's `contracts` pallet to be re-used by other applications: from wallets to decentralized exchanges.

## Implementations
Examples of implementations:

- [OpenBrush](https://github.com/Brushfam/openbrush-contracts/blob/main/contracts/src/token/psp37/psp37.rs), written in [ink!](https://github.com/paritytech/ink).

## Motivation for having a standard separate from ERC-1155
Due to the different nature of WebAssembly smart contracts and the difference between EVM and the [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts) in Substrate, this standard proposal has specific rules and methods,
therefore PSP-37 differs from ERC-1155 in its implementation.

Also the proposal contains new:
- types to keep interoperability with Fungible and Non-Fungible tokens standard defined in previous PSPs.
- methods to introduce new features

# This standard is at ABI level

Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts) can execute any WebAssembly contract that implements its defined API; we do not want to restrain this standard to only Rust and [the ink! language](https://github.com/paritytech/ink), but make it possible to be implemented by any language/framework that compiles to WebAssembly.

## Specification
1. [Interfaces](#Interfaces)
2. [Extension](#Extension)
3. [Events](#Events)
4. [Types](#Types)
5. [Errors](#Errors)

### Interfaces

#### PSP-37 Interface

This section defines the required interface for this standard.

##### **balance_of**(owner: AccountId, id: Option<Id>) ➔ Balance
Selector: `0xc42919e2` - first 4 bytes of `blake2b_256("PSP37::balance_of")`
```json
{
  "args": [
    {
      "label": "owner",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "id",
      "type": {
        "displayName": [
          "Option"
        ],
        "type": "Option<Id>"
      }
    }
  ],
  "docs": [
    " Returns the amount of tokens of token type `id` owned by `account`.",
    "",
    " If `id` is `None` returns the total number of `owner`'s tokens."
  ],
  "label": "PSP37::balance_of",
  "mutates": false,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": "Balance"
  },
  "selector": "0xc42919e2"
}
```

##### **total_supply**(id: Option<Id>) ➔ Balance
Selector: `0x9a49e85a` - first 4 bytes of `blake2b_256("PSP37::total_supply")`
```json
{
  "args": [
    {
      "label": "id",
      "type": {
        "displayName": [
          "Option"
        ],
        "type": "Option<Id>"
      }
    }
  ],
  "docs": [
    " Returns the total amount of token type `id` in the supply.",
    "",
    " If `id` is `None` returns the total number of tokens."
  ],
  "label": "PSP37::total_supply",
  "mutates": false,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": "Balance"
  },
  "selector": "0x9a49e85a"
}
```

##### **allowance**(owner: AccountId, operator: AccountId, id: Option<Id>) ➔ Balance
Selector: `0xcb78a065` - first 4 bytes of `blake2b_256("PSP37::allowance")`
```json
{
  "args": [
    {
      "label": "owner",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "operator",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "id",
      "type": {
        "displayName": [
          "Option"
        ],
        "type": "Option<Id>"
      }
    }
  ],
  "docs": [
    " Returns amount of `id` token of `owner` that `operator` can withdraw",
    " If `id` is `None` returns allowance `Balance::MAX` of all tokens of `owner`"
  ],
  "label": "PSP37::allowance",
  "mutates": false,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": "Balance"
  },
  "selector": "0xcb78a065"
}
```

##### **approve**(operator: AccountId, id: Option<Id>, value: Balance) ➔ Result<(), PSP37Error>
Selector: `0x31a1a453` - first 4 bytes of `blake2b_256("PSP37::approve")`
```json
{
  "args": [
    {
      "label": "operator",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "id",
      "type": {
        "displayName": [
          "Option"
        ],
        "type": "Option<Id>"
      }
    },
    {
      "label": "value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    }
  ],
  "docs": [
    " Allows `operator` to withdraw the `id` token from the caller's account",
    " multiple times, up to the `value` amount.",
    " If this function is called again it overwrites the current allowance with `value`",
    " If `id` is `None` approves or disapproves the operator for all tokens of the caller.",
    "",
    " An `Approval` event is emitted."
  ],
  "label": "PSP37::approve",
  "mutates": true,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": 1
  },
  "selector": "0x31a1a453"
}
```

##### **transfer**(to: AccountId, id: Id, value: Balance, data: [u8]) ➔ Result<(), PSP37Error>
Selector: `0x04e09961` - first 4 bytes of `blake2b_256("PSP37::transfer")`
```json
{
  "args": [
    {
      "label": "to",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "id",
      "type": {
        "displayName": [
          "Id"
        ],
        "type": "Id"
      }
    },
    {
      "label": "value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    },
    {
      "label": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [
    " Transfers `value` of `id` token from `caller` to `to`",
    "",
    " On success a `TransferSingle` event is emitted.",
    "",
    " # Errors",
    "",
    " Returns `TransferToZeroAddress` error if recipient is zero account.",
    "",
    " Returns `NotAllowed` error if transfer is not approved.",
    "",
    " Returns `InsufficientBalance` error if `caller` doesn't contain enough balance.",
    "",
    " Returns `SafeTransferCheckFailed` error if `to` doesn't accept transfer."
  ],
  "label": "PSP37::transfer",
  "mutates": true,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": 1
  },
  "selector": "0x04e09961"
}
```

##### **transfer_from**(from: AccountId, to: AccountId, id: Id, value: Balance, data: [u8]) ➔ Result<(), PSP37Error>
Selector: `0x5cf8b7d4` - first 4 bytes of `blake2b_256("PSP37::transfer_from")`
```json
{
  "args": [
    {
      "label": "from",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "to",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "id",
      "type": {
        "displayName": [
          "Id"
        ],
        "type": "Id"
      }
    },
    {
      "label": "amount",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    },
    {
      "label": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [
    " Transfers `amount` tokens of token type `id` from `from` to `to`. Also some `data` can be passed.",
    "",
    " On success a `TransferSingle` event is emitted.",
    "",
    " # Errors",
    "",
    " Returns `TransferToZeroAddress` error if recipient is zero account.",
    "",
    " Returns `NotAllowed` error if transfer is not approved.",
    "",
    " Returns `InsufficientBalance` error if `from` doesn't contain enough balance.",
    "",
    " Returns `SafeTransferCheckFailed` error if `to` doesn't accept transfer."
  ],
  "label": "PSP37::transfer_from",
  "mutates": true,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": 1
  },
  "selector": "0x5cf8b7d4"
}
```

### Extension

#### PSP37Metadata

`PSP37Metadata` is an **optional** extension for this Multi Token standard to provide information regarding each token.

##### **get_attribute**(id: Id, key: [u8]) ➔ Option<[u8]>
Selector: `0x61dda97c` - first 4 bytes of `blake2b_256("PSP37Metadata::get_attribute")`
```json
{
  "args": [
    {
      "label": "id",
      "type": {
        "displayName": [
          "Id"
        ],
        "type": "Id"
      }
    },
    {
      "label": "key",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [],
  "label": "PSP37Metadata::get_attribute",
  "mutates": false,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": "Option<[u8]>"
  },
  "selector": "0x61dda97c"
}
```

Attributes are more flexible than single `uri` function like in [`Erc1155`](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md).
The list of required attributes for Multi token should be defined in a separate
proposal based on the scope of the usage.

#### PSP37Batch

`PSP37Batch` is an **optional** extension for this Multi Token standard to support batch operations.


##### **batch_transfer**(to: AccountId, ids_amounts: [(Id, Balance)], data: [u8]) ➔ Result<(), PSP37Error>
Selector: `0x9bfb1d2b` - first 4 bytes of `blake2b_256("PSP37Batch::batch_transfer")`
```json
{
  "args": [
    {
      "label": "to",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "ids_amounts",
      "type": {
        "displayName": [
          "[(Id, Balance)]"
        ],
        "type": "[(Id, Balance)]"
      }
    },
    {
      "label": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [],
  "label": "PSP37Batch::batch_transfer",
  "mutates": true,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": 1
  },
  "selector": "0x9bfb1d2b"
}
```

##### **batch_transfer_from**(from: AccountId, to: AccountId, ids_amounts: [(Id, Balance)], data: [u8]) ➔ Result<(), PSP37Error>
Selector: `0xf4ebeed2` - first 4 bytes of `blake2b_256("PSP37Batch::batch_transfer_from")`
```json
{
  "args": [
    {
      "label": "from",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "to",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "ids_amounts",
      "type": {
        "displayName": [
          "[(Id, Balance)]"
        ],
        "type": "[(Id, Balance)]"
      }
    },
    {
      "label": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [],
  "label": "PSP37Batch::batch_transfer_from",
  "mutates": true,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": 1
  },
  "selector": "0xf4ebeed2"
}
```

#### PSP37Enumerable

`PSP37Enumerable` is an **optional** extension for this Multi Token standard to support enumeration of tokens.


##### **owners_token_by_index**(owner: AccountId, index: u128) ➔ Option<Id>
Selector: `0x4cc01ee0` - first 4 bytes of `blake2b_256("PSP37Enumerable::owners_token_by_index")`
```json
{
  "args": [
    {
      "label": "owner",
      "type": {
        "displayName": [
          "AccountId"
        ],
        "type": "AccountId"
      }
    },
    {
      "label": "index",
      "type": {
        "displayName": [
          "u128"
        ],
        "type": "u128"
      }
    }
  ],
  "docs": [
    " Returns a token `Id` owned by `owner` at a given `index` of its token list.",
    " Use along with `balance_of` to enumerate all of ``owner``'s tokens."
  ],
  "label": "PSP37Enumerable::owners_token_by_index",
  "mutates": false,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": "Option<Id>"
  },
  "selector": "0x4cc01ee0"
}
```

##### **token_by_index**(index: u128) -> Option<Id>
Selector: `0x127b5477` - first 4 bytes of `blake2b_256("PSP37Enumerable::token_by_index")`
```json
{
  "args": [
    {
      "label": "index",
      "type": {
        "displayName": [
          "u128"
        ],
        "type": "u128"
      }
    }
  ],
  "docs": [
    " Returns a token `Id` at a given `index` of all the tokens stored by the contract.",
    " Use along with `total_supply` to enumerate all tokens."
  ],
  "label": "PSP37Enumerable::token_by_index",
  "mutates": false,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": "Option<Id>"
  },
  "selector": "0x127b5477"
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
      "label": "from",
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
      "label": "to",
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
      "label": "id",
      "type": {
        "displayName": [
          "Id"
        ],
        "type": "Id"
      }
    },
    {
      "docs": [],
      "indexed": false,
      "label": "value",
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
  "label": "Transfer"
}
```

#### TransferBatch
When a contract creates (mints) new tokens, `from` will be `None`.
When a contract deletes (burns) tokens, `to` will be `None`.
```json
{
  "args": [
    {
      "docs": [],
      "indexed": true,
      "label": "from",
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
      "label": "to",
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
      "label": "ids_amounts",
      "type": {
        "displayName": [
          "[(Id, Balance)]"
        ],
        "type": "[(Id, Balance)]"
      }
    }
  ],
  "docs": [
    "Event emitted when a batch token transfer occurs."
  ],
  "label": "Transfer"
}
```

### Approval
```json
{
  "args": [
    {
      "docs": [],
      "indexed": true,
      "label": "owner",
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
      "label": "operator",
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
      "label": "id",
      "type": {
        "displayName": [
          "Option<Id>"
        ],
        "type": "Option<Id>"
      }
    },
    {
      "docs": [],
      "indexed": false,
      "label": "value",
      "type": {
        "displayName": [
          "Balance"
        ],
        "type": "Balance"
      }
    }
  ],
  "docs": [
    "Event emitted when a token approve occurs."
  ],
  "label": "Approval"
}
```

### AttributeSet
```json
{
  "args": [
    {
      "docs": [],
      "indexed": false,
      "label": "id",
      "type": {
        "displayName": [
          "Id"
        ],
        "type": "Id"
      }
    },
    {
      "docs": [],
      "indexed": false,
      "label": "key",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    },
    {
      "docs": [],
      "indexed": false,
      "label": "data",
      "type": {
        "displayName": [
          "[u8]"
        ],
        "type": "[u8]"
      }
    }
  ],
  "docs": [
    "Event emitted when an attribute is set for a token.",
  ],
  "label": "AttributeSet"
}
```

### Types
```rust
// Id is an Enum and its variant are types
enum Id {
    U8(u8),
    U16(u16),
    U32(u32),
    U64(u64),
    U128(u128),
    Bytes(Vec<u8>),
}

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
                            "name": "SelfApprove"
                          },
                          {
                            "name": "NotApproved"
                          },
                          {
                            "name": "TokenExists"
                          },
                          {
                            "name": "TokenNotExists"
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
                      "PSP37Error"
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
enum PSP37Error {
    /// Custom error type for cases if writer of traits added own restrictions
    Custom(String),
    /// Returned if owner approves self
    SelfApprove,
    /// Returned if the caller doesn't have allowance for transferring.
    NotApproved,
    /// Returned if the owner already own the token.
    TokenExists,
    /// Returned if the token doesn't exist
    TokenNotExists,
    /// Returned if safe transfer check fails
    SafeTransferCheckFailed(String),
}
```

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
