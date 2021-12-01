# Paratoken Standard

- **PSP Number:** 32
- **Authors:** Witek Radomski <witek@enjin.io>, Lohann Paterno Coutinho Ferreira <lohann@enjin.io>, Francisco Miguel García <miguel@enjin.io>, Patrick O'Dacre <patrick@enjin.io>, Jay Pavlina <jay@enjin.io>
- **Status:** Draft
- **Created:** 2021-12-01
- **Reference Implementation** TBD

## Summary

The Paratoken standard is a unified interface that can represent any number of Fungible and Non-fungible (NFT) assets.
Each ID may represent a new configurable asset type, which may have its own metadata, supply, and other attributes.

## Motivation

Existing Substrate pallets have a lot of fragmentation, upcoming specifications, and custom third-party implementations
for dealing with cross-chain tokens, assets, and non-fungible tokens (NFTs). Wallets and applications need to be able
to index, display, and interact with tokens consistently.

With Polkadot's goal of becoming an interoperable, cross-blockchain ecosystem, the entire community needs a flexible
standard that is designed to anticipate cross-chain minting, transfers, and teleportation -- across Polkadot & Kusama
parachains, parathreads, smart contracts and external blockchains.

## Implementations

[Efinity](https://github.com/enjin/efinity-dev/tree/master/pallets/multi-assets) - Multi-Asset Pallet (coming soon)

## Setup

- Enjin-SOWG Room: `#enjin-sowg:matrix.org`

## Specification

- [Overview](#overview)
- [Asset Identification](#asset-identification)
- [Paratoken Structure](#paratoken-structure)
    - [Base Interface](#base-interface)
    - [Common Types](#base-interface)
    - [Extension Detection](#extension-detection)
        - [Registered Extensions](#registered-extensions)
- [Policies](#policies)
    - [Minting Policy](#minting-policy)
    - [Operator Policy](#operator-policy)
    - [Transfer Policy](#transfer-policy)
    - [Mutability Policy](#mutability-policy)
- [Metadata](#metadata)
    - [On-Chain Metadata](#on-chain-metadata)
    - [Off-Chain Metadata](#off-chain-metadata)
- [Events](#events)
- [Extensions](#extensions)
  - [Extension: Batch](#extension-batch)
  - [Extension: Chunk](#extension-chunks)
  - [Extension: Enumerable](#extension-enumerable)
  - [Extension: Operator](#extension-operator)
- [Source of Truth](#source-of-truth)
- [Interactions](#interactions)
    - [Wallet Interactions](#wallet-interactions)
    - [Explorer Interactions](#explorer-interactions)


## Overview

The Paratoken standard should support both fungible and non-fungible assets with minimal friction.
If established early, we can avoid the mess with multiple asset standards and the difficulty of
implementing them.

## Asset Identification

The current Asset ID implementation in the assets pallet identifies that asset within the chain. In
order to identify assets across all parachains and bridges, a different pattern is necessary.

The paratoken standard enforces the use of [`Universally Unique Asset Identifier`](uuaid/uuaid.md) as an
identifier of the asset on its API.

## Paratoken Structure

The Paratoken standard provides common traits to be implemented by developers, and will use a
custom, native implementation for overall performance.

### Base Interface

Inter-pallet interface between modules (if someone wants to use the standard in their runtime)

```Rust
pub trait Paratoken {

	// Extension support
	// ===================================

	/// Returns the version of the given `extension` if it is supported by `asset`.
	/// It will return `None` if `asset` does not exist or the `extension` is not supported. 
	fn extension_version(asset: AssetId, extension: ExtensionId) -> Option<ExtensionVersion>;



	// Asset creation and Management
	// ===================================

	/// Creates a new asset using the given `policy` where `origin` will be the 
	/// owner. 
	fn create_asset( owner: Origin, policy: MonetaryPolicy);

	/// Transfer the ownership of `asset` from `origin` to `target`.
	fn transfer_ownership( owner: Origin, new_owner: T::Account, asset: T::AssetId);



	// Mint and burn
	// ===================================

	/// Mints new `amount` of `asset` and transfer to `owner` account.
	fn mint( owner: Origin, asset: T::AssetId, amount: T::AssetBalance);

	/// Destroies `amount` from `token` of `asset`. 
	fn burn( owner: Origin, asset: T::AssetId, token: T::TokenId, amount: T::AssetBalance); 



	// Balances and transfers 
	// ===================================

	/// Transfers `amount` of `token` from `asset` from `from` account to `target` account.
	fn transfer(
		from: Origin, 
		to: T::Account, 
		asset: T::AssetId, 
		token: T::TokenId, 
		amount: T::AssetBalance);

	/// Fetchs the balance of `tokens` of `asset` for `owner` account. 
	fn balance(owner: Account, asset: T::Account, token: T::TokenId) -> T::AssetBalance;

	/// Returns the current total supply of the `asset`.
	fn total_supply(asset: T::AssetId) -> T::AssetBalance;



	// Attributes on assets and tokens
	// ===================================

	/// Returns the asset attribute of `key` for the given `asset`.
	fn asset_attribute( asset: T::AssetId, key: T::AttributeKey) -> Option<T::AttributeValue>;

	/// Updates the asset attribute `key` to `value` for the given `asset`.
	fn set_asset_attribute(
		owner: Origin, 
		asset: T::AssetId, 
		key: T::AttributeKey,
		value: T::AttributeValue);

	/// Removes the `key` attribute from the given `asset`.
	fn clear_asset_attribute( owner: Origin, asset: T::AssetId, key: T::AttributeKey);

	/// Returns the token attribute of `key` for  `token` of `asset` .
	fn token_attribute( 
		asset: T::AssetId, 
		token: T::TokenId, 
		key: T::AttributeKey
	) -> Option<T::AttributeValue>;

	/// Updates attribute `key` of the `token` of `asset` using the given `value`.
	fn set_token_attribute(
		owner: Origin,
		asset: T::AssetId,
		token: T::TokenId,
		key: T::AttributeKey,
		value: T::AttributeValue);

	/// Removes attribute `key` of the `token` of `asset`.
	fn clear_token_attribute(
		owner: Origin, 
		asset: T::AssetId,
		token: T::TokenId,
		key: T::AttributeKey);
}
```

### Common Types

```Rust
/// Extension ID.
pub type ExtensionId = u16;

/// Bounded-sized for attribute keys.
pub type AttributeKey = BoundedVec<u8, KeyLimit>;

/// Bounded-sized for attribute values.
pub type AttributeValue = BoundedVec<u8, ValueLimit>;
```

### Extension Detection

This standard **can be extended** with future proposals or optional capabilities. They _are optional_,
so some networks may not implement them.

The **extension_version** function is required to check if a given asset (in its current
network) supports specific capabilities.

The `ExtensionId` type implementation **requires** the following statements:
- It should be **flexible enough to accept new future values** without a serialization error. It
  means that new extension IDs can be used to query legacy assets and no error should be raised. In
  that case, `extension_version` will return `None`.
- It should be enough to **store up to 2^16 extension identifiers**.

On `ETH`, [ERC-165](https://eips.ethereum.org/EIPS/eip-165) could be used to support this detection.

See [`Paratoken::extension_version(..)`](#base-interface).

```Rust

/// Defines a [Semantic Versioning](https://semver.org/)
pub struct ExtensionVersion {
	/// The **major version** is incremented when an incompatible API change is done.
	pub major: u64,
	/// The **minor version** is incremented when a new functionality in a backwards compatible 
	/// manner is added.
	pub minor: u64,
	/// The **patch version** is incremented when a backwards compatible bug is fixed.
	pub patch: u64,
}`
```

#### Registered Extensions

The following table contains the extensions already registered by this standard.

| Ext. ID | Version |  Extension |
| ------: | :-----: | :--------- |
| 0x0001  | 1.0.0   | [Batch Extension](extensions/batch.md) |
| 0x0002  | 1.0.0   | [Chunks Extension](extensions/chunks.md) |
| 0x0003  | 1.0.0   | [Enumerable Extension](extensions/enumerable.md) |
| 0x0004  | 0.1.0   | **TBD** Teleportation |
| 0x0005  | 1.0.0   | [Operator](extensions/operator.md) |
| 0x0006  |         | **TBD** HTLC support |


## Policies

Polices define what is the behaviour of the asset for specific operations.

### Minting Policy

Minting policies determines who can mint new tokens:

```Rust
enum MintingPolicy {
	/// Minting is not allowed.
	NotAllowed,

	/// Only this issuer can mint tokens.
	Issuer(AccountId),

	/// Only this issuer can mint tokens but just once.
	/// That ensures that no more tokens will be created after the first minting.
	OneTimeIssuer(AccountId),

	// Network Extensions
	// ---- 
	/// Customized or optimized  
	NetworkCustom(NetworkId, PolicyId),
}

pub trait MintableAsset {
	/// Returns the minting policy of the given asset.
	///
	/// If `asset` is knonwn, it will return `None`. 
	fn minting_policy(asset: T::AssetId) -> Option<MintingPolicy>;
}
```

Please keep in mind `AccountId` can be **a multi-signature account with threshold N of M** (if
the underlying network supports it), so minting operation could be delegated on a secure group of
accounts. That case covers use cases like bridges of wrapped elements, like _Merchants_ in _WBTC_.

If issuer is a **smart contract**, the minting could be more flexible like _MakerDAO_ and its _DAI_
token.

The policy `OneTimeIssuer(..)` ensures that the token has no inflation because the
administrator/operator cannot mint more tokens later. After the first minting operation, that policy
becomes [_immutable_](#mutable-policy).

The **Network extensions** is a mechanism allows network customizations or optimizations on
specialized parachains. Some parachains could create complex native policies that will be cheaper
than smart-contract ones.

### Transfer Policy

**NOTE @miguel:** Review ERC-1400 and security token transfer restrictions. It will need a customized
field too.


### Mutability Policy

The mutability of a policy indicates if other policies could change in the future. Once a policy has
been set as immutable, no one can revert it.  The aim of this policy is the improvement of
transparency on the asset. For instance, an immutable `NotAllowed` minting policy ensures that the
token won't support any inflation.

```Rust
	struct Mutability {
		Minting,
		Teleportation,
	}
```

```Rust
	/// Returns `true` if the `policy` of asset `id` is immutable.
	fn is_policy_immutable( id: T::AssetId, policy: Mutability) -> bool; 

	/// Sets `policy` of `id` asset as immutable. 
	fn set_policy_immutable( id: T::AssetId, policy: Mutability);

	/// Batch  
	fn set_policies_immutable( id: T::AssetId, policies: Vec<Mutability>);
```


## Metadata

The Paratoken metadata is designed to minimize the amount of storage usage, so the metadata is separated in two groups:

### On-Chain Metadata
You should only store consensus critical data here, this kind of metadata will be stored forever in the blockchain and
synchronized across all nodes, as a result its usage must be high bandwidth and storage cost. A good way to verify if
the data must be stored on-chain is to check if it is accessed at any point in your runtime logic. If it is not it
probably should be stored off-chain.

On-chain metadata is stored in a key-value fashion, the both the key and value are a byte array.

Characteristics:
- DepositBase: A base fee regardless of the size of metadata. This parameter was primarily introduced to prevent Distributed-Denial-of-Service (DDoS) attacks. `DepositBase` makes such attacks prohibitively expensive, and eliminates the possibility of an attacker generating millions of small attributes to flood and crash the system.
- DepositPerByte: Reflects the dependence of the cost on the size of the metadata. The larger the data, the more resources are needed to store and process it.
- Formula: Metadata fees are constructed around two constants (MetadataDepositBase and MetadataDepositPerByte). The formula for calculating the metadata store fees is: `MetadataDepositBase + MetadataDepositPerByte * (key.len() + value.len())`
- Limited: The maximum length of data stored on-chain per token/asset is limited by `KeyLimit`, `ValueLimit` and `AttributeLimit` properties.
- Validatable: You can define some contraints on-chain to validate the data stored on-chain.

### Off-Chain Metadata

For store any data that you don't need on-chain, such as images, names, description, etc.

Characteristics:
- Cheap: Can be stored in (URI, IPFS, etc).
- on-chain reference: Must have a on-chain reference stored at `:metadata` token attribute key.
- Not validatable: Not readable on-chain, consequently you can't perform any validation or logic using this data.

There are currently no plans in place to restrict the metadata payload format.

## Events

```Rust

pub enum Event<T: Config> {
	/// A new asset was created by `owner`.
	/// \[ asset_id, owner \]
	AssetCreated(T::AssetId, T::AccountId),

	/// The given asset was destroyed.
	/// \[ asset_id]
	AssetDestroyed(T::AssetId),

	/// The `amount` balance of the `asset` was minted by owner.
	/// \[owner, asset, amount\]
	Minted(T::AccountId, T::AssetId, T::AssetBalance),

	/// New tokens were minted on asset. 
	/// \[owner, asset, from_token, to_token \]
	TokensMinted(T::AccountId, T::AssetId, T::TokenId, T::TokenId),

	/// `TransferredSingle` MUST emit when tokens are transferred.
	/// The `from` argument MUST be the address of the holder whose balance is decreased.
	/// The `to` argument MUST be the address of the recipient whose balance is increased.
	/// The `asset_id` argument MUST be the token's asset type.
	/// The `token_id` argument MUST be the token being transferred.
	/// The `amount` argument MUST be the number of tokens by which the holder balance is decreased 
	/// and the recipient balance is increased.
	/// \[from, to, asset_id, token_id, amount\]
	TransferredSingle(T::AccountId, T::AccountId, T::AssetId, T::TokenId, T::Balance),

	/// `AssetAttributeSet` MUST emit when asset attribute is set.
	/// \[asset_id, attribute key, attribute value\]
	AssetAttributeSet(T::AssetId, T::AttributeKey, T::AttributeValue),

	/// `AssetAttributeCleared` MUST emite when the asset attribute is cleared.
	/// \[asset_id, key\]
	AssetAttributeCleared(T::AssetId, T::AttributeKey),

	/// `TokenAttributeSet` MUST emit when the token attribute is set.
	/// \[asset_id, token_id, key, value\]
	TokenAttributeSet(T::AssetId, T::TokenId, T::AttributeKey, T::AttributeValue),

	/// `TokenAttributeCleared` MUST emit when the token attribute is cleared.
	/// \[asset_id, token_id, key\]
	TokenAttributeCleared(T::AssetId, T::TokenId, T::AttributeKey),
}
```

## Extension: Batch

The batch extension adds interfaces to execute some operations in **batch mode**.
These functions may represent an improvement over the individual execution of their individual
operations.

The bach parameter is always bounded by a `BatchLimit` network parameter.


### Execution Batch Policy

Any batch function that modifies the state, can select its execution policy. That policy defines how
to deal with errors on individual elements of the batch. There are two types:

- `ExecutionBatchPolicy::Transactional`, where all items in the batch produce a success execution or
  no update is done to the storage. That means that any error in a single element of the batch will
  revert all previous changes applied to the storage.
- `ExecutionBatchPolicy::BestEfford`, any error on single item might raise an event up, but it does
  not mean a short-circuit of the batch operation.

### Interfaces and Traits

The batch extension defines the following interface:

```Rust
pub trait BatchExtension {
	/// Returns the maximum number of assets allowed per transaction.
	fn batch_asset_limit() -> u32 ;

	/// Returns the maximum number of tokens per asset allowed per transaction.
	fn batch_token_per_asset_limit() -> u32;

	/// Creates a batch of assets using the information from `details` and using the given
	`execution_policy`. 
	fn batch_create_asset( 
		owner: Origin, 
		details: BatchCreateAssetDetails, 
		execution_policy: ExecutionBatchPolicy); 

	/// Transfers the specific amount of tokens from `recipients` of `asset` from `source`
	/// account. 
	/// See `Paratoken::transfer`.
	fn batch_transfer(
		source: Origin, 
		recipients: AssetRecipientsOf<T>, 
		execution_policy: ExecutionBatchPolicy); 



	/// Returns the balance of `tokens` for the given `asset` and `owner` account. 
	fn batch_token_balance(
		owner: Account, 
		asset: T::AssetId, 
		tokens: BoundedVec<TokenId, BatchTokenPerAssetLimit>
	) -> Result< Vec<Balance>, ErrorType>;

	/// Returns the balances of `assets` for the given `owner` account.
	fn batch_asset_balance( 
		owner: Account, 
		assets: BoundedVec<T::AssetId, BatchAssetLimit> 
	) -> Result< Vec<Balance>, ErrorType>;
}

```

### Types

```Rust
/// Defines how the bach function tackles errors on individual elements. 
pub ExecutionBatchPolicy {
	/// Any error on an individual item, will revert any success operation in previous items.
	/// In that case, the batch execution will be cancelled inmediatly.
	Transactional,
	/// Errors on individual items could be registered as events, and they do not mean a
	/// short-circuit on the execution of the batch function.
	BestEfford, 
}

pub struct Recipient {
	to: T::AccountId,
	token: T::TokenId,
	amount: T::AssetBalance,
}

/// Bounded list of recipients.
pub type RecipientsOf<T> = BoundedVec<RecipientOf<T>, <T as Config>::BatchTokenPerAssetLimit>;

pub struct AssetRecipient {
	asset: T::AssetId,
	recipients: RecipientsOf<T>,
}

pub type AssetRecipientsOf<T> = BoundedVec<AssetRecipient, BatchAssetLimit>;

/// Details for batch asset creation.
pub struct BatchCreateAssetDetails {
	pub policy: MonetaryPolicy,
	pub asset_count: AssetCounter,
}

```

### Events

```Rust
pub enum Event<T: Config> {
	/// `TransferredBatch` MUST emit when tokens are transferred in batch mode.
	/// The `from` argument MUST be the address of the holder whose balance is decreased.
	/// The `asset_ids` argument MUST be the list of the token's asset types.
	/// The `to` argument MUST be the address of the recipient whose balance is increased.
	/// The `token_ids` argument MUST be the list of tokens being transferred.
	/// The `values` argument MUST be the list of number of assets / tokens (matching the list and 
	/// order of assets and tokens specified in _ids) by which the holder balance is decreased and
	/// the recipient balance is increased. When minting/creating tokens, the `from` argument
	/// MUST be the address of the Account authorized to mind / create the token.  When 
	/// burning/destroying tokens, the `to` argument MUST be set to `0x0` (i.e. zero address).  
	/// \[from, to, asset_ids, token_ids, amounts\]
	TransferredBatch(T::AccountId, T::AssetId, Vec<T::AccountId, Vec<T::TokenId>, T::Balance>),
}
```

### Errors
**TBD**: Error definition.

## Extension: Chunks

An optimization where non-fungible assets store their tokens in chunks in storage.
If tokens are created sequentially, those chunks can be compressed using ranges in order to
improve the efficiency of the storage and reduces the size of messages.


### Chunk

A chunk is just a bounded group of tokens. This group can contain up to
`ChunkExtension::token_chunk_size()` elements. As an example, let's assume we have an account
which owns the following tokens of a given asset:

```
	tokens_of_account_for_asset = [0, 1, 2, 3, 4, 5, 6, 7];
```

And the network implements a `T::TokenChunkSize == 5`. In that case, the tokens will be
grouped and saved into two chunks:

```
	chunk_0 = [0, 1, 2, 3, 4];
	chunk_1 = [5, 6, 7];
```

The **chunk index** identifies the chunk on the storage, and it can be calculated deterministically
from the token ID and the network token chunk size parameter.

```
	chunk_index = token_id / T::TokenChunkSize;
```

### Range compression

If tokens are created sequentially, chunks can be compressed using open ranges. Following the
above example, same chunks will be represented as:

```
	chunk_0 = [0,5);
	chunk_1 = [5,8);
```

### Interfaces and Traits

```Rust
pub type TokenChunkOf<T> = Chunk<<T as Config>::Token, <T as Config>::TokenChunkSize>;
pub type TokenChunkListOf<T> = BoundedVec<TokenChunkOf<T>, <T as Config>::BatchLimit>;

pub trait ChunkExtension {
	/// Returns the `T::TokenChunkSize`.
	fn token_chunk_size() -> u32;

	/// Optimized `burn` of `tokens` in the same chunk.
	///
	/// See `Paratoken::burn()`.
	fn burn_by_chunk( owner: Origin, asset: T::AssetId, tokens: TokenChunkOf<T>);

	/// Optimized version of `batch_burn` by chunks.
	///
	/// See `BatchExtension::batch_burn()`.
	fn burn_by_chunks( owner: Origin, asset: T::AssetId, chunks: TokenChunkListOf<T>);

	/// Optimized version of `transfer` where `tokens` belong to the same `chunk`.
	///
	/// See `Paratoken::transfer`.
	fn transfer_by_chunk( owner: Origin, asset: T::AssetId, tokens: TokenChunkOf<T>);

	/// Optimized version of `batch_transfer` by chunks.
	///
	/// See `BatchExtension::batch_transfer`.
	fn transfer_by_chunks( owner: Origin, asset: T::AssetId, chunks: TokenChunkListOf<T>);
}
```

### Types

**TBD**

### Events

The `burn_by_*` functions MUST emit `Event::Burned` or `Event::TokenBurned`.
The `transfer_by_*` functions MUST emit `Event::TransferredSingle` if the transfered chunk only
contains one token, and MUST emit `Event::TransferredBatch` when the chunk contains more than one
token.

### Errors

**TBD**

## Extension: Enumerable

The enumerable extension supports general use-cases to list assets and tokens owned by an account.
Wallets or harvesting services would use this extension to gather that information.
All functions of this interface are read-only, so we can expose them in the RPC or WebSocket interface.

### Interface

```Rust

trait EnumerableParatoken <Account, AssetId, TokenId, Size> {
	/// Defines the page size for assets.
	type AssetPageSize = Get<Size>;
	/// Defines the page size for tokens.
	type TokenPageSize = Get<Size>;



	/// Returns the number of tokens associated to `asset`.
	fn total_token_count(asset: AssetId) -> Size;

	/// Returns the number of assets owned by `account`.
	fn asset_count_of(account: Account) -> Size;

	/// Returns the number of tokens of `asset` owned by `account`. 
	fn token_count_of(account: Account, asset: AssetId) -> usize;



	/// Returns the given `page_index` page of the list of assets owned by `account`.
	/// It provides a way to access secuentially to the unbounded list of assets owned by some
	/// account. 
	/// It returns `None` if `page_index` is greater than the last valid index for that account.
	///
	/// # Params
	/// - `page_index`, It MUST start at `0`.
	fn get_assets_of(account: AccountId, page_index: u32) -> Option<AssetPage>;

	/// Returns the given page of the list of tokens of `asset` owned by `account`.
	/// It provides a way to access secuentially to the unbounded list of tokens of the given asset
	/// owned by some account. 
	/// It returns `None` if `page_index` is greater than the last valid index for that account.
	///
	/// # Params
	/// - `page_index`, It MUST start at `0`.
	fn get_tokens_of(account: AccountId, asset: AssetId, page_index: u32) -> Option<TokenPage>;
}
```

### Types

```Rust

pub type AssetPageOf<T> = Page<<T as Config>::AssetId, <T as Config>::AssetPageSize>;
pub type TokenPageOf<T> = Page<<T as Config>::TokenId, <T as Config>::TokenPageSize>;

/// Represents a page of an unbounded list of elements.
pub trait Page<T, S>
{
	/// Access to the elements of this page.
	pub fn items(&self) -> &[T];

	/// Page number of self.
	pub fn page_number(&self) -> S;
}
```

## Extension: Operator

The operator extension allows an account to make transfers on behalf of the owner of the token.

### Interface

```Rust
/// Allows a third party `operator` account to manage an `owner`'s tokens
/// All the approval operations use the `expiration` parameter to enable, limit or disable completely:
/// - if `expiration == None` then it **enables the approval** of `operator` **without any time
/// limit**.
/// - if `expiration == Some(future_block_id)`, the approval will be **valid until**
/// `future_block_id` is reached.
/// - if `expiration == Some(old_block_id)`, it **disables** any previous approval.
pub trait OperatorExtension<AccountId, AssetId, Balance, TokenId, Time: PartialOrd> {
  /// Returned if there is an error
  type Error;

  /// The type used for expiration
  type Expiration: Expirable<Time>;

  /// Approve `operator` to manage all the `owner`'s assets.
  fn approve_for_all(
    owner: AccountId,
    operator: AccountId,
    expiration: Self::Expiration,
  ) -> Result<(), Self::Error>;

  /// Unapprove `operator` to manage `owner's` assets
  fn unapprove_for_all(owner: AccountId, operator: AccountId);

  /// Approve the `operator` to manage all of `owner`'s tokens belonging to `asset`
  fn approve_asset(
    owner: AccountId,
    operator: AccountId,
    asset: AssetId,
    expiration: Self::Expiration,
  ) -> Result<(), Self::Error>;

  /// Unapprove the `operator` to manage all of `owner`'s tokens belonging to `asset`
  fn unapprove_asset(owner: AccountId, operator: AccountId, asset: AssetId);

  /// Approve the `operator` to transfer up to `amount` of `owner`'s `token`s
  fn approve_token(
    owner: AccountId,
    operator: AccountId,
    asset: AssetId,
    token: TokenId,
    amount: Balance,
    expiration: Self::Expiration,
  ) -> Result<(), Self::Error>;

  /// Unapprove `operator` to transfer `owner`'s `token`s
  fn unapprove_token(owner: AccountId, operator: AccountId, asset: AssetId, token: TokenId);

  /// Transfers `amount` of `tokens` from account `from` to address `to` if `operator` has enough allowance
  fn transfer_from(
    operator: AccountId,
    from: AccountId,
    to: AccountId,
    asset: AssetId,
    token: TokenId,
    amount: Balance,
  ) -> Result<(), Self::Error>;

  /// Returns the amount which `operator` is still allowed to withdraw from `owner` for the given
  /// `token` of `asset`.
  ///
  /// If the approval was enabled by `set_approval_for_all` or `set_asset_approval`, the
  /// returned value will be the maximum value of the `Balance` type.
  fn allowance(
    owner: &AccountId,
    operator: &AccountId,
    asset: Option<&AssetId>,
    token: Option<&TokenId>,
  ) -> Balance;
}

/// Can expire
pub trait Expirable<T: PartialOrd> {
  /// If it is expired at `time`
  fn is_expired_at(&self, time: T) -> bool;
}
```


# TODOs

## Open questions
- Do we need batch support for tokens attributes?
- Do we need deposits over attributes?

### Source of Truth

Facilitating Asset movement between Parachains and Disparate Chains like Ethereum is central to the Paratoken Standard.
Establishing how the Source of Truth for Assets will be maintained, then, is essential.

Some important Asset properties that require careful consideration:
* minting / burning operations
* balance
* metadata
* ownership history

One important rule-of-thumb is that Asset state must never fall out of sync as the Asset moves. There must be
guarantees that ensure state will be maintained synchronously. Cross-Chain Messaging by themselves can never guarantee
this. But Polkadot SPREE promises the infrastructure required:

> Finally, SPREE, also known as Secure Protected Runtime Execution Enclaves, is a feature allowing the same piece of
> logic to be securely and homogeneously sharded across all parachains. This allows parachains which do not trust each
> other’s business logic or governance apparatus to nonetheless interact. Moving tokens between stranger-chains,
> maintaining the properties of NFTs as they migrate between chains and even sharing smart-contracts can all be
> achieved using this functionality.
> - https://medium.com/polkadot-network/the-launch-of-parachains-78188fcf024f

While XCMP guarantees the delivery of a message, it does not guarantee what code will be executed on a target parachain. When moving an NFT from Chain A to Chain B, you must _know_ that the NFT will be "removed" from Chain A and "added" to Chain B, and when moving n FTs from Chain A to Chain B, you must _know_ that the FT balance will be reduced by n on Chain A and increased by n on Chain B. This guarantee is required for the trustless migration of Assets. Shared Protected Runtime Execution Enclaves (SPREE) Modules are fragments of shared WebAssembly logic uploaded to the Relay Chain that make trustless transfer of Assets across parachains possible; like smart contracts, they guarantee the execution of code.

As Parity is still in the design phase of SPREE we can say little about possible API now. We will continue to update this section as more information becomes available.

Other Resources:
* https://wiki.polkadot.network/docs/en/learn-spree
* https://w3f-research.readthedocs.io/en/latest/polkadot/XCMP/index.html#xcmp-and-spree

### Interactions

For easy integration with existing wallets and explorers, the Paratoken will retain API familiar in the ERC20, ERC721, and ERC1155.  To that end, [ERC165](https://eips.ethereum.org/EIPS/eip-165) will be implemented to reflect this compatibility.

Both wallets and explorers will need to be able to parse the Global Asset ID to identify the parachain and the asset.

#### Wallet Interactions

Some common interactions:

- Sign transactions
    - Types: transfer, burn, stake, place bids
    - Where do the transactions get sent to?
- Read balance
    - Where is the source of truth?
- Read icon, symbol, name, value, and ID for FTs
- Read image, name, ID, description for NFTs
- Lookup other tokens from the same creator or holder
- Lookup other tokens of the same type/kind
- Lookup transaction history


#### Explorer Interactions

Support existing frame-bases APIs at https://docs.api.subscan.io/:

- General API
- Staking API
- Price API
- Governance API
- Runtime API
- Parachain API

#### Operator Extension Review 
- Review attack vector on `approve` [described here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/) and [discussed here](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729)

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
