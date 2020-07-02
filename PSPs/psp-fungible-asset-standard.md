# Fungible Asset Standard

* **PSP Number:** [To be assigned]
* **Authors:** xlc
* **Status:** Draft
* **Created:** 2019-10-08
* **Reference Implementation** [To be implemented]

## Summary

A fungible asset standard for Substrate-based blockchains.

## References

- https://hackmd.io/gQKQGf42TeOODid3hM4_1w
- https://eips.ethereum.org/EIPS/eip-20
- https://eips.ethereum.org/EIPS/eip-777
- https://eips.ethereum.org/EIPS/eip-1155
- https://github.com/paritytech/xcm-format

## Motivation

A cross-chain asset standard is required for blockchains to transfer assets across the ecosystem and for clients (e.g. wallet, browser extension) to offer a unified support of all the assets provided by different chains.

## Specification

A fungible asset is mainly made on two parts: Descriptions and Operations.

### Asset Descriptions

Asset descriptions offers a universal way to describe, display the details about a asset. This information may not be stored on-chain to reduce usage of on-chain storages. A centralized or decentralized asset registry may be used to discovery the asset description, which is out of the scope here.

### Asset Operations

An Asset implementation should provide set of operations to achieve common functionalities such as transfer and query balances.

Write operations can be dispatched via extrinsic (i.e. transaction), smart contract call (on a chain with contracts module), or runtime module call (via public Rust methods). Query operations can be dispatched via RPC (`state_storage`), smart contract call, or runtime module call.

## Specification

### Asset Descriptions

With TypeScript syntax:

```typescript
interface AssetDescription {
  name: string;
  symbol: string;
  decimals: number;
  uri?: string; // Substrate URI. See psp-uri
  description?: string;
  icon?: string; // HTTP URL or IPFS hash or dataURI
}
```

Asset description can also be embedded in genesis file as additional properties. It is expected to be immutable but maybe changed under speical circumstances (e.g. project rename / rebrand).

### Asset Operations

#### Data Types

With TypeScript syntax:

```typescript
interface AssetOperationDescription {
  types: TypeRegistry;
  queries: {
    [name: string]: {
      parameterTypes: Record<string, TypeWithHahser>;
      returnType: string;
      path: Path;
    };
  };
  operations: {
    [name: string]: {
      parameterTypes: Record<string, TypeName>;
      path: Path;
    };
  };
  events: {
  	[name: string]: {
      types: Record<string, TypeName>;
      path: Path;
    } 
  };
}

interface TypeWithHahser {
  type: TypeName;
  hasher: Hasher;
}
type TypeName = string;
type Hasher = 'Blake2_128' | 'Blake2_256' | 'Blake2_128Concat' | 'Twox128' | 'Twox256' | 'Twox64Concat' | 'Identity';
type Hasher = ''
type Path = string | [ModuleName, MethodName, ...string[]]
```

Asset operation description stores the nessusary information to perform asset operations and might need to be updated with new runtime versions. There should be a way to fetch the corresponding `AssetOperationDescription` . e.g. From a registry or a commonly known location.

#### TypeRegistry

Type registry stores the nessusary type information to interact with the runtime. This is inspired by polkadot.js type registry.

```typescript
interface TypeRegistry {
  [typename: string]: TypeDefination;
}

type TypeDefination = string | // primitives or buildin types
  Record<string, TypeDefination> | // struct
  { _enum: EnumDefination } | // enum
  Array<TypeDefination> // tuple

type EnumDefination = Array<string> | // C like enum
  Record<string, TypeDefination> // Algebraic data types like enum, the order of the key matches to the variant index
```

Currently the type registry needs to be manually maintained and at later stage, https://github.com/paritytech/scale-info/ could be used to automatically generate the type inforamtion.

#### Queries

The initial query ability will be limited on accessing the storage directly without any additional computation work. This means all the assets that confirming the standard must have the same underlying storage format. This limitation will be lift on later version, by allowing execute a WASM code in a sandbox to process the storage.

- totalIssuance: Balance
  - parameters
    - assetId: AssetId
  - description
    - The total amount of issuance in the system for a given asset

- freeBalance: Balance
  - parameters
    - assetId: AssetId
    - who: AccountId
  - description
    - Query the free balance of a given account for a given asset

- totalBalance: Balance
  - parameters
    - assetId: AssetId
    - who: AccountId
  - description
    - Query the combined balance of a given account for a given asset

#### Operations

- transfer
  - parameters
    - assetId: AssetId
    - to: AccountId
    - amount: Balance
  - description
    - Transfer some liquid free balance to another account

#### Events

- Transfer
  - parameters
    - from: AccountId
    - to: AccountId
    - amount: Balance
  - description
    - Transfer succeeded

## Examples

[To be created]

## Tests

[To be created]

## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
