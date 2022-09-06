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
  get: (anyType?: boolean) => Promise<InjectedAccount[]>;
  subscribe: (cb: (accounts: InjectedAccount[]) => void | Promise<void>) => Unsubcall;
}
```

```typescript
export interface InjectedMetadata {
  get: () => Promise<InjectedMetadataKnown[]>;
  provide: (definition: MetadataDef) => Promise<boolean>;
}
```

```typescript
export interface InjectedProvider extends ProviderInterface {
  listProviders: () => Promise<ProviderList>;
  startProvider: (key: string) => Promise<ProviderMeta>;
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
```

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
