# App Extension API

- **PSP Number:** 52
- **Authors:** Fabio Lama <fabio.lama@pm.me>
- **Status:** Draft
- **Created:** 2022-08-30
- **Reference Implementation** https://github.com/polkadot-js/extension/tree/master/packages/extension-app

## Summary

This PSP describes how web applications ("apps") and browser extensions, such as
wallets like the [polkadot{.js} extensions](https://polkadot.js.org/extension/),
can interact with each other. Essentially, apps should be able to interact with
any extension that implements this standard and vice-versa.

## Motivation

This standard unifies the Polkadot and Kusama network by allowing apps to
interact with commonly used extensions and makes it easier to develop new
extensions that are compatible with existing apps.

## Specification

This specification only describes the interface for how apps and extensions
should interact with each other. Everything else is implementation specific.
Addtionally, all types are expressed in Typescript and are therefore also
accessible with javascript.

### Communication

Extensions inject an `injectedWeb3` attribute with a specific datastructure into
the [`window` object](https://developer.mozilla.org/en-US/docs/Web/API/Window).
Apps can then interspect that attribute, search for the desired extension and
then interact with the extension by calling the defined functions. Implementers
of extension can decide for themselves on how the functions are implemented, as
long as the standardized structures are defined correctly.

The `window.injectedWeb3` is of type:

```typescript
Record<string, InjectedWindowProvider>
```

The key identifies the name of the extension (e.g. `polkadot-js`), which the
app can use to identify the correct extension. Its corresponding value is a
datastructure of the following format:

```typescript
interface InjectedWindowProvider {
    // Start communication process.
    enable: (origin: string) => Promise<InjectedExtension>;
    // Version of the extension.
    version: string;
}
```

To start the communication with the extension, the app calls the `enable`
function, passing the `origin` parameter indicating the arbitrary name of the
app, where an action is then executed by the extension. Normally, this is where
the extension asks the user for permission on whehter the app should be allowed
to access the extension or throws an exception if not.

The returned value contains (meta)data about the extension and offers some
functions such as the ability to retrieve accounts, sign messages and interact
with RPC servers. The extension itself does not create any sort of transactions
itself, the apps is responsible for that.

### Types

#### Extension

This structure is the primary way to interact with the extension.

```typescript
interface InjectedExtension {
    // The name of the extension, e.g. `"polkadot-js"`
    name: string;
    // The version of the extension, e.g. `"0.44.1"`
    version: string;
    // A structure to retrieve accounts.
    accounts: InjectedAccounts;
    // A structure containing metadata (optional)
    metadata?: InjectedMetadata;
    // A structure to allow the app to submit
    // RPC request to the network (optional).
    provider?: InjectedProvider;
    // A structure to sign messages with a given account.
    signer: InjectedSigner;
}
```

#### Accounts

The accounts structure allows the apps to retrieve a list of accounts,
including listening for account changes. The extension is responsible for
handling permissions accordingly.

```typescript
interface InjectedAccounts {
    // Returns a list of accounts.
    get: (anyType?: boolean) => Promise<InjectedAccount[]>;
    // Subscribers to the list of accounts, useful for
    // detecting account changes. 
    subscribe: (cb: (accounts: InjectedAccount[]) => void | Promise<void>) => Unsubcall;
}

// Stops the subscription if called.
type Unsubcall = () => void;

interface InjectedAccount {
// The address of the account.
  address: string;
  // The genesis hash of the blockchain (optional).
  genesisHash?: string | null;
  // The custom name of the account (optional).
  name?: string;
  // The cryptographic type of the account (optional).
  type?: KeypairType;
}

type KeypairType = 'ed25519' | 'sr25519' | 'ecdsa' | 'ethereum';
```

#### Metadata

The metadata stucture allows apps to retrieve information about known
blockchain, but also allows to register new blockchains with the extension. The
extension is responsible for handling permissions accordingly.

```typescript
interface InjectedMetadata {
    // Retuns a list of know parameters about the blockchain.
    get: () => Promise<InjectedMetadataKnown[]>;
    // Provide the extension with information about a blockchain.
    provide: (definition: MetadataDef) => Promise<boolean>;
}

interface InjectedMetadataKnown {
    // The genesis hash of the blockchain.
    genesisHash: string;
    // The chain spec version of the blockchain
    // (https://docs.substrate.io/build/chain-spec/).
    specVersion: number;
}

interface MetadataDef {
    // The name of the blockchain.
    chain: string;
    // The genesis hash of the blockchain.
    genesisHash: string;
    // HEX-encoded icon of the blockchain.
    icon: string;
    // The SS58-encoding prefix of the blockchain
    // (https://docs.substrate.io/fundamentals/accounts-addresses-keys/).
    ss58Format: number;
    // The chain type.
    chainType?: 'substrate' | 'ethereum'
    // A chosen color for the blockchain (optional).
    color?: string;
    // The chain specification version of the blockchain
    // (https://docs.substrate.io/build/chain-spec/).
    specVersion: number;
    // The number of decimals the token supports.
    tokenDecimals: number;
    // The name of the token.
    tokenSymbol: string;
    types: Record<string, Record<string, string> | string>;
    metaCalls?: string;
    userExtensions?: Record<string, ExtInfo>;
}

type ExtInfo = {
  extrinsic: ExtTypes;
  payload: ExtTypes;
}
```

#### Signer

The signer datastructure allows apps to sign messages with a given account. The
extension is responsible for handling permissions accordingly.

```typescript
interface Signer {
    // Signs the given JSON payload and returns the result.
    signPayload?: (payload: SignerPayloadJSON) => Promise<SignerResult>;
    // Signs the given raw payload and returns the result.
    signRaw?: (raw: SignerPayloadRaw) => Promise<SignerResult>;
    update?: (id: number, status: H256 | ISubmittableResult) => void;
}

interface SignerPayloadJSON {
    // The address of the account that should sign the message.
    address: string;
    // The HEX-encoded block hash.
    blockHash: string;
    // The block number.
    blockNumber: string;
    // The era of the blockchain.
    era: string;
    // The gensis hash 
    genesisHash: string;
    method: string;
    // The lastest nonce for the given account.
    nonce: string;
    // The chain spec version of the blockchain
    // (https://docs.substrate.io/build/chain-spec/).
    specVersion: string;
    // The tip for the block author.
    // TODO: As decimals?
    tip: string;
    // The transaction version.
    transactionVersion: string;
    signedExtensions: string[];
    version: number;
}

interface SignerPayloadRaw {
    address: string;
    type: 'bytes' | 'payload';
}

interface SignerResult {
    id: number;
    // The HEX-encoded signature.
    signature: string;
}

interface ISubmittableResult {
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

#### RPC Provider

The RPC provider structure allows apps to communicate with an RPC server, such
as submitting transactions. The extension is responsible for initiating and
maintaining the connection to the RPC server, including handling all requests
and forwarding responses to the app.

```typescript
interface ProviderInterface {
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

interface ProviderStats {
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

interface InjectedProvider extends ProviderInterface {
  listProviders: () => Promise<ProviderList>;
  startProvider: (key: string) => Promise<ProviderMeta>;
}

type ProviderList = Record<string, ProviderMeta>

// Metadata about a provider
interface ProviderMeta {
  // Network of the provider
  network: string;
  // Light or full node
  node: 'full' | 'light';
  // The extension source
  source: string;
  // Provider transport: 'WsProvider' etc.
  transport: string;
}
```

##### Error Types

```typescript
interface DispatchError {
  readonly isOther: boolean;
  readonly isCannotLookup: boolean;
  readonly isBadOrigin: boolean;
  readonly isModule: boolean;
  readonly asModule: DispatchErrorModule;
  readonly isConsumerRemaining: boolean;
  readonly isNoProviders: boolean;
  readonly isTooManyConsumers: boolean;
  readonly isToken: boolean;
  readonly asToken: TokenError;
  readonly isArithmetic: boolean;
  readonly asArithmetic: ArithmeticError;
  readonly isTransactional: boolean;
  readonly asTransactional: TransactionalError;
  readonly isExhausted: boolean;
  readonly isCorruption: boolean;
  readonly isUnavailable: boolean;
  readonly type: 'Other' | 'CannotLookup' | 'BadOrigin' | 'Module' | 'ConsumerRemaining' | 'NoProviders' | 'TooManyConsumers' | 'Token' | 'Arithmetic' | 'Transactional' | 'Exhausted' | 'Corruption' | 'Unavailable';
}

interface DispatchErrorModule {
  readonly index: u8;
  // TODO
  readonly error: U8aFixed;
}

interface TokenError {
  readonly isNoFunds: boolean;
  readonly isWouldDie: boolean;
  readonly isBelowMinimum: boolean;
  readonly isCannotCreate: boolean;
  readonly isUnknownAsset: boolean;
  readonly isFrozen: boolean;
  readonly isUnsupported: boolean;
  readonly isUnderflow: boolean;
  readonly isOverflow: boolean;
  readonly type: 'NoFunds' | 'WouldDie' | 'BelowMinimum' | 'CannotCreate' | 'UnknownAsset' | 'Frozen' | 'Unsupported' | 'Underflow' | 'Overflow';
}

interface ArithmeticError {
  readonly isUnderflow: boolean;
  readonly isOverflow: boolean;
  readonly isDivisionByZero: boolean;
  readonly type: 'Underflow' | 'Overflow' | 'DivisionByZero';
}

interface TransactionalError extends Enum {
  readonly isLimitReached: boolean;
  readonly isNoLayer: boolean;
  readonly type: 'LimitReached' | 'NoLayer';
}
```

## Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
