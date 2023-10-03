# Fungible Token Standard for Substrate's `contracts` pallet

- **PSP Number:** 55
- **Authors:** Konrad Wierzbik <konrad.wierzbik@gmail.com >
- **Status:** Draft
- **Created:** 2022-10-31

## Summary

A standard for a fungible token interface for WebAssembly smart contracts which run on Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts).

This proposal aims to define the standard fungible token interface for WebAssembly smart contracts, simillar to [PSP22](https://github.com/w3f/PSPs/blob/master/PSPs/psp-22.md).
## Motivation

PSP22 has a limited use case to representing "positve value" assets. It is not possible to use PSP22 to represent "negative value" assets like debts.
The limitation comes from the mechanism of transfers and allowances. In the PSP22 case anyone can always transfer tokens to anyone. However if asset represents a debt is should not be possible to transfer the debt to other user unlees the other user gave allowance to the sender.

## Implementations
not tested implementation:
https://github.com/DesiredDesire/openbrush-contracts/tree/main/contracts/src/token/psp55


# This standard is at ABI level

Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts) can execute any WebAssembly contract that implements its defined API; we do not want to restrain this standard to only Rust and [the ink! language](https://github.com/paritytech/ink), but make it possible to be implemented by any language/framework that compiles to WebAssembly.

## Specification
1. [Interfaces](#Interfaces)
2. [Events](#Events)
3. [Types](#Types)
4. [Errors](#Errors)

### Interfaces

#### PSP-55 Interface

This section defines the required interface for this standard.

##### **total_supply()** ➔ Balance
Selector: `0xe232aa19` - first 4 bytes of `blake2b_256("PSP55::total_supply")`
```json
{
  "args": [],
  "docs": [
    "Returns the total token supply."
  ],
  "mutates": false,
  "name": [
    "PSP55",
    "total_supply"
  ],
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": "Balance"
  },
  "selector": "0xe232aa19"
}
```

##### **balance_of**(owner: AccountId) ➔ Balance
Selector: `0xc086b8cb` - first 4 bytes of `blake2b_256("PSP55::balance_of")`
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
    "PSP55",
    "balance_of"
  ],
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": "Balance"
  },
  "selector": "0xc086b8cb"
}
```

##### **allowance**(spender: AccountId, owner: AccountId) ➔ Balance
Selector: `0xe080062b` - first 4 bytes of `blake2b_256("PSP55::allowance")`
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
    "PSP55",
    "allowance"
  ],
  "returnType": {
    "displayName": [
      "Balance"
    ],
    "type": "Balance"
  },
  "selector": "0xe080062b"
}
```

##### **transfer**(to: AccountId, value: Balance, data: [u8]) ➔ Result<(), PSP55Error>
Selector: `0xd50dc37f` - first 4 bytes of `blake2b_256("PSP55::transfer")`
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
    " Transfers `value` amount of tokens from the caller's account to account `to`",
    " with additional `data` in unspecified format.",
    "",
    " On success a `Transfer` event is emitted.",
    "",
    " # Errors",
    "",
    " /// Returns `InsufficientAllowance` error if there are not enough tokens allowed",
    " for the caller to send to `to`.",
    "",
    " Returns `InsufficientBalance` error if there are not enough tokens on",
    " the caller's account Balance.",
    "",
    " Returns `ZeroSenderAddress` error if sender's address is zero.",
    "",
    " Returns `ZeroRecipientAddress` error if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP55",
    "transfer"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0xd50dc37f"
}
```

##### **transfer_from**(from: AccountId, to: AccountId, value: Balance, data: [u8]) ➔ Result<(), PSP55Error>
Selector: `0x828ca044` - first 4 bytes of `blake2b_256("PSP55::transfer_from")`
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
    " Returns `InsufficientAllowance` error if there are not enough tokens allowed",
    " for the caller to send to `to`.",
    "",
    " Returns `InsufficientBalance` error if there are not enough tokens on",
    " the the account Balance of `from`.",
    "",
    " Returns `ZeroSenderAddress` error if sender's address is zero.",
    "",
    " Returns `ZeroRecipientAddress` error if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP55",
    "transfer_from"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x828ca044"
}
```

##### **approve**(spender: AccountId, value: Balance) ➔ Result<(), PSP55Error>
Selector: `0x739254b6` - first 4 bytes of `blake2b_256("PSP55::approve")`
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
    " Allows `spender` to transfer to the caller's account multiple times, up to",
    " the `value` amount.",
    "",
    " If this function is called again it overwrites the current allowance with `value`.",
    "",
    " An `Approval` event is emitted.",
    "",
    " # Errors",
    "",
    " Returns `ZeroSenderAddress` error if sender's address is zero.",
    "",
    " Returns `ZeroRecipientAddress` error if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP55",
    "approve"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0x739254b6"
}

```

##### **increase_allowance**(spender: AccountId, delta_value: Balance) ➔ Result<(), PSP55Error>
Selector: `0xe080062b` - first 4 bytes of `blake2b_256("PSP55::increase_allowance")`
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
    " Atomically increases the allowance granted to `spender` by the caller.",
    "",
    " An `Approval` event is emitted.",
    "",
    " # Errors",
    "",
    " Returns `ZeroSenderAddress` error if sender's address is zero.",
    "",
    " Returns `ZeroRecipientAddress` error if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP55",
    "increase_allowance"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0xe080062b"
}
```

##### **decrease_allowance**(spender: AccountId, delta_value: Balance) ➔ Result<(), PSP55Error>
Selector: `0xe1fa11d7` - first 4 bytes of `blake2b_256("PSP55::decrease_allowance")`
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
    " Atomically decreases the allowance granted to `spender` by the caller.",
    "",
    " An `Approval` event is emitted.",
    "",
    " # Errors",
    "",
    " Returns `InsufficientAllowance` error if there are not enough tokens allowed",
    " by owner for `spender`.",
    "",
    " Returns `ZeroSenderAddress` error if sender's address is zero.",
    "",
    " Returns `ZeroRecipientAddress` error if recipient's address is zero."
  ],
  "mutates": true,
  "name": [
    "PSP55",
    "decrease_allowance"
  ],
  "returnType": {
    "displayName": [
      "Result"
    ],
    "type": 1
  },
  "selector": "0xe1fa11d7"
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
    "Event emitted when an approval occurs that `spender` is allowed to transfer",
    "up to the amount of `value` tokens to `owner`."
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
                      "PSP55Error"
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
                      "PSP55ReceiverError"
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
enum PSP55Error {
    /// Custom error type for cases in which an implementation adds its own restrictions.
    Custom(String),
    /// Returned if not enough balance to fulfill a request is available.
    InsufficientBalance,
    /// Returned if not enough allowance to fulfill a request is available.
    InsufficientAllowance,
    /// Returned if recipient's address is zero.
    ZeroRecipientAddress,
    /// Returned if sender's address is zero.
    ZeroSenderAddress,
    /// Returned if a safe transfer check fails (e.g. if the receiving contract does not accept tokens).
    SafeTransferCheckFailed(String),
}

enum PSP55ReceiverError {
    /// Returned if a transfer is rejected.
    TransferRejected(String),
}
```

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
