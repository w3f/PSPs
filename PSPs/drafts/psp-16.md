# SubToken Standard

- **PSP Number:** 16
- **Authors:** @poria-cat
- **Status:** Draft
- **Created:** 2021-05-18
- * **Reference Implementation** https://github.com/Starry-Network/starry_node/tree/master/pallets/pallet-sub
## Summary

This is an application for batch mintable NFTs, where a single NFT can be split into more SubNFTs

## Motivation

Many NFTs are expensive, and if you want more people to have a particular NFT, you can split the NFT so that more people can have it. 

## Specification

/// Stores
/// The set of SubToken creators. subtoken_collection => creator_address
SubTokenCreator get(fn sub_token_creator): map hasher(blake2_128_concat) T::Hash => T::AccountId;
/// Record the collection_id of the SubToken corresponding to the locked NFT subtoken_collection => nft(collection_id, start_idx) 
SubTokens get(fn sub_tokens): map hasher(blake2_128_concat) T::Hash => (T::Hash, u128);

## Tests

[To be created once more feedback is collected]

## Copyright

This document is placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
