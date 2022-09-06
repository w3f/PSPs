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


### Types

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

```typescript
export interface InjectedAccounts {
}
```

```typescript
export interface InjectedMetadata {
}
```

```typescript
export interface InjectedProvider {
}
```

```typescript
export interface InjectedSigner {
}
```


## Tests

...

## Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
