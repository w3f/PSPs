# Multi Token Standard for Substrate's `contracts` pallet

- **PSP Number:** 35
- **Authors:** Pierre Ossun <pierre.ossun@supercolony.net>, Green Baneling <green.baneling@supercolony.net>, Markian <markian@supercolony.net>
- **Status:** Draft
- **Created:** 2022-03-01
- **Reference Implementation** [OpenBrush](https://github.com/Supercolony-net/openbrush-contracts/blob/main/contracts/src/token/psp1155/psp1155.rs)

## Summary

A standard for a Multi Token interface for WebAssembly smart contracts which run on Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts).

This proposal aims to define the standard Multi Token interface for WebAssembly smart contracts, just like [EIP-1155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md) for the Ethereum ecosystem.

## Motivation

Without a standard interface for Multi Token every contract will have different signature and types. Hence, no interoperability is possible.
This proposal aims to resolve that by defining one **interface** that shares the same **ABI** of **permissionless** methods between all implementations.

The goal is to have a standard contract interface that allows tokens deployed on Substrate's `contracts` pallet to be re-used by other applications: from wallets to decentralized exchanges.

## Implementations
Examples of implementations:

- [OpenBrush](https://github.com/Supercolony-net/openbrush-contracts/blob/main/contracts/src/token/psp1155/psp1155.rs), written in [ink!](https://github.com/paritytech/ink).

## Motivation for having a standard separate from ERC-1155
Due to the different nature of WebAssembly smart contracts and the difference between EVM and the [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts) in Substrate, this standard proposal has specific rules and methods,
therefore PSP-35 differs from ERC-1155 in its implementation. 

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

#### PSP-35 Interface

This section defines the required interface for this standard.

##### **balance_of**(owner: AccountId, id: Id) ➔ Balance
Selector: `0x6dfd737e` - first 4 bytes of `blake2b_256("PSP35::balance_of")`
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
          "Id"
        ],
        "type": "Id"
      }
    }
  ],
  "docs": [
    "Returns the amount of tokens of token type `id` owned by `account`.",
  ],
  "mutates": false,
  "label": "PSP35::balance_of",
  "payable": false,
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": 1
  },
  "selector": "0x6dfd737e"
}
```

##### **allowance**(owner: AccountId, operator: AccountId, id: Option<Id>) ➔ Balance
Selector: `0xdf7a9d29` - first 4 bytes of `blake2b_256("PSP35::allowance")`
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
    "Returns the amount of `id` token which `operator` is still",
    "allowed to withdraw from `owner`.",
    "Returns `0` if no allowance has been set.",
    "If `id` is `None` then the non zero return value means ",
    "that `operator` can withdraw any amount of any token."
  ],
  "mutates": false,
  "label": "PSP35::allowance",
  "payable": false,
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": "Balance"
  },
  "selector": "0xdf7a9d29"
}
```

##### **approve**(operator: AccountId, id: Option<Id>, value: Balance) ➔ Result<(), PSP35Error>
Selector: `0xadeb64c6` - first 4 bytes of `blake2b_256("PSP35::approve")`
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
    "Allows `operator` to withdraw  the `id` token from the caller's account",
    "multiple times, up to the `value` amount.",
    "",
    "If this function is called again it overwrites the current allowance with `value`.",
    "If `id` is `None` approves or disapproves the operator for all tokens of the caller.",
    "",
    "An `Approval` event is emitted."
  ],
  "mutates": true,
  "label": "PSP35::approve",
  "payable": false,
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0xadeb64c6"
}
```

##### **transfer**(to: AccountId, id: Id, value: Balance, data: [u8]) ➔ Result<(), PSP35Error>
Selector: `0x43742281` - first 4 bytes of `blake2b_256("PSP35::transfer")`
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
      "name": "value",
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
    "Transfers to `to` the `value` of approved or owned `id` token from caller.",
    "",
    "On success a `Transfer` event is emitted.",
    "",
    "# Errors",
    "",
    "Returns `TokenNotExists` error if `id` does not exist.",
    "",
    "Returns `SafeTransferCheckFailed` error if `to` doesn't accept transfer."
  ],
  "mutates": true,
  "label": "PSP35::transfer",
  "payable": false,
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x43742281"
}
```

