# Nesting NFTs

- **PSP Number:** [To be assigned (=number of the initial PR to the PSPs repo)]
- **Authors:** [boyswan](https://github.com/boyswan), [Maar-io](https://github.com/Maar-io), 
- **Status:** Draft 
- **Created:** [2023-05-08]
- **Reference Implementation** [Link to a first reference implementation](https://github.com/rmrk-team/rmrk-ink)

## Summary

An interface for Nestable Non-Fungible Tokens with emphasis on parent token's control over the relationship.

## Motivation

The Parent-Governed Nestable NFT standard extends PSP34 by allowing for a new inter-NFT relationship and interaction.

At its core, the idea behind the proposal is simple: the owner of an NFT does not have to be an Externally Owned Account (EOA) or a smart contract, it can also be an NFT.

The process of nesting an NFT into another is functionally identical to sending it to another user. The process of sending a token out of another one involves issuing a transaction from the account owning the parent token.

An NFT can be owned by a single other NFT, but can in turn have a number of NFTs that it owns. This proposal establishes the framework for the parent-child relationships of NFTs. A parent token is the one that owns another token. A child token is a token that is owned by another token. A token can be both a parent and child at the same time. Child tokens of a given token can be fully managed by the parent token's owner, but can be proposed by anyone.

## Specification

1. Interfaces
2. Events
3. Types
4. Errors

### Interfaces
This section defines the required interface for this standard.

#### Nesting::add_child(parent_token_id: Id, child_nft: ChildNft) -> Result<(), NestingError>

Selector: `0x1d6f5156`, first 4 bytes of `blake2b_256(Nesting::add_child)`
```json
{
    "args": [
        {
        "label": "parent_token_id",
        "type": {
            "displayName": [
            "ParentTokenId"
            ],
            "type": "Id"
        }
        },
        {
        "label": "child_nft",
        "type": {
            "displayName": [
            "ChildNft"
            ],
            "type": "ChildNft"
        }
        }
    ],
    "docs": [
        " Add a child NFT (from different collection) to the NFT in this (parent) collection.",
        " The status of the added child is `Pending` if caller is not owner of parent NFT",
        " The status of the added child is `Accepted` if caller is the owner of parent NFT",
        " The caller needs not to be the owner of the parent_token_id,",
        " but the Caller must be owner of the child NFT,",
        " in order to perform transfer() ownership of the child nft to parent_token_id.",
        "",
        " # Requirements:",
        " * `child_contract_address` needs to be added by collection owner",
        " * `to_parent_token_id` must exist.",
        " * `child_token_id` must exist.",
        " * There cannot be two identical children.",
        "",
        " # Arguments:",
        " * `to_parent_token_id`: is the tokenId of the parent NFT. The receiver of child.",
        " * `child_nft`: (collection_id, token_id) of the child instance.",
        "",
        " # Result:",
        " Ownership of child NFT will be transferred to this parent NFT",
        " On success emits `ChildAdded`",
        " On success emits `ChildAccepted` - only if caller is already the owner of the parent NFT"
    ],
    "label": "Nesting::add_child",
    "mutates": true,
    "payable": false,
    "returnType": {
        "displayName": [
        "MessageResult"
        ],
        "type": "Result<(), NestingError>"
    }
}
```

#### Nesting::remove_child(parent_token_id: Id, child_nft: ChildNft) -> Result<(), NestingError>

Selector: `0x27e7420e`, first 4 bytes of `blake2b_256(Nesting::remove_child)`
```json
{
    "args": [
        {
        "label": "parent_token_id",
        "type": {
            "displayName": [
            "ParentTokenId"
            ],
            "type": "Id"
        }
        },
        {
        "label": "child_nft",
        "type": {
            "displayName": [
            "ChildNft"
            ],
            "type": "ChildNft"
        }
        }
    ],
    "docs": [
        " Transfer the child NFT ownership from parent_token_id to the parent token owner.",
        "",
        " # Requirements:",
        " * The status of the child is `Accepted`",
        "",
        " # Arguments:",
        " * `parent_token_id`: is the tokenId of the parent NFT.",
        " * `child_nft`: (collection_id, token_id) of the child instance.",
        "",
        " # Result:",
        " Ownership of child NFT will be transferred to parent NFT owner",
        " On success emits `ChildRemoved`"
    ],
    "label": "Nesting::remove_child",
    "mutates": true,
    "payable": false,
    "returnType": {
        "displayName": [
        "MessageResult"
        ],
        "type": "Result<(), NestingError>"
    }
}
```

#### Nesting::accept_child(parent_token_id: Id, child_nft: ChildNft) -> Result<(), NestingError>

Selector: `0x3b3e2643`, first 4 bytes of `blake2b_256(Nesting::accept_child)`

```json
{
    "args": [
        {
        "label": "parent_token_id",
        "type": {
            "displayName": [
            "ParentTokenId"
            ],
            "type": "Id"
        }
        },
        {
        "label": "child_nft",
        "type": {
            "displayName": [
            "ChildNft"
            ],
            "type": "ChildNft"
        }
        }
    ],
    "docs": [
        " Accept a child NFT (from different collection) to be owned by parent token.",
        "",
        " # Requirements:",
        " * The status of the child is `Pending`",
        "",
        " # Arguments:",
        " * `parent_token_id`: is the tokenId of the parent NFT.",
        " * `child_nft`: (collection_id, token_id) of the child instance.",
        "",
        " # Result:",
        " Child Nft is moved from pending to accepted",
        " On success emits `ChildAccepted`"
    ],
    "label": "Nesting::accept_child",
    "mutates": true,
    "payable": false,
    "returnType": {
        "displayName": [
        "MessageResult"
        ],
        "type": "Result<(), NestingError>"
    }
}
```

### Nesting::reject_child(parent_token_id: Id, child_nft: ChildNft) -> Result<(), NestingError>

Selector: `0xdd308ed4`, first 4 bytes of `blake2b_256(Nesting::reject_child)`
```json
{
    "args": [
        {
        "label": "parent_token_id",
        "type": {
            "displayName": [
            "ParentTokenId"
            ],
            "type": "Id"
        }
        },
        {
        "label": "child_nft",
        "type": {
            "displayName": [
            "ChildNft"
            ],
            "type": "ChildNft"
        }
        }
    ],
    "docs": [
        " Reject a child NFT (from different collection).",
        "",
        " # Requirements:",
        " * The status of the child is `Pending`",
        "",
        " # Arguments:",
        " * `parent_token_id`: is the tokenId of the parent NFT.",
        " * `child_nft`: (collection_id, token_id) of the child instance.",
        "",
        " # Result:",
        " Child Nft is removed from pending",
        " On success emits `ChildRejected`"
    ],
    "label": "Nesting::reject_child",
    "mutates": true,
    "payable": false,
    "returnType": {
        "displayName": [
        "MessageResult"
        ],
        "type": "Result<(), NestingError>"
    }
}
```

#### Nesting::transfer_child(from: Id, to: Id, child_nft: ChildNft) -> Result<(), NestingError>;

Selector: `0xdb43324e`, first 4 bytes of `blake2b_256(Nesting::transfer_child)`
```json
{
    "args": [
        {
        "label": "from",
        "type": {
            "displayName": [
            "From"
            ],
            "type": "Id"
        }
        },
        {
        "label": "to",
        "type": {
            "displayName": [
            "To"
            ],
            "type": "Id"
        }
        },
        {
        "label": "child_nft",
        "type": {
            "displayName": [
            "ChildNft"
            ],
            "type": "ChildNft"
        }
        }
    ],
    "docs": [
        " Transfer the child NFT from one parent to another (in this collection).",
        "",
        " # Requirements:",
        " * The status of the child is `Accepted`",
        "",
        " # Arguments:",
        " * `current_parent`: current parent tokenId which holds child nft",
        " * `new_parent`: new parent tokenId which will hold child nft",
        " * `child_nft`: (collection_id, token_id) of the child instance.",
        "",
        " # Result:",
        " Ownership of child NFT will be transferred to this contract (cross contract call)",
        " On success emits `ChildAdded`",
        " On success emits `ChildAccepted` - only if caller is already owner of child NFT"
    ],
    "label": "Nesting::transfer_child",
    "mutates": true,
    "payable": false,
    "returnType": {
        "displayName": [
        "MessageResult"
        ],
        "type": "Result<(), NestingError>"
    }
}
```

#### Nesting::get_parent_of_child(&self, child_nft: ChildNft) -> Option<Id>

Selector: `0x40255e26`, first 4 bytes of `blake2b_256(Nesting::get_parent_of_child)`
```json
{
    "args": [
        {
        "label": "child_nft",
        "type": {
            "displayName": [
            "ChildNft"
            ],
            "type": "ChildNft"
        }
        }
    ],
    "docs": [
        " Returns the parent token id of the provided child nft."
    ],
    "label": "Nesting::get_parent_of_child",
    "mutates": false,
    "payable": false,
    "returnType": {
        "displayName": [
        "MessageResult"
        ],
        "type": "Option<Id>"
    }
}
```

### Events

### ChildAdded
```json
{
    "args": [
        {
        "docs": [],
        "indexed": true,
        "label": "to",
        "type": {
            "displayName": [
            "ParentId"
            ],
            "type": "Id"
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "collection",
        "type": {
            "displayName": [
            "ChildCollectionAddress"
            ],
            "type": "AccountId"
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "child",
        "type": {
            "displayName": [
            "ChildId"
            ],
            "type": "Id"
        }
        }
    ],
    "docs": [
        " Event emitted when a new child is added."
    ],
    "label": "ChildAdded"
}
```
#### ChildAccepted
```json
{
    "args": [
        {
        "docs": [],
        "indexed": true,
        "label": "parent",
        "type": {
            "displayName": [
            "ParentId"
            ],
            "type": "Id"
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "collection",
        "type": {
            "displayName": [
            "CollectionAddress"
            ],
            "type": "AccountId"
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "child",
        "type": {
            "displayName": [
            "ChildId"
            ],
            "type": "Id"
        }
        }
    ],
    "docs": [
        " Event emitted when a child is accepted."
    ],
    "label": "ChildAccepted"
}
```

### ChildRemoved
```json
{
    "args": [
        {
        "docs": [],
        "indexed": true,
        "label": "parent",
        "type": {
            "displayName": [
            "ParentId"
            ],
            "type": 11
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "child_collection",
        "type": {
            "displayName": [
            "ChildCollection"
            ],
            "type": "AccountId"
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "child_token_id",
        "type": {
            "displayName": [
            "ChildId"
            ],
            "type": "Id"
        }
        }
    ],
    "docs": [
        " Event emitted when a child is removed."
    ],
    "label": "ChildRemoved"
}
```

#### ChildRejected
```json
{
    "args": [
        {
        "docs": [],
        "indexed": true,
        "label": "parent",
        "type": {
            "displayName": [
            "ParentId"
            ],
            "type": 11
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "child_collection",
        "type": {
            "displayName": [
            "ChildCollection"
            ],
            "type": "AccountId"
        }
        },
        {
        "docs": [],
        "indexed": true,
        "label": "child_token_id",
        "type": {
            "displayName": [
            "ChildId"
            ],
            "type": "Id"
        }
        }
    ],
    "docs": [
        " Event emitted when a child is rejected."
    ],
    "label": "ChildRejected"
}
```
### Types
```rust
// AccountId is a 32 bytes Array, like in Substrate-based blockchains.
type AccountId = [u8; 32];
```
```rust
// Id is a 128 bit unsigned integer
type Id = u128;
```
```rust
// ChildNft is a tuple of Child's contract address and child's Id
type ChildNft = (AccountId, Id)
```

### Errors
The suggested methods revert the transaction and return a [SCALE-encoded](https://github.com/paritytech/parity-scale-codec) `Result` type with one of the following `Error` enum variants:

```rust
enum NestingError {
    AlreadyAddedChild,
    AddingPendingChild,
    InvalidParentId,
    ChildNotFound,
    NotTokenOwner,
}
```

## Tests

Check reference implementation

## Copyright

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
