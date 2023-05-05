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

#### Nesting::add_child
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
            "type": "u128"
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
        " Add a child NFT (from different collection) to the NFT in this collection.",
        " The status of the added child is `Pending` if caller is not owner of child NFT",
        " The status of the added child is `Accepted` if caller is is owner of child NFT",
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
        " Ownership of child NFT will be transferred to this contract (cross contract call)",
        " On success emits `ChildAdded`",
        " On success emits `ChildAccepted` - only if caller is already the owner of the child NFT"
    ],
    "label": "Nesting::add_child",
    "mutates": true,
    "payable": false,
    "returnType": {
        "displayName": [
        "ink",
        "MessageResult"
        ],
        "type": "Result"
    }
}
```

#### Nesting::remove_child
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
            "type": "u128"
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
        " Remove a child NFT (from different collection) from parent_token_id in this collection.",
        " The status of added child is `Pending` if caller is not owner of child NFT",
        " The status of added child is `Accepted` if caller is owner of child NFT",
        "",
        " # Requirements:",
        " * The status of the child is `Accepted`",
        "",
        " # Arguments:",
        " * `parent_token_id`: is the tokenId of the parent NFT.",
        " * `child_nft`: (collection_id, token_id) of the child instance.",
        "",
        " # Result:",
        " Ownership of child NFT will be transferred to parent NFT owner (cross contract call)",
        " On success emits `ChildRemoved`"
    ],
    "label": "Nesting::remove_child",
    "mutates": true,
    "payable": false,
    "returnType": {
        "displayName": [
        "ink",
        "MessageResult"
        ],
        "type": "Result"
    }
}
```
### Events

### Types
```rust
// AccountId is a 32 bytes Array, like in Substrate-based blockchains.
type AccountId = [u8; 32];
```
```rust
// ChildNft is a tuple of Child's contract address and child's Id
type ChildNft = (AccountId, u128)
```

### Errors
The suggested methods revert the transaction and return a [SCALE-encoded](https://github.com/paritytech/parity-scale-codec) `Result` type with one of the following `Error` enum variants:

```rust
enum NestingError {
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

enum PSP34ReceiverError {
    /// Returned if transfer is rejected.
    TransferRejected(String),
}
```

## Tests

If applicable, please include a list of potential test cases to validate an implementation.

## Copyright

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
