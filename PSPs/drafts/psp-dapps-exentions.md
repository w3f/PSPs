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
### Top Level Function

export { packageInfo } from './packageInfo';
export { unwrapBytes, wrapBytes } from './wrapBytes';
declare let isWeb3Injected: boolean;
declare let web3EnablePromise: Promise<InjectedExtension[]> | null;


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


## Tests

...

## Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
