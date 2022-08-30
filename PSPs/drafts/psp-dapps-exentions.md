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

### Top Level Variables

#### isWeb3Injected

Declaration:

```typescript
let isWeb3Injected: boolean;
```

#### web3EnablePromise

Declaration:

```typescript
let web3EnablePromise: Promise<InjectedExtension[]> | null;
```

#### packageInfo

Declaration:

```typescript
const packageInfo: {
    name: string;
    path: string;
    type: string;
    version: string;
};
```

### Top Level Function

> TODO: Should `export { unwrapBytes, wrapBytes } from './wrapBytes';` be documented?

#### web3Enable

Declaration:

```typescript
function web3Enable(originName: string, compatInits?: (() => Promise<boolean>)[]): Promise<InjectedExtension[]>
```

#### web3Accounts

Declaration:

```typescript
function web3Accounts({ accountType, extensions, ss58Format }?: Web3AccountsOptions): Promise<InjectedAccountWithMeta[]>
```

#### web3AccountsSubscribe

Declaration:

```typescript
function web3AccountsSubscribe(cb: (accounts: InjectedAccountWithMeta[]) => void | Promise<void>, { extensions, ss58Format }?: Web3AccountsOptions): Promise<Unsubcall>
```

#### web3FromSource

Declaration:

```typescript
function web3FromSource(source: string): Promise<InjectedExtension>
```

#### web3FromAddress

Declaration:

```typescript
function web3FromAddress(address: string): Promise<InjectedExtension>
```

#### web3ListRpcProviders

Declaration

```typescript
function web3ListRpcProviders(source: string): Promise<ProviderList | null>
```

#### web3UseRpcProvider

Declaration

```typescript
function web3UseRpcProvider(source: string, key: string): Promise<InjectedProviderWithMeta>
```

### Types

```typescript
export declare type InjectedExtension = InjectedExtensionInfo & Injected;

export interface InjectedExtensionInfo {
    name: string;
    version: string;
}

export interface Injected {
    accounts: InjectedAccounts;
    metadata?: InjectedMetadata;
    provider?: InjectedProvider;
    signer: InjectedSigner;
}
```

```typescript
export interface InjectedAccountWithMeta {
    address: string;
    meta: {
        genesisHash?: string | null;
        name?: string;
        source: string;
    };
    type?: KeypairType;
}
```

```typescript
export declare type ProviderList = Record<string, ProviderMeta>;

export interface ProviderMeta {
    network: string;
    node: 'full' | 'light';
    source: string;
    transport: string;
}
```

```typescript
export interface InjectedProviderWithMeta {
    provider: InjectedProvider;
    meta: ProviderMeta;
}

export interface InjectedProvider extends ProviderInterface {
    listProviders: () => Promise<ProviderList>;
    startProvider: (key: string) => Promise<ProviderMeta>;
}

export interface ProviderInterface {
    readonly hasSubscriptions: boolean;
    readonly isConnected: boolean;
    readonly stats?: ProviderStats;
    clone(): ProviderInterface;
    connect(): Promise<void>;
    disconnect(): Promise<void>;
    on(type: 'connected' | 'disconnected' | 'error', sub: (value?: any) => any): () => void;
    send<T = any>(method: string, params: unknown[], isCacheable?: boolean): Promise<T>;
    subscribe(type: string, method: string, params: unknown[], cb: (error: Error | null, result: any) => void): Promise<number | string>;
    unsubscribe(type: string, method: string, id: number | string): Promise<boolean>;
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
```

## Tests

...

## Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
