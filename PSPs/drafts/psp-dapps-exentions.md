# DApp Extension API

- **PSP Number:** [To be assigned (=number of the initial PR to the PSPs repo)]
- **Authors:** Fabio Lama <fabio.lama@pm.me>
- **Status:** Draft
- **Created:** [2022-08-30]
- **Reference Implementation** https://github.com/polkadot-js/extension/tree/master/packages/extension-dapp

## Summary

...

## Motivation

...

## Specification

...

### Communication

Extensions inject a specific structure into the `injectedWeb3` field in the main
`window` object of the DOM. Dapps can then interspect that field, search for the
desired extension and then interact with the extension by calling the defined
functions.

`window.injectedWeb` is of type `Record<string, InjectedWindowProvider>`

where

```type
export interface InjectedWindowProvider {
    enable: (origin: string) => Promise<Injected>;
    version: string;
}
```


The datastucture contains (meta)data about the extension and offers some
functions such as the ability to retrieve accounts and sign messages. The
extension itself does not create any sort of transactions and does not submit
anything to any network, the dapps is responsible for that. Ultimately, the
extension primarily manages accounts and creates signatures of messages.

### Types

#### Extension

This structure is the primary way to interact with the Extension. 

```typescript
export interface InjectedExtension {
    name: string;
    version: string;
    accounts: InjectedAccounts;
    metadata?: InjectedMetadata;
    provider?: InjectedProvider;
    signer: InjectedSigner;
}
```

* `name`: the name of the extension, e.g. `"polkadot-js"`.
* `version`: the version of the extension, e.g. `"0.44.1"`.
* `accounts`: an [Accounts](#accounts) structure to retrieve accounts.
* `metadata`: a [Metadata](#metadata) structure containing metadata (optional).
* `provider`: a [Provider](#rpc-provider) structure to allow the Dapp to submit
  RPC request to the network (optional).
* `signer`: a [Signer](#signer) structure to sign messages with a given account.

#### Accounts

```typescript
export interface InjectedAccounts {
  get: (anyType?: boolean) => Promise<InjectedAccount[]>;
  subscribe: (cb: (accounts: InjectedAccount[]) => void | Promise<void>) => Unsubcall;
}

export type Unsubcall = () => void;
```

* `get`: returns a list of accounts.
* `subscribe`: subscribers to the list of accounts, useful for detecting account
  changes. The returns value of type `Unsubcall` will stop the subscription if
  called.

The account structure is defined as follows:

```typescript
export interface InjectedAccount {
  address: string;
  genesisHash?: string | null;
  name?: string;
  type?: KeypairType;
}

export type KeypairType = 'ed25519' | 'sr25519' | 'ecdsa' | 'ethereum';
```

* `address`: the address of the account.
* `genesisHash`: the genesis hash of the chain (optional).
* `name`: the custom name of the account (optional).
* `type`: the cryptographic type of the account (optional).

#### Metadata

```typescript
export interface InjectedMetadata {
  get: () => Promise<InjectedMetadataKnown[]>;
  provide: (definition: MetadataDef) => Promise<boolean>;
}

export interface InjectedMetadataKnown {
  genesisHash: string;
  specVersion: number;
}
```

#### RPC Provider

```typescript
export interface InjectedProvider extends ProviderInterface {
  listProviders: () => Promise<ProviderList>;
  startProvider: (key: string) => Promise<ProviderMeta>;
}

export type ProviderList = Record<string, ProviderMeta>

// Metadata about a provider
export interface ProviderMeta {
  // Network of the provider
  network: string;
  // Light or full node
  node: 'full' | 'light';
  // The extension source
  source: string;
  // Provider transport: 'WsProvider' etc.
  transport: string;
}

export interface ProviderInterface {
  readonly hasSubscriptions: boolean;
  readonly isConnected: boolean;
  readonly stats?: ProviderStats;

  clone (): ProviderInterface;
  connect (): Promise<void>;
  disconnect (): Promise<void>;
  on (type: 'connected' | 'disconnected' | 'error', sub: (value?: any) => any): () => void;
  send <T = any> (method: string, params: unknown[], isCacheable?: boolean): Promise<T>;
  subscribe (type: string, method: string, params: unknown[], cb: (error: Error | null, result: any) => void): Promise<number | string>;
  unsubscribe (type: string, method: string, id: number | string): Promise<boolean>;
}

export interface ProviderStats {
  active: {
    requests: number;
    subscriptions: number;
  };
  total: {
    bytesRecv: number;
    bytesSent: number;
    cached: number;
    errors: number;
    requests: number;
    subscriptions: number;
    timeout: number;
  };
}

export interface SignerPayloadJSON {
  address: string;
  blockHash: string;
  blockNumber: string;
  era: string;
  genesisHash: string;
  method: string;
  nonce: string;
  specVersion: string;
  tip: string;
  transactionVersion: string;
  signedExtensions: string[];
  version: number;
}

export interface SignerPayloadRaw extends SignerPayloadRawBase {
  address: string;
  type: 'bytes' | 'payload';
}

export interface SignerResult {
  id: number;
  signature: HexString;
}

export interface ISubmittableResult {
  readonly dispatchError?: DispatchError;
  readonly dispatchInfo?: DispatchInfo;
  readonly events: EventRecord[];
  readonly internalError?: Error;
  readonly status: ExtrinsicStatus;
  readonly isCompleted: boolean;
  readonly isError: boolean;
  readonly isFinalized: boolean;
  readonly isInBlock: boolean;
  readonly isWarning: boolean;
  readonly txHash: Hash;
  readonly txIndex?: number;

  filterRecords (section: string, method: string): EventRecord[];
  findRecord (section: string, method: string): EventRecord | undefined;
  toHuman (isExtended?: boolean): AnyJson;
}
```

#### Signer

```typescript
export interface Signer {
  signPayload?: (payload: SignerPayloadJSON) => Promise<SignerResult>;
  signRaw?: (raw: SignerPayloadRaw) => Promise<SignerResult>;
  update?: (id: number, status: H256 | ISubmittableResult) => void;
}
```

## Tests

...

## Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