##### **transfer_from**(from: AccountId, to: AccountId, id: Id, value: Balance, data: [u8]) ➔ Result<(), PSP35Error>
Selector: `0xbd18d9d7` - first 4 bytes of `blake2b_256("PSP35::transfer_from")`
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
      "name": "value",
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
    "Transfers from `from` to `to` the `value` of approved or owned `id` token from caller.",
    "",
    "On success a `Transfer` event is emitted.",
    "",
    "# Errors",
    "",
    "Returns `TokenNotExists` error if `id` does not exist.",
    "",
    "Returns `NotApproved` error if `from` doesn't have allowance for transferring.",
    "",
    "Returns `SafeTransferCheckFailed` error if `to` doesn't accept transfer."
  ],
  "mutates": true,
  "label": "PSP35::transfer_from",
  "payable": false,
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0xbd18d9d7"
}
```

#### PSP35Receiver
`PSP35Receiver` is an interface for any contract that wants to support safe transfers from a PSP-35 token smart contract to avoid unexpected tokens in the balance of contract.
This method is called before a transfer to ensure the recipient of the tokens acknowledges the receipt.

##### **before_received**(operator: AccountId, from: AccountId, ids_amounts: [(Id, Balance)], data: [u8]) ➔ Result<(), PSP35ReceiverError>
Selector: `0x0d04ad35` - first 4 bytes of `blake2b_256("PSP35Receiver::before_received")`
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
      "label": "from",
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
  "docs": [
    "Ensures that the smart contract allows reception of PSP35 token(s).",
    "Returns `Ok(())` if the contract allows the reception of the token(s) and Error `TransferRejected(String)` otherwise.",
    "",
    "This method will get called on every transfer to check whether the recipient in `transfer`",
    "`transfer_from`, `transfer_batch` or `transfer_from_batch`, is a contract, and if it is, does it accept tokens.",
    "This is done to prevent contracts from locking tokens forever.",
    "",
    "Returns `PSP35ReceiverError` if the contract does not accept the tokens."
  ],
  "mutates": true,
  "label": "PSP35Receiver::before_received",
  "payable": false,
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 2
  },
  "selector": "0x0d04ad35"
}
```

### Extension

#### PSP35Metadata

`PSP35Metadata` is an **optional** extension for this Multi Token standard to provide information regarding each token.

##### **get_attribute**(id: Id, key: [u8]) ➔ Option<[u8]>
Selector: `0xc0eb7172` - first 4 bytes of `blake2b_256("PSP35Metadata::get_attribute")`
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
  "docs": [
    "Returns the attribute of `id` for the given `key`.",
    "",
    "If `id` is a collection id of the token, it returns attributes for collection."
  ],
  "mutates": false,
  "label": "PSP35Metadata::get_attribute",
  "payable": false,
  "returnType": {
    "displayName": [
      "Option"
    ],
    "type": "Option<[u8]>"
  },
  "selector": "0xc0eb7172"
}
```

Attributes are more flexible than single `uri` function like in [`Erc1155`](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md).
The list of required attributes for Multi token should be defined in a separate
proposal based on the scope of the usage.

#### PSP35Batch

`PSP35Batch` is an **optional** extension for this Multi Token standard to support batch operations.


##### **transfer**(to: AccountId, ids_amounts: [(Id, Balance)], data: [u8]) ➔ Result<(), PSP35Error>
Selector: `0x3ea08b55` - first 4 bytes of `blake2b_256("PSP35Batch::transfer")`
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
  "docs": [
    "Batched version of `PSP35::transfer` method."
  ],
  "mutates": true,
  "label": "PSP35Batch::transfer",
  "payable": false,
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x3ea08b55"
}
```

##### **transfer_from**(from: AccountId, to: AccountId, ids_amounts: [(Id, Balance)], data: [u8]) ➔ Result<(), PSP35Error>
Selector: `0x4dc1123e` - first 4 bytes of `blake2b_256("PSP35Batch::transfer_from")`
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
  "docs": [
    "Batched version of `PSP35::transfer_from` method."
  ],
  "mutates": true,
  "label": "PSP35Batch::transfer_from",
  "payable": false,
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x4dc1123e"
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
                      "PSP35Error"
                    ]
                  }
                }
              ],
              "name": "Err"
            }
          ]
        }
      }
    },
    "2": {
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
                            "name": "TransferRejected"
                          }
                        ]
                      }
                    },
                    "path": [
                      "PSP35ReceiverError"
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
enum PSP35Error {
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

enum PSP35ReceiverError {
    /// Returned if transfer is rejected.
    TransferRejected(String),
}
```

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
