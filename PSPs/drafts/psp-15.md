# SubToken Standard

- **PSP Number:** 15
- **Authors:** @poria-cat
- **Status:** Draft
- **Created:** 2021-05-18

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

pub trait SubTokens {
    /// Lock NFT to this pallet and create a new collection.
    /// 
    /// Parameters:
    /// - `collection_id`: The collection in which NFT is located.
    /// - `start_idx`: NFT's Index
    /// - `is_fungible`: SubToken is FT or not.
    fn create(collection_id: T::Hash, start_idx: u128, is_fungible: bool) -> DispatchResult;

    /// Mint one or a batch of SubNFTs.
    ///
    /// Parameters:
    /// - `receiver`: The address that accepts minted tokens.
    /// - `sub_token_collection_id`: The collection where the minted SubNFTs is located
    /// - `uri`: Uri representing the detailed information of SubNFT.
    /// - `amount`: How many tokens to mint.
    fn mint_non_fungible(
        receiver: T::AccountId,
        sub_token_collection_id: T::Hash,
        uri: Vec<u8>,
        amount: u128
    ) -> DispatchResult;

    /// Mint some SubFTs
    ///
    /// Parameters:
    /// - `receiver`: The address that accepts minted tokens.
    /// - `sub_token_collection_id`: The collection where the minted FTs is located
    /// - `amount`: How many tokens to mint.
    fn mint_fungible(
        receiver: T::AccountId,
        sub_token_collection_id: T::Hash,
        amount: u128
    ) -> DispatchResult;

    /// Burn all SubTokens and restore to NFT.
    ///
    /// Parameters:
    /// - `sub_token_collection_id`: The collection where subtokens are located.
    fn recover(sub_token_collection_id: T::Hash) -> DispatchResult;

}

## Tests

[To be created once more feedback is collected]

## Copyright

This document is placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
