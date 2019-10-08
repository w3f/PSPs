# Fungible Token Standard

* **PSP Number:** [To be assigned]
* **Authors:** xlc
* **Status:** Draft
* **Created:** 2019-10-08
* **Reference Implementation** [To be implemented]

## Summary

A fungible token standard for Substrate-based blockchains.

## References

- https://hackmd.io/gQKQGf42TeOODid3hM4_1w
- https://eips.ethereum.org/EIPS/eip-20
- https://eips.ethereum.org/EIPS/eip-777
- https://eips.ethereum.org/EIPS/eip-1155

## Motivation

A cross-chain token standard is required for blockchains to transfer assets across the ecosystem and for clients (e.g. wallet, browser extension) to offer a unified support of all the tokens provided by different chains.

## Specification

A fungible token is mainly made on two parts: Descriptions and Operations.

### Token Descriptions

Token descriptions offers a universal way to describe, display the details about a token. This information may not be stored on-chain to reduce usage of on-chain storages. A centralized or decentralized token registry may be used to discovery the token description, which is out of the scope here.

### Token Operations

A Token implementation should provide set of operations to achieve common functionalities such as transfer and query balances.

Write operations can be dispatched via extrinsic (i.e. transaction), smart contract call (on a chain with contracts module), or runtime module call (via public Rust methods). Query operations can be dispatched via RPC (`state_storage`), smart contract call, or runtime module call.

## Specification

### Token Descriptions

With TypeScript syntax:

```typescript
interface Token {
    name: string,
    symbol: string,
    decimals: number,
    uri?: string, // Substrate URI. See psp-uri
    description?: string,
    icon?: string, // HTTP URL or IPFS hash or dataURI
}
```

If the token description is stored off-chain, a file hash could be committed on-chain to ensure the integrality of the data.

### Token Operations

#### Query

The initial query ability will be limited on accessing the storage directly without any additional computation work. This means all the tokens that confirming the standard must have the same underlying storage format. This limitation will be lift on next version, by allowing execute a WASM code in a sandbox to process the storage.

Two storage formats are specified: single token module storage format and multi token module storage format.

Storage metadata will includes additional information such as the key hasher, map kind (`map`, `double_map` or `linked_map`) and later the type information. Before the type information is ready, it should be hardcoded as part of the SDK.

##### Single Token Module

Required:

- `TotalIssuance: Balance`
- `FreeBalance: Balance`

Optional:

- `ReservedBalance: Balance`

##### Multi Token Module

Required:

- `TotalIssuance: map AssetId => Balance`
- `FreeBalance: map (AssetId, AccountId) => Balance`

Optional:

- `ReservedBalance: map (AssetId, AccountId) => Balance`

#### Events

##### Single Token Module

- `Transfer(AccountId, AccountId, Balance, Balance)`
  - Transfer succeeded (from, to, value, fees).

##### Multi Token Module

- `Transfer(AssetId, AccountId, AccountId, Balance, Balance)`
  - Transfer succeeded (asset_id, from, to, value, fees).

#### Transaction

##### Single Token Module

Required:

- `transfer(dest: Account, value: Balance)`
  - Transfer some liquid free balance to another account.

Optional:

- `send(dest: Account, value: Balance, data: Vec<u8>)`
  - Transfer some liquid free balance to another account with additional data. Similar to `send` from ERC777.

##### Multi Token Module

Required:

- `transfer(asset_id: AssetId, dest: Account, value: Balance)`
  - Transfer some liquid free balance to another account.

Optional:

- `send(asset_id: AssetId, dest: Account, value: Balance, data: Vec<u8>)`
  - Transfer some liquid free balance to another account with additional data. Similar to `send` from ERC777.

## Tests

[To be created]

## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).

----

TODOs:
  - Smart Contract interface?
  - Multi Token Module defined based on child storage instead of double map?
  - More events?
  - More transactions? ERC777 style?
  - Incorporate more ideas from ERC1155?

