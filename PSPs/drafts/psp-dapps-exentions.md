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

Extensions inject a `injectedWeb3` field with a specific structure into the
`window` entry in the DOM. Dapps can then interspect that field, search for the
desired extension and then interact with the extension by calling the defined
functions.

`window.injectedWeb` is defined as TODO

The datastucture contains (meta)data about the extension and offers some
functions such as the ability to retrieve accounts and sign messages. The
extension itself does not create any sort of transactions and does not submit
anything to any network, the dapps is responsible for that. Ultimately, the
extension primarily manages accounts and creates signatures of messages.

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
