# Wallet Interaction Standard

- **PSP Number:** 23
- **Authors:** Andreas Gassmann <a.gassmann@papers.ch>, Alessandro De Carli <a.decarli@papers.ch>, Mike Godenzi <m.godenzi@papers.ch>, Pascal Brun <@pascuin>
- **Status:** Draft
- **Created:** 2021-10-23
- **Reference** [Beacon SDK](https://github.com/airgap-it/beacon-sdk)

## Summary

Communication between wallets and decentralised applications.

## Abstract

This document describes the communication between a dApp (decentralised application) and wallet (eg. mobile wallet).

## Motivation

A user wants to be able to interact with decentralised applications e.g. uniswap-like decentralised exchange or a game that uses NFTs, they want to use their preferred wallet and avoid having to install a new wallet for each application they use.

To enable adoption of decentralised applications in the Polkadot ecosystem, a standard for the communication between these apps and wallets is needed. Application developers shouldn't need to implement yet another wallet just for their use case and users shouldn't need a multitude of wallets just to use certain services.

This standard should form a consensus of how the types of messages for this communication look like.

## Specification

### Communication Protocol Version 1.0.0 (Version code 1)

The entire protocol relies on asynchronous calls. A call consists of a request message and an asynchronous response message.

## Versioning

The transport layer is responsible for the communication of the protocol version. All of the discussed messages will include the version number.

## Message Types

The following chapters describe the different message types that exist between the app and wallet. This protocol assumes a secure and authenticated transport layer was established hence no messages can be spoofed.

### Shared Types

```typescript
export interface AppMetadata {
  senderId: string;
  name: string;
  icon?: string;
}
```

#### Network

// THIS SECTION IS NOT YET FINAL AND CURRENTLY BEING WORKED ON

```typescript
export interface Network {
  type: NetworkType;
  genesisHash: string;
  name?: string;
  rpcUrl?: string;
}
```

#### BaseMessage

```typescript
export enum MessageType {
  PermissionRequest = "permission_request",
  SignPayloadRequest = "sign_payload_request",
  OperationRequest = "operation_request",
  BroadcastRequest = "broadcast_request",
  PermissionResponse = "permission_response",
  SignPayloadResponse = "sign_payload_response",
  OperationResponse = "operation_response",
  BroadcastResponse = "broadcast_response",
  Disconnect = "disconnect",
  Error = "error",
}

export interface BaseMessage {
  type: MessageType;
  version: string;
  id: string; // ID of the message. The same ID is used in the request and response
  senderId: string; // ID of the sender. This is used to identify the sender of the message
}
```

### Communication Flow

After the channel between app and wallet has been opened (see Transport Layers chapter), the communication should always be initiated with a `PermissionRequest`.

`OperationRequest` and `SignPayloadRequest` always have to be addressed to a specific account in the wallet. It is not possible to send either of those requests without a prior `PermissionRequest`. If an operation is requested where no permissions have been given, or they have been rejected, a `NO_PERMISSION` error will be returned.

The `PermissionRequest` and `BroadcastRequest` can be sent without a prior `PermissionRequest` because they are not directly related to a specific account.

### 1. Permission

App requests permission to access account details from wallet.

#### Request

```typescript
export enum PermissionScope {
  SIGN = "sign",
  OPERATION_REQUEST = "operation_request",
}

export interface PermissionRequest extends BaseMessage {
  version: string;
  id: string;
  senderId: string;
  type: MessageType.PermissionRequest;
  appMetadata: AppMetadata;
  network: Network;
  scopes: PermissionScope[];
}
```

`appMetadata`: An App should be identifiable by the user, for example with a name and icon.

`network`: Network on which the permissions are requested. Only one network can be specified. In case you need permissions on multiple networks, you need to request permissions multiple times.

`scope`: The App should ask for permissions to do certain things, for example the App can request to sign a transaction (maybe even without confirmation for micro-transactions). The following scopes are defined:

- `sign`: this indicates the app wants to be able to sign.
- `operation_request`: this indicates the app wants to perform payment requests.

#### Response

```typescript
export interface PermissionResponse extends BaseMessage {
  version: string;
  id: string;
  senderId: string;
  type: MessageType.PermissionResponse;
  publicKey: string; // Public Key, because it can be used to verify signatures
  network: Network; // Network on which the permissions have been granted
  scopes: PermissionScope[]; // Permissions that have been granted for this specific address / account
}
```

- `publicKey`: The public key of the account
- `network`: The network on which the permissions have been granted. (Should usually be the same as has been requested)
- `scopes`: The permission scopes that have been granted

#### Errors

[`NO_ADDRESS_ERROR`](#5-error): Will be returned if there is no address present for the protocol / network requested.

[`NOT_GRANTED_ERROR`](#5-error): Will be returned if the permissions requested by the App were not granted.

### 2. Sign Payload

App requests that an arbitrary payload / message is signed.

#### Request

```typescript
export interface SignPayloadRequest extends BaseMessage {
  type: MessageType.SignPayloadRequest;
  payload: string;
  sourceAddress: string;
}
```

- `payload`: The unsigned payload as a string. It does not have to be a valid transaction
- `sourceAddress`: The address that will be used to select the keypair for signing

#### Response

```typescript
export interface SignPayloadResponse extends BaseMessage {
  type: MessageType.SignPayloadResponse;
  signature: string; // Signature of the payload
}
```

`signature`: The signature of the payload

#### Errors

[`NOT_GRANTED_ERROR`](#5-error): Will be returned if the signature was blocked

[`NO_PRIVATE_KEY_FOUND_ERROR`](#5-error): Will be returned if the private key matching the sourceAddress could not be found

### 3. Operation Request

App sends partial operation request to the wallet and the wallet will prepare the transaction and broadcast it.

"Partial" means, that only the necessary values need to be set (eg. recipient, amount).

// THIS SECTION IS NOT YET FINAL AND CURRENTLY BEING WORKED ON, A FINAL OPERATION MESSAGE FORMAT HAS NOT YET BEEN DEFINED

#### Request

```typescript
export interface OperationRequest extends BaseMessage {
  type: MessageType.OperationRequest;
  network: Network;
  operationDetails: PartialOperation[];
  sourceAddress: string;
}
```

`network`: Network on which the operation will be broadcast

`operationDetails`: Partial operations that may lack certain information that the wallet can add

`sourceAddress`: The address that will be used to select the keypair for signing

#### Response

```typescript
export interface OperationResponse extends BaseMessage {
  type: MessageType.OperationResponse;
  transactionHash: string;
}
```

`transactionHash`: Hash of the broadcasted transaction

#### Errors

[`PARAMETERS_INVALID_ERROR`](#5-error): Will be returned if any of the parameters are invalid

[`BROADCAST_ERROR`](#5-error): Will be returned if the user choses that the transaction is broadcast but there is an error (eg. node not available)

### 4. Broadcast Transactions

App requests a signed transaction to be broadcast.

#### Request

```typescript
export interface BroadcastRequest extends BaseMessage {
  type: MessageType.BroadcastRequest;
  network: Network;
  signedTransaction: string;
}
```

`network`: Network on which the transaction will be broadcast

`signedTransaction`: Signed transaction that will be broadcast

#### Response

```typescript
export interface BroadcastResponse extends BaseMessage {
  type: MessageType.BroadcastResponse;
  transactionHash: string;
}
```

`transactionHash`: Hash of the broadcasted transaction

#### Errors

[`TRANSACTION_INVALID_ERROR`](#5-error): Will be returned if the transaction is not parsable or is rejected by the node.

[`BROADCAST_ERROR`](#5-error): Will be returned if the user choses that the transaction is broadcast but there is an error (eg. node not available).

### 5. Error

In case of an error, the wallet will send back a dedicated message with an `errorType`. The actual error should be handled in the wallet and will not be sent back to the app.

```typescript
export interface ErrorResponse extends BaseMessage {
  type: MessageType.Error;
  errorType: ErrorType;
}

export enum ErrorType {
  BROADCAST_ERROR = "BROADCAST_ERROR", // Broadcast | Operation Request: Will be returned if the user chooses that the transaction is broadcast but there is an error (eg. node not available).
  NETWORK_NOT_SUPPORTED = "NETWORK_NOT_SUPPORTED", // Permission: Will be returned if the selected network is not supported by the wallet / extension.
  NO_ADDRESS_ERROR = "NO_ADDRESS_ERROR", // Permission: Will be returned if there is no address present for the protocol / network requested.
  NO_PRIVATE_KEY_FOUND_ERROR = "NO_PRIVATE_KEY_FOUND_ERROR", // Sign: Will be returned if the private key matching the sourceAddress could not be found.
  NOT_GRANTED_ERROR = "NOT_GRANTED_ERROR", // Sign: Will be returned if the signature was blocked // (Not needed?) Permission: Will be returned if the permissions requested by the App were not granted.
  PARAMETERS_INVALID_ERROR = "PARAMETERS_INVALID_ERROR", // Operation Request: Will be returned if any of the parameters are invalid.
  TRANSACTION_INVALID_ERROR = "TRANSACTION_INVALID_ERROR", // Broadcast: Will be returned if the transaction is not parsable or is rejected by the node.
  ABORTED_ERROR = "ABORTED_ERROR", // Permission | Operation Request | Sign Request | Broadcast: Will be returned if the request was aborted by the user or the wallet.
  UNKNOWN_ERROR = "UNKNOWN_ERROR", // Used as a wildcard if an unexpected error occured.
}
```

### 6. Disconnect

A message that indicates that the app or wallet wants to terminate the connection. This is a "fire and forget" message, no response is expected.

```typescript
export interface DisconnectMessage extends BaseMessage {
  type: MessageType.Disconnect;
}
```

## Transport Layers

This standard should be compatible with different transport layers. We will cover 2 common transport layers here, a `PostMessageTransport` (to talk to browser extensions) and a `P2PTransport` (communicating over a peer-to-peer network).

The user should always be given the option to connect to any wallet, no matter the transport.

#### Serialisation

The serialisation used for the transport layer messages requires an algorithm which is well known and fast to encode/decode.

`JSON` is used as the underlying structure with `base58check` encoding. The encoding is well known and produces URL compatible strings.

```typescript
const str = JSON.stringify(message);

return bs58check.encode(Buffer.from(str));
```

#### Handshake (opening a channel)

Communication between app and wallet requires a secure setup. This is the proposed handshake process:

1. We define the communication/transport layer as 'channel'
2. App generates and stores (e.g. local storage) a channel asymmetric keypair. This keypair will be reused for all channels
3. App serialises public key, app name, app id and app icon in an handshake uri <sup>1</sup>
4. Handshake uri is either sent to the wallet (PostMessage) or shown as QR / clickable link (P2PTransport)
5. Wallet receives handshake uri
6. Wallet generates and stores (e.g. local storage) a channel asymmetric keypair. This keypair will be reused for all channels
7. Wallet serialises its own public key and info in a handshake uri
8. Wallet uses PostMessage or P2P network layer to reach out to app and send its own handshake uri
9. Wallet and app compute DH symmetric channel key
10. Symmetric encrypted acknowledgment message is sent
11. Channel is open

The handshake messages are different depending on the transport and will be described in the respective chapter.

**Why not simply use directly the crypto address in the wallet?**  
There would be one big upside namely the wallet-app channel could be easily restored on any wallet that imported the crypto private key. This would also allow the user to accept/confirm messages coming from the app on different devices. The reason for the decision against such a solution is because it would imply that if an app sets up a channel once with a wallet it will always be able to send messages to it in future and since security is of utmost importance for this standard the decision has been made to have random channel keys which can be destroyed/ignored when an app turns rogue.

<sup>1</sup> app id and app icon can be spoofed by any dapp, however the browser extension should make sure that spoofing is avoided. Also the user always has to check him/herself if the shown URL matches
the one in his/her address bar.

#### Closing a channel

Closing a channel will require one of the parties to send a channel close operation. After closing the channel the party will no longer listen to the channel (fire and forget).

#### Use Cases

The following use cases should be covered

| DApp    | Wallet    | Important Use Case | Communication | Handshake        |
| ------- | --------- | ------------------ | ------------- | ---------------- |
| Desktop | Mobile    | &#x2705;           | P2P           | QR scanning      |
| Desktop | Desktop   | &#x2705;           | P2P           | Desktop Deeplink |
| Desktop | Browser   | &#x2705;           | P2P           | Browser Deeplink |
| Desktop | Extension | &#x2705;           | PostMessage   | PostMessage      |
| Mobile  | Mobile    | &#x2705;           | P2P           | Mobile Deeplink  |
| Mobile  | Desktop   | &#x274C;           | P2P           | Copy Paste       |
| Mobile  | Browser   | &#x274C;           | P2P           | Copy Paste       |
| Mobile  | Extension | &#x274C;           | P2P           | Copy Paste       |

### P2P Transport

The peer-to-peer transport should cover the following use cases:

- Same Device Desktop to Desktop
- Same Device Desktop to Hardware Wallet
- Same Device Mobile to Mobile
- Two Device Desktop to online Mobile
- Two Device Desktop to offline Mobile

As the P2P transport layer, the implementation is built on the decentralized messaging service [matrix protocol](https://matrix.org/). The [Beacon](https://walletbeacon.io) implementation has been utilizing this approach for a while now and is has proven effective.

The main reasons why we propose to use a decentralised messaging service as the transport layer between app and wallet are:

- Decentralization
- Enables modern user experience
- Covers all of the scenarios described above
- Easy to integrate with all kind of wallets
- Minimal effort to "join" for a wallet
- Support for oracles (e.g. push service)

#### P2PTransport handshake

The following request has to be serialized using `JSON.stringify`, `base58check` encoded and displayed as a QR code or converted into a clickable link.

```typescript
export interface P2PPairingRequest {
  name: string;
  icon?: string;
  appUrl?: string;
  publicKey: string;
  relayServer: string;
}

export interface P2PPairingResponse {
  name: string;
  icon?: string;
  appUrl?: string;
  publicKey: string;
}
```

The response is sent pack in the following format as a string (over the matrix protocol).

```typescript
["@channel-open", recipient, encryptedMessage].join(":");
```

#### P2PTransport communication

The serialized message described above will be encrypted and sent through the channel to the recipient.

### PostMessage Transport (Browser Extension)

#### Browser extension detection

The detection of a compatible chrome extension should be done by using a ping-pong approach.

After loading, the app will send a message containing `ping` via postMessage interface. The browser extension will reply with a `pong` as soon as possible.

```typescript
// App: Sending ping

const message = {
  target: ExtensionMessageTarget.EXTENSION,
  payload: "ping",
};

window.postMessage(message as any, "*");

// Extension: Sending pong

const message = {
  target: ExtensionMessageTarget.PAGE,
  payload: "pong",
};
```

#### Browser extension handshake

As described above, a secure connection channel has to be established. For this, the app will first send a `PairingRequest` to the extension. The extension will store the information it receives and send back its own information in the `PairingResponse`

```typescript
export interface PostMessagePairingRequest {
  name: string;
  icon?: string;
  appUrl?: string;
  publicKey: string;
}

export interface PostMessagePairingResponse {
  name: string;
  icon?: string;
  appUrl?: string;
  publicKey: string;
}
```

After this handshake, the public keys have been exchanged and the subsequent communication is encrypted. This makes sure that no messages can be spoofed.

#### Browser extension communication

The serialized message described above will be encrypted and wrapped in an object before being sent via PostMessage.

```typescript
export enum ExtensionMessageTarget {
  BACKGROUND = "toBackground",
  PAGE = "toPage",
  EXTENSION = "toExtension",
}

export interface ExtensionMessage<T, U = unknown> {
  target: ExtensionMessageTarget;
  encryptedPayload: T;
}
```

### Deeplinks

Deeplinks allow us to cover more combinations of DApp (Desktop), DApp (mobile) with web-wallets, mobile wallets and desktop wallets. This is only used for the handshake, after the initial handshake the P2P Transport described above is used.

#### Deeplink Handshake

// THIS SECTION IS NOT YET FINAL AND CURRENTLY BEING WORKED ON

`substrate://?type=psp23&data=<data>`

## Tests

_TBD: If applicable, please include a list of potential test cases to validate an implementation._

## Implementations

- [Beacon SDK Typescript](https://github.com/airgap-it/beacon-sdk)
- [Beacon SDK Android](https://github.com/airgap-it/beacon-android-sdk)
- [Beacon SDK iOS](https://github.com/airgap-it/beacon-ios-sdk)

## Apendix

### Wallets

Wallets implementing the defined standard.

- [AirGap](https://airgap.it)
- [Fearless - Proof of Concept](https://fearlesswallet.io/)

### Applications

Applications implementing the defined standard.

- [Beacon Landing Page](https://walletbeacon.io/)
- [Example DApp](https://airgap-it.github.io/beacon-polkadot-example-dapp/)

### Libraries using with implementations

Libraries implementing the defined standard.

- TBA

## Copyright

This document is placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
