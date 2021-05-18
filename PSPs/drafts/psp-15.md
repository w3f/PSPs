# Batch Mint/Transfer NFT Standard

- **PSP Number:** 15
- **Authors:** @poria-cat
- **Status:** Draft
- **Created:** 2021-05-18

## Summary

This is a method that allows for batch minting or transfer of NFTs.

## Motivation

Many times it is necessary to mint many NFTs at the same time, if using the cyclic method will cause waste of resources, so this standard applies to different structures to achieve batch minting/transfer

## Specification

/// The set of minted NFTs. (collection_id, start_idx) => nft_info
Tokens get(fn tokens): double_map hasher(blake2_128_concat) T::Hash, hasher(blake2_128_concat) u128 => TokenInfo<T::AccountId>;

/// Use start_idx and end_idx to represent a continuous id,
/// if end_idx is equal to start_idx, then this is a single NFT, 
/// if end_idx is greater than start_idx, it is a batch-minted NFT
pub struct TokenInfo<AccountId> {
    pub end_idx: u128,
    pub owner: AccountId,
    pub uri: Vec<u8>,
}

## Tests

[To be created once more feedback is collected]

## Copyright

This document is placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
