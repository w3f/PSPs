# 1. JSON-RPC Interfaces for Polkadot Nodes

* **PSP Number:** 006
* **Authors:** Fabio Lama <github.com/lamafab, fabio@web3.foundation>
* **Status:** Draft 
* **Date:** 2020-11-25
* **Reference Implementation:** <https://github.com/paritytech/substrate>

## 1.1. Summary

This standard describes the JSON-RPC interfaces that Polkadot node implementations should provide, including each function call's parameter types, behavior and possible return values.

## 1.2. Motivation

This standard should serve as  common ground for API endpoints in order for external applications to be able to reuse existing functionality on various implementations of the Polkadot node. Avoiding disparity among node implementations will simplify their development, as well as allow applications which use different node implementations to switch seamlessly between them.

## 1.3. Specification

[JSON-RPC](https://www.jsonrpc.org/specification) is a stateless, light-weight remote procedure call (RPC) protocol. Primarily this specification defines several data structures and the rules around their processing. It is transport agnostic in that the concepts can be used within the same process, over sockets, over http, or in many various message passing environments. It uses [JSON](https://www.json.org/json-en.html) ([RFC 4627](https://www.ietf.org/rfc/rfc4627.txt)) as data format.

All parameters in the specified JSON-RPC methods are **REQUIRED**, unless explicitly mentioned otherwise (**OPTIONAL**). Return values which are OPTIONAL indicate the possibility of `null` being returned. Additionally, the JSON-RPC API should be made available over the [WebSocket Protocol](https://tools.ietf.org/html/rfc6455).

Certain APIs, indicated by `pubsub`, communicate exclusively over the WebSocket protocol and follow the publish-subscribe pattern. The client needs to subscribe to those APIs by specifying the `method`. The publisher will then create response messages for the corresponding subscription (defined in the `"method"` field) and generate a `subscription` ID, which can either be an integer or a string. The client needs to keep track of that ID in order to unsubscribe from messages.

Request:

```json
{
    "id": 1,
    "jsonrpc": "2.0",
    "method": <METHOD>,
    "params": [
        <PARAMS>
    ]
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "method": <SUBSCRIPTION>,
    "params": {
        "result": <RESULT>,
        "subscription": <ID>
    }
}
```

This document contains examples on how `pubsub` APIs are supposed to be used.

### 1.3.1. Safety

Exposing RPC calls to the public may open up a huge surface of attacks, such as denial or service attacks, and has to be carefully considered. There are quite a few RPC methods that should be never (or rarely) directly exposed. Those methods are considered **UNSAFE** and are explicitly highlighted.

The implementation of this specification must disable the JSON API by default and should offer four different flags (for the RPC/TCP and the RPC/WebSocket interface each) in order to enable specific parts of the API. Two flags for exposing safe and unsafe PRC to localhost only, and two flags for exposing safe and unsafe RPC to all interfaces.

| API / Network | Localhost                 | All interfaces             |
| ------------- | ------------------------- | -------------------------- |
| Safe API      | `--local-{rpc/ws}`        | `--public-{rpc/ws}`        |
| Unsafe API    | `--local-unsafe-{rpc/ws}` | `--public-unsafe-{rpc/ws}` |

**Warning**: Enabling `--public-unsafe-{rpc|ws}` is dangerous and should be avoided.

### 1.3.2. Errors

Each namespace specifies RPC error code ranges of possible error types that can be returned. Known error types are defined in this document and some APIs may return undefined error messages. Defined error messages are subject to expansion and changes.

### 1.3.3. Common types

This document references commonly used types, represented in capital letters, as defined in the subsections below.

#### 1.3.3.1. STRING

A sequence of characters: `"value"`

#### 1.3.3.2. U32 / U64

Unsigned integers.

* `U32` - A 32-byte unsigned integer (min: `0`, max: `4294967295`).
* `U64` - A 64-byte unsigned integer (min: `0`, max: `18446744073709551615`).

#### 1.3.3.3. BOOL

A boolean type, which can either be `true` or `false`.

#### 1.3.3.4. MAP

A key/value object, where keys are `STRING`s and values are arbitrary types. Each API defines its own value types.

```json
{
    "key": "value1",
    "key2": "value2"
}
```

#### 1.3.3.5. ARRAY

An object containing a varying amount of items.

```json
[
    "item1",
    "item2"
]
```

#### 1.3.3.6. HASH

A string which represents a 32-byte, hexadecimal-encoded Blake2 hash. Contains a `0x` prefix to indicate the hexadecimal encoding. This type has 66 characters in total.

#### 1.3.3.7. HEX

A string of varying size representing hexadecimal-encoded data. Contains a `0x` prefix to indicate the hexadecimal encoding.

## 1.4. JSON-PRC methods

- [1. JSON-RPC Interfaces for Polkadot Nodes](#1-json-rpc-interfaces-for-polkadot-nodes)
  - [1.1. Summary](#11-summary)
  - [1.2. Motivation](#12-motivation)
  - [1.3. Specification](#13-specification)
    - [1.3.1. Safety](#131-safety)
    - [1.3.2. Errors](#132-errors)
    - [1.3.3. Common types](#133-common-types)
      - [1.3.3.1. STRING](#1331-string)
      - [1.3.3.2. U32 / U64](#1332-u32--u64)
      - [1.3.3.3. BOOL](#1333-bool)
      - [1.3.3.4. MAP](#1334-map)
      - [1.3.3.5. ARRAY](#1335-array)
      - [1.3.3.6. HASH](#1336-hash)
      - [1.3.3.7. HEX](#1337-hex)
  - [1.4. JSON-PRC methods](#14-json-prc-methods)
  - [1.5. System](#15-system)
    - [1.5.1. System Errors](#151-system-errors)
    - [1.5.2. system_name](#152-system_name)
      - [1.5.2.1. Parameter](#1521-parameter)
      - [1.5.2.2. Response](#1522-response)
      - [1.5.2.3. Example](#1523-example)
    - [1.5.3. system_version](#153-system_version)
      - [1.5.3.1. Parameter](#1531-parameter)
      - [1.5.3.2. Response](#1532-response)
      - [1.5.3.3. Example](#1533-example)
    - [1.5.4. system_chain](#154-system_chain)
      - [1.5.4.1. Parameter](#1541-parameter)
      - [1.5.4.2. Response](#1542-response)
      - [1.5.4.3. Example](#1543-example)
    - [1.5.5. system_chainType](#155-system_chaintype)
      - [1.5.5.1. Parameter](#1551-parameter)
      - [1.5.5.2. Response](#1552-response)
      - [1.5.5.3. Example](#1553-example)
    - [1.5.6. system_properties](#156-system_properties)
      - [1.5.6.1. Parameter](#1561-parameter)
      - [1.5.6.2. Response](#1562-response)
      - [1.5.6.3. Example](#1563-example)
    - [1.5.7. system_health](#157-system_health)
      - [1.5.7.1. Parameter](#1571-parameter)
      - [1.5.7.2. Response](#1572-response)
      - [1.5.7.3. Example](#1573-example)
    - [1.5.8. system_localPeerId](#158-system_localpeerid)
      - [1.5.8.1. Parameter](#1581-parameter)
      - [1.5.8.2. Response](#1582-response)
      - [1.5.8.3. Example](#1583-example)
    - [1.5.9. system_localListenAddresses](#159-system_locallistenaddresses)
      - [1.5.9.1. Parameter](#1591-parameter)
      - [1.5.9.2. Response](#1592-response)
      - [1.5.9.3. Example](#1593-example)
    - [1.5.10. system_peers](#1510-system_peers)
      - [1.5.10.1. Parameter](#15101-parameter)
      - [1.5.10.2. Response](#15102-response)
      - [1.5.10.3. Example](#15103-example)
    - [1.5.11. system_networkState](#1511-system_networkstate)
    - [1.5.12. system_addReservedPeer](#1512-system_addreservedpeer)
      - [1.5.12.1. Parameter](#15121-parameter)
      - [1.5.12.2. Response](#15122-response)
      - [1.5.12.3. Example](#15123-example)
    - [1.5.13. system_removeReservedPeer](#1513-system_removereservedpeer)
      - [1.5.13.1. Parameter](#15131-parameter)
      - [1.5.13.2. Response](#15132-response)
      - [1.5.13.3. Example](#15133-example)
    - [1.5.14. system_nodeRoles](#1514-system_noderoles)
      - [1.5.14.1. Parameter](#15141-parameter)
      - [1.5.14.2. Response](#15142-response)
      - [1.5.14.3. Example](#15143-example)
    - [1.5.15. system_syncState](#1515-system_syncstate)
      - [1.5.15.1. Parameter](#15151-parameter)
      - [1.5.15.2. Response](#15152-response)
      - [1.5.15.3. Example](#15153-example)
    - [1.5.16. system_accountNextIndex](#1516-system_accountnextindex)
      - [1.5.16.1. Parameter](#15161-parameter)
      - [1.5.16.2. Response](#15162-response)
      - [1.5.16.3. Example](#15163-example)
    - [1.5.17. system_dryRun](#1517-system_dryrun)
      - [1.5.17.1. Parameter](#15171-parameter)
      - [1.5.17.2. Response](#15172-response)
  - [1.6. Babe](#16-babe)
    - [1.6.1. babe_epochAuthorship](#161-babe_epochauthorship)
      - [1.6.1.1. Parameter](#1611-parameter)
      - [1.6.1.2. Response](#1612-response)
      - [1.6.1.3. Example](#1613-example)
  - [1.7. Grandpa](#17-grandpa)
    - [1.7.1. grandpa_roundState](#171-grandpa_roundstate)
      - [1.7.1.1. Parameter](#1711-parameter)
      - [1.7.1.2. Response](#1712-response)
      - [1.7.1.3. Example](#1713-example)
    - [1.7.2. grandpa_proveFinality](#172-grandpa_provefinality)
      - [1.7.2.1. Parameter](#1721-parameter)
      - [1.7.2.2. Response](#1722-response)
      - [1.7.2.3. Example](#1723-example)
    - [1.7.3. grandpa_subscribeJustifications (pubsub)](#173-grandpa_subscribejustifications-pubsub)
      - [1.7.3.1. Parameter](#1731-parameter)
      - [1.7.3.2. Response](#1732-response)
      - [1.7.3.3. Example](#1733-example)
    - [1.7.4. grandpa_unsubscribeJustifications (pubsub)](#174-grandpa_unsubscribejustifications-pubsub)
      - [1.7.4.1. Parameter](#1741-parameter)
      - [1.7.4.2. Response](#1742-response)
      - [1.7.4.3. Example](#1743-example)
  - [1.8. Author](#18-author)
    - [1.8.1. Common types](#181-common-types)
      - [1.8.1.1. Key types](#1811-key-types)
      - [1.8.1.2. Author Errors](#1812-author-errors)
    - [1.8.2. author_submitExtrinsic](#182-author_submitextrinsic)
      - [1.8.2.1. Parameter](#1821-parameter)
      - [1.8.2.2. Response](#1822-response)
    - [1.8.3. author_pendingExtrinsics](#183-author_pendingextrinsics)
      - [1.8.3.1. Parameter](#1831-parameter)
      - [1.8.3.2. Response](#1832-response)
    - [1.8.4. author_removeExtrinsic](#184-author_removeextrinsic)
      - [1.8.4.1. Parameter](#1841-parameter)
      - [1.8.4.2. Response](#1842-response)
    - [1.8.5. author_insertKey](#185-author_insertkey)
      - [1.8.5.1. Parameter](#1851-parameter)
      - [1.8.5.2. Response](#1852-response)
      - [1.8.5.3. Example](#1853-example)
    - [1.8.6. author_rotateKeys](#186-author_rotatekeys)
      - [1.8.6.1. Parameter](#1861-parameter)
      - [1.8.6.2. Response](#1862-response)
      - [1.8.6.3. Example](#1863-example)
    - [1.8.7. author_hasSessionKeys](#187-author_hassessionkeys)
      - [1.8.7.1. Parameter](#1871-parameter)
      - [1.8.7.2. Response](#1872-response)
      - [1.8.7.3. Example](#1873-example)
    - [1.8.8. author_hasKey](#188-author_haskey)
      - [1.8.8.1. Parameter](#1881-parameter)
      - [1.8.8.2. Response](#1882-response)
      - [1.8.8.3. Example](#1883-example)
    - [1.8.9. author_submitAndWatchExtrinsic (pubsub)](#189-author_submitandwatchextrinsic-pubsub)
      - [1.8.9.1. Parameter](#1891-parameter)
      - [1.8.9.2. Response](#1892-response)
      - [1.8.9.3. Example](#1893-example)
    - [1.8.10. author_unwatchExtrinsic (pubsub)](#1810-author_unwatchextrinsic-pubsub)
      - [1.8.10.1. Parameter](#18101-parameter)
      - [1.8.10.2. Response](#18102-response)
      - [1.8.10.3. Example](#18103-example)
  - [1.9. Chain](#19-chain)
    - [1.9.1. Chain Errors](#191-chain-errors)
    - [1.9.2. chain_getHeader](#192-chain_getheader)
      - [1.9.2.1. Parameter](#1921-parameter)
      - [1.9.2.2. Response](#1922-response)
      - [1.9.2.3. Example](#1923-example)
    - [1.9.3. chain_getBlock](#193-chain_getblock)
      - [1.9.3.1. Parameter](#1931-parameter)
      - [1.9.3.2. Response](#1932-response)
      - [1.9.3.3. Example](#1933-example)
    - [1.9.4. chain_getBlockHash](#194-chain_getblockhash)
      - [1.9.4.1. Parameter](#1941-parameter)
      - [1.9.4.2. Response](#1942-response)
      - [1.9.4.3. Example](#1943-example)
    - [1.9.5. chain_getFinalizedHead](#195-chain_getfinalizedhead)
      - [1.9.5.1. Parameter](#1951-parameter)
      - [1.9.5.2. Response](#1952-response)
      - [1.9.5.3. Example](#1953-example)
    - [1.9.6. chain_subscribeAllHeads (pubsub)](#196-chain_subscribeallheads-pubsub)
      - [1.9.6.1. Parameter](#1961-parameter)
      - [1.9.6.2. Response](#1962-response)
      - [1.9.6.3. Example](#1963-example)
    - [1.9.7. chain_unsubscribeAllHeads (pubsub)](#197-chain_unsubscribeallheads-pubsub)
      - [1.9.7.1. Parameter](#1971-parameter)
      - [1.9.7.2. Response](#1972-response)
      - [1.9.7.3. Example](#1973-example)
    - [1.9.8. chain_subscribeNewHeads (pubsub)](#198-chain_subscribenewheads-pubsub)
      - [1.9.8.1. Parameter](#1981-parameter)
      - [1.9.8.2. Response](#1982-response)
      - [1.9.8.3. Example](#1983-example)
    - [1.9.9. chain_unsubscribeNewHeads (pubsub)](#199-chain_unsubscribenewheads-pubsub)
      - [1.9.9.1. Parameter](#1991-parameter)
      - [1.9.9.2. Response](#1992-response)
      - [1.9.9.3. Example](#1993-example)
    - [1.9.10. chain_subscribeFinalizedHeads (pubsub)](#1910-chain_subscribefinalizedheads-pubsub)
      - [1.9.10.1. Parameter](#19101-parameter)
      - [1.9.10.2. Response](#19102-response)
      - [1.9.10.3. Example](#19103-example)
    - [1.9.11. chain_unsubscribeFinalizedHeads (pubsub)](#1911-chain_unsubscribefinalizedheads-pubsub)
      - [1.9.11.1. Parameter](#19111-parameter)
      - [1.9.11.2. Response](#19112-response)
      - [1.9.11.3. Example](#19113-example)
  - [1.10. Offchain](#110-offchain)
    - [1.10.1. Offchain Errors](#1101-offchain-errors)
    - [1.10.2. Storage kinds](#1102-storage-kinds)
    - [1.10.3. offchain_localStorageSet](#1103-offchain_localstorageset)
      - [1.10.3.1. Parameter](#11031-parameter)
      - [1.10.3.2. Response](#11032-response)
      - [1.10.3.3. Example](#11033-example)
    - [1.10.4. offchain_localStorageGet](#1104-offchain_localstorageget)
      - [1.10.4.1. Parameter](#11041-parameter)
      - [1.10.4.2. Response](#11042-response)
      - [1.10.4.3. Example](#11043-example)
  - [1.11. State](#111-state)
    - [1.11.1. State Errors](#1111-state-errors)
    - [1.11.2. Common types](#1112-common-types)
      - [1.11.2.1. State Errors](#11121-state-errors)
    - [1.11.3. state_call](#1113-state_call)
    - [1.11.4. state_getPairs](#1114-state_getpairs)
      - [1.11.4.1. Parameter](#11141-parameter)
      - [1.11.4.2. Response](#11142-response)
      - [1.11.4.3. Example](#11143-example)
    - [1.11.5. state_getKeysPaged](#1115-state_getkeyspaged)
      - [1.11.5.1. Parameter](#11151-parameter)
      - [1.11.5.2. Response](#11152-response)
      - [1.11.5.3. Example](#11153-example)
    - [1.11.6. state_getStorage](#1116-state_getstorage)
      - [1.11.6.1. Parameter](#11161-parameter)
      - [1.11.6.2. Response](#11162-response)
      - [1.11.6.3. Example](#11163-example)
    - [1.11.7. state_getStorageHash](#1117-state_getstoragehash)
      - [1.11.7.1. Parameter](#11171-parameter)
      - [1.11.7.2. Response](#11172-response)
      - [1.11.7.3. Example](#11173-example)
    - [1.11.8. state_getStorageSize](#1118-state_getstoragesize)
      - [1.11.8.1. Parameter](#11181-parameter)
      - [1.11.8.2. Response](#11182-response)
      - [1.11.8.3. Example](#11183-example)
    - [1.11.9. state_getMetadata](#1119-state_getmetadata)
      - [1.11.9.1. Parameter](#11191-parameter)
      - [1.11.9.2. Response](#11192-response)
      - [1.11.9.3. Example](#11193-example)
    - [1.11.10. state_getRuntimeVersion](#11110-state_getruntimeversion)
      - [1.11.10.1. Parameter](#111101-parameter)
      - [1.11.10.2. Response](#111102-response)
      - [1.11.10.3. Example](#111103-example)
    - [1.11.11. state_queryStorage](#11111-state_querystorage)
      - [1.11.11.1. Parameter](#111111-parameter)
      - [1.11.11.2. Response](#111112-response)
      - [1.11.11.3. Example](#111113-example)
    - [1.11.12. state_getReadProof](#11112-state_getreadproof)
      - [1.11.12.1. Parameter](#111121-parameter)
      - [1.11.12.2. Response](#111122-response)
      - [1.11.12.3. Example](#111123-example)
    - [1.11.13. state_subscribeRuntimeVersion (pubsub)](#11113-state_subscriberuntimeversion-pubsub)
      - [1.11.13.1. Parameter](#111131-parameter)
      - [1.11.13.2. Response](#111132-response)
      - [1.11.13.3. Example](#111133-example)
    - [1.11.14. state_unsubscribeRuntimeVersion (pubsub)](#11114-state_unsubscriberuntimeversion-pubsub)
      - [1.11.14.1. Parameter](#111141-parameter)
      - [1.11.14.2. Response](#111142-response)
      - [1.11.14.3. Example](#111143-example)
    - [1.11.15. state_subscribeStorage (pubsub)](#11115-state_subscribestorage-pubsub)
      - [1.11.15.1. Parameter](#111151-parameter)
      - [1.11.15.2. Response](#111152-response)
      - [1.11.15.3. Example](#111153-example)
    - [1.11.16. state_unsubscribeStorage (pubsub)](#11116-state_unsubscribestorage-pubsub)
      - [1.11.16.1. Parameter](#111161-parameter)
      - [1.11.16.2. Response](#111162-response)
      - [1.11.16.3. Example](#111163-example)
  - [1.12. Child State](#112-child-state)
    - [1.12.1. childstate_getKeys](#1121-childstate_getkeys)
      - [1.12.1.1. Parameter](#11211-parameter)
      - [1.12.1.2. Response](#11212-response)
    - [1.12.2. childstate_getStorage](#1122-childstate_getstorage)
      - [1.12.2.1. Parameter](#11221-parameter)
      - [1.12.2.2. Response](#11222-response)
    - [1.12.3. childstate_getStorageHash](#1123-childstate_getstoragehash)
      - [1.12.3.1. Parameter](#11231-parameter)
      - [1.12.3.2. Response](#11232-response)
    - [1.12.4. childstate_getStorageSize](#1124-childstate_getstoragesize)
      - [1.12.4.1. Parameter](#11241-parameter)
      - [1.12.4.2. Response](#11242-response)
  - [1.13. Engine](#113-engine)
    - [1.13.1. Engine Errors](#1131-engine-errors)
    - [1.13.2. engine_createBlock](#1132-engine_createblock)
    - [1.13.3. engine_finalizeBlock](#1133-engine_finalizeblock)
  - [1.14. Payment](#114-payment)
    - [1.14.1. Payment Errors](#1141-payment-errors)
    - [1.14.2. payment_queryInfo](#1142-payment_queryinfo)
  - [1.15. Contracts](#115-contracts)
    - [1.15.1. Contracts Errors](#1151-contracts-errors)
    - [1.15.2. contracts_call](#1152-contracts_call)
    - [1.15.3. contracts_getStorage](#1153-contracts_getstorage)
      - [1.15.3.1. Parameter](#11531-parameter)
      - [1.15.3.2. Response](#11532-response)
    - [1.15.4. contracts_rentProjection](#1154-contracts_rentprojection)
  - [1.16. BABE](#116-babe)
    - [1.16.1. BABE Errors](#1161-babe-errors)
    - [1.16.2. babe_epochAuthorship](#1162-babe_epochauthorship)
      - [1.16.2.1. Parameter](#11621-parameter)
      - [1.16.2.2. Response](#11622-response)
  - [1.17. Sync](#117-sync)
    - [1.17.1. sync_state_genSyncSpec](#1171-sync_state_gensyncspec)
      - [1.17.1.1. Parameter](#11711-parameter)
      - [1.17.1.2. Response](#11712-response)
      - [1.17.1.3. Example](#11713-example)
  - [1.18. Copyright](#118-copyright)

## 1.5. System

System RPC API.

### 1.5.1. System Errors

RPC error codes are in the `2000` - `2999` range. No known error types are specified.

### 1.5.2. system_name

Get the node's implementation name.

#### 1.5.2.1. Parameter

None.

#### 1.5.2.2. Response

`STRING` - The node's name.

#### 1.5.2.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_name", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "parity-polkadot",
    "id": 1
}
```

### 1.5.3. system_version

Get the node implementation's version. Should be a semver string.

#### 1.5.3.1. Parameter

None.

#### 1.5.3.2. Response

`STRING` - The node's version.

#### 1.5.3.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_version", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "0.7.20",
    "id": 1
}
```

### 1.5.4. system_chain

Get the chain's type. Given as a string identifier.

#### 1.5.4.1. Parameter

None.

#### 1.5.4.2. Response

`STRING` - The chain's name.

#### 1.5.4.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_chain", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "Kusama CC3",
    "id": 1
}
```

### 1.5.5. system_chainType

Get the chain's type.

#### 1.5.5.1. Parameter

None.

#### 1.5.5.2. Response

* `STRING` - The chain's type.

#### 1.5.5.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_chainType", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "Live",
    "id": 1
}
```

### 1.5.6. system_properties

Get a custom set of properties as a JSON object, defined in the chain specification.

#### 1.5.6.1. Parameter

None.

#### 1.5.6.2. Response

* `MAP` - (OPTIONAL)
  * `STRING`: `STRING` - Property name and value

#### 1.5.6.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_properties", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "ss58Format": 2,
        "tokenDecimals": 12,
        "tokenSymbol": "KSM"
    },
    "id": 1
}
```

### 1.5.7. system_health

Return health status of the node.

Node is considered healthy if it is:

* Connected to some peers (unless running in dev mode).
* Not performing a major sync.

#### 1.5.7.1. Parameter

None.

#### 1.5.7.2. Response

* `MAP`
  * `"isSyncing"`: `BOOL` - Whether the node is syncing.
  * `"peers"`: `U32` - Number of connected peers.
  * `"shouldHavePeers"`: `BOOL` - Should this node have any peers. Might be false for local chains or when running without discovery.

#### 1.5.7.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_health", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "isSyncing": false,
        "peers": 46,
        "shouldHavePeers": true
    },
    "id": 1
}
```

### 1.5.8. system_localPeerId

Returns the base58-encoded PeerId fo the node.

#### 1.5.8.1. Parameter

None.

#### 1.5.8.2. Response

* `STRING` - The base58 encoded PeerId.

#### 1.5.8.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_localPeerId", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "12D3KooWNU8nVAimt6DgxTKUqt4gQwZMJ6AWhjXCZbYbDEg5Wwg9",
    "id": 1
}
```

### 1.5.9. system_localListenAddresses

Returns the libp2p multiaddresses that the local node is listening on.

The addresses include a trailing `/p2p/` with the local PeerId, and are thus suitable to be passed to `system_addReservedPeer` or as a bootnode address for example.

#### 1.5.9.1. Parameter

None.

#### 1.5.9.2. Response

* `ARRAY`
  * `STRING` - A multiaddress.

#### 1.5.9.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_localListenAddresses", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": [
        "/ip4/127.0.0.1/tcp/30333/p2p/12D3KooWNU8nVAimt6DgxTKUqt4gQwZMJ6AWhjXCZbYbDEg5Wwg9"
    ],
    "id": 1
}
```

### 1.5.10. system_peers

Returns currently connected peers.

#### 1.5.10.1. Parameter

None.

#### 1.5.10.2. Response

* `ARRAY` - (OPTIONAL)
  * `MAP`
    * `"PeerId"`: `STRING` - Peer ID.
    * `"roles"`: `STRING` - Roles of the node. One of the following values is possible:
      * `"NONE"` - No network.
      * `"FULL"` - Full node, does not participate in consensus.
      * `"LIGHT"` - Light client node.
      * `"AUTHORITY"` - Acts as an authority.
    * `"protocolVersion"`: `U32` - Protocol version.
    * `"bestHash"`: `HEX` - Peer best block hash.
    * `"bestNumber"`: `U64` - Peer best block number.

#### 1.5.10.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_peers", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": [
        {
            "bestHash": "0x603b65f208656860f7d31f494896ac2ddcff37674442a84dfbcc1de0eacd83e2",
            "bestNumber": 1193270,
            "peerId": "QmTjJKhuLXKg9CfKkgGgJrzZ7LVfSmSUkQqFfZk1prF7LE",
            "protocolVersion": 6,
            "roles": "AUTHORITY"
        },
        {
            "bestHash": "0x603b65f208656860f7d31f494896ac2ddcff37674442a84dfbcc1de0eacd83e2",
            "bestNumber": 1193270,
            "peerId": "Qme89h5f5MkdN37R173z5GSJVBVSGeUriSrp4u3Y2ZRmUv",
            "protocolVersion": 6,
            "roles": "AUTHORITY"
        }
    ],
    "id": 1
}
```

### 1.5.11. system_networkState

*NOTE: This API is future-reserved, specification will be adjusted.*

### 1.5.12. system_addReservedPeer

**Warning**: This method is [UNSAFE](#Safety).

Adds a reserved peer. The string parameter should encode a `p2p` multiaddr.

`/ip4/198.51.100.19/tcp/30333/p2p/QmSk5HQbn6LhUwDiNMseVUjuRYhEtYj4aUZ6WfWoGURpdV` is an example of a valid, passing multiaddr with PeerId attached.

#### 1.5.12.1. Parameter

* `STRING` - Multiaddr to be added.

#### 1.5.12.2. Response

None.

#### 1.5.12.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_addReservedPeer", "params":["/ip4/198.51.100.19/tcp/30333/p2p/QmSk5HQbn6LhUwDiNMseVUjuRYhEtYj4aUZ6WfWoGURpdV"]}' http://localhost:9933
```

Response

```json
{
    "jsonrpc": "2.0",
    "result": null,
    "id": 1
}
```

### 1.5.13. system_removeReservedPeer

**Warning**: This method is [UNSAFE](#Safety).

Remove a reserved peer. The string should encode only the PeerId e.g. `QmSk5HQbn6LhUwDiNMseVUjuRYhEtYj4aUZ6WfWoGURpdV`.

#### 1.5.13.1. Parameter

* `STRING` - Peer ID to be removed.

#### 1.5.13.2. Response

None.

#### 1.5.13.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_removeReservedPeer", "params":["QmSk5HQbn6LhUwDiNMseVUjuRYhEtYj4aUZ6WfWoGURpdV"]}' http://localhost:9933
```

Response"

```json
{
    "jsonrpc": "2.0",
    "result": null,
    "id": 1
}
```

### 1.5.14. system_nodeRoles

Returns the roles the node is running as.

#### 1.5.14.1. Parameter

None.

#### 1.5.14.2. Response

* `ARRAY`
  * `STRING` - One of the following values is possible:
    * `"Full"` - The node is a full node.
    * `"LightClient"` - The node is a a light client.
    * `"Authority"` - The node is an authority.
    * `"UnknownRole"`: `ARRAY` - An unknown role followed by an arbitrary byte value.
      * `U8` - Undefined value.

#### 1.5.14.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_nodeRoles", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": [
        "Authority"
    ],
    "id": 1
}
```

### 1.5.15. system_syncState

Returns the state of the syncing of the node.

#### 1.5.15.1. Parameter

None.

#### 1.5.15.2. Response

* `MAP`
  * `"currentBlock"`: `U32` - The current best block number.
  * `"highestBlock"`: `U32` - The highest known block number.
  * `"startingBlock"`: `U32` - The starting block number.

#### 1.5.15.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_syncState", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "currentBlock": 2614956,
        "highestBlock": 2614956,
        "startingBlock": 2614925
    },
    "id": 1
}
```

### 1.5.16. system_accountNextIndex

Returns the next valid index (aka. nonce) for given account.

This method takes into consideration all pending transactions currently in the pool and if no transactions are found in the pool it fallbacks to query the index from the runtime (aka. state nonce).

Alias: `account_nextIndex`

#### 1.5.16.1. Parameter

* `STRING` - The address of the account.

#### 1.5.16.2. Response

* `U64` - The nonce of the account.

#### 1.5.16.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_accountNextIndex", "params":["FJaSzBUAJ1Nwa1u5TbKAFZG5MBtcUouTixdP7hAkmce2SDS"]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": 35,
    "id": 1
}
```

### 1.5.17. system_dryRun

Dry run an extrinsic. Returns a SCALE encoded `ApplyExtrinsicResult`.

#### 1.5.17.1. Parameter

* `HEX` - The raw, SCALE encoded extrinsic.
* `HASH` - The block hash indicating the state. `Null` implies the current state.

#### 1.5.17.2. Response

* `HEX` - The SCALE encoded `ApplyExtrinsicResult`.

## 1.6. Babe

Babe RPC API.

### 1.6.1. babe_epochAuthorship

Returns data about which slots (primary or secondary) can be claimed in the current epoch with the key in the keystore.

#### 1.6.1.1. Parameter

None.

#### 1.6.1.2. Response

* `MAP`
  * `STRING`: `MAP` - The Address of the authority.
    * `"primary"`: `ARRAY` - Primary slots that can be claimed.
      * `U64`
    * `"secondary"`: `ARRAY` - Secondary slots that can be claimed.
      * `U64`
    * `"secondary_vrf"`: `ARRAY` - Secondary VRF slots that can be claimed.
      * `U64`

#### 1.6.1.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "babe_epochAuthorship", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "1bsPcS69W1Mz3ZJimGn2dipzYh1S16bD2V4JsFVVP3u27tA": {
            "primary": [],
            "secondary": [],
            "secondary_vrf": [
                267645497,
                267646635,
                267646719,
                267647175,
                267647610
            ]
        }
    },
    "id": 1
}
```

## 1.7. Grandpa

Grandpa RPC API.

### 1.7.1. grandpa_roundState

Returns the state of the current best round state as well as the ongoing background rounds.

#### 1.7.1.1. Parameter

None.

#### 1.7.1.2. Response

* `MAP`
  * `"setId"`: `U32`
  * `"best"`: `RoundState`
  * `"background"`: `ARRAY`
    * `RoundState`

`RoundState` is defined as:

* `MAP`
  * `"round"`: `U32`
  * `"totalWeight"`: `U32`
  * `"thresholdWeight"`: `U32`
  * `"prevotes"`: `Votes`
  * `"precommits"`: `Votes`

`Votes` is defined as:

* `MAP`
  * `"currentWeight"`: `U32`
  * `"missing"`: `ARRAY`
    * `STRING` - The address of the authority.

#### 1.7.1.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "grandpa_roundState", "params":[]}' http://localhost:9933
```

Response (shortened):

```json
{
    "jsonrpc": "2.0",
    "result": {
        "background": [],
        "best": {
            "precommits": {
                "currentWeight": 0,
                "missing": [
                    "1342Tr5GgVmEFZB9EQh4CvBwZukc8b9wWkf1jWURRNx5DHd",
                    "1878HXjkN7yHw63iKqhW5d7Q7MfhceaS5sZmADBW32WRHLq",
                    "16gGqUnuCFPHHc6W34EznXe2FNcy6V8KzM9VymRF1EiKMKoG",
                    "16ist66RDhSo3DoTQ3scCSWPvMCrVeDMa7uqmMLvsGHGcVtz",
                    "16jY7XReiowJ2MwiF9nFKomCQPJfykwnBu3hqJM25FqYiyfC"
                ]
            },
            "prevotes": {
                "currentWeight": 0,
                "missing": [
                    "1342Tr5GgVmEFZB9EQh4CvBwZukc8b9wWkf1jWURRNx5DHd",
                    "1878HXjkN7yHw63iKqhW5d7Q7MfhceaS5sZmADBW32WRHLq",
                    "16gGqUnuCFPHHc6W34EznXe2FNcy6V8KzM9VymRF1EiKMKoG",
                    "16ist66RDhSo3DoTQ3scCSWPvMCrVeDMa7uqmMLvsGHGcVtz",
                    "16jY7XReiowJ2MwiF9nFKomCQPJfykwnBu3hqJM25FqYiyfC"
                ]
            },
            "round": 8329,
            "thresholdWeight": 157,
            "totalWeight": 234
        },
        "setId": 238
    },
    "id": 1
}
```

### 1.7.2. grandpa_proveFinality

Prove finality for the provided block range. Returns `NULL` if there are no known finalized blocks in the range. If no authorities set is provided, the current one will be attempted.

#### 1.7.2.1. Parameter

* `HASH` - The block hash indicating the beginning of the range.
* `HASH` - The block hash indicating the ending of the range.
* `U64` - (OPTIONAL) The Authority Set ID.

#### 1.7.2.2. Response

* `HEX` - (OPTIONAL) The SCALE encoded array of finality proofs.

#### 1.7.2.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "grandpa_proveFinality", "params":["0xd8c6bfd22432e1fdf2fbe4e6f32fb650b7bb5a9c4c992d53129bd64882f50290", "0xa324b88c18fb7109af79f03de4eee88bd6207376a085898c82d67b7d1a35ac41", 244]}' http://localhost:9933
```

Response (shortened):

```json
{
   "jsonrpc":"2.0",
   "result":"0x044e8e00c93905add716002173aa62feb3...83666edbb4da7ae3573c962f710f6e8500",
   "id":1
}
```

### 1.7.3. grandpa_subscribeJustifications (pubsub)

Returns the block most recently finalized by Grandpa, alongside side its justification.

This endpoints communicates over the Websocket protocol (`grandpa_justifications` subscription).

#### 1.7.3.1. Parameter

None.

#### 1.7.3.2. Response

* `HEX` - The block itself.

#### 1.7.3.3. Example

Request:

```json
{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "grandpa_subscribeJustifications",
    "params": []
}
```

Response (shortened):

```json
{
   "jsonrpc":"2.0",
   "method":"grandpa_justifications",
   "params":{
      "result":"0xa01600000...0df166000",
      "subscription":"E6lixkMvbRHl4xf0"
   }
}
```

### 1.7.4. grandpa_unsubscribeJustifications (pubsub)

Unsubscribe from justification watching.

#### 1.7.4.1. Parameter

* `STRING` or `U32` - The subscriber ID, depending on subscription initialization.

#### 1.7.4.2. Response

* `BOOL` - `true` if the subscriber was unsubscribed, `false` if there was no such provided subscriber.

#### 1.7.4.3. Example

Request:

```json
{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "grandpa_unsubscribeJustifications",
    "params": [
        "FO4PWcGOGLxuESDw"
    ]
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

## 1.8. Author

Authoring RPC API.

### 1.8.1. Common types

#### 1.8.1.1. Key types

Polkadot separates keys into different types, depending on its use-case. Following variants are available.

* `"babe"` - Babe functionality.
* `"gran"` - GRANDPA functionality.
* `"acco"` - Controlling an account.
* `"aura"` - Aura functionality.
* `"imon"` - *ImOnline* functionality.
* `"audi"` - Authority discovery functionality.
* `"dumy"` - Useful for testing.

#### 1.8.1.2. Author Errors

RPC error codes are in the `1000` - `1999` range. The following known error types are possible:

* *BadFormat*
  * Occurrence: incorrect extrinsic format.
  * RPC code: 1001
  * RPC message: "Extrinsic has invalid format: ${details}"
    * *${details}*: details of the error message.
* *Verification*
  * Occurrence: verification error.
  * RPC code: 1002
  * RPC message: "Extrinsic verification error: ${details}"
    * *${details}*: details of the error message.
* *Pool*
  * *InvalidTransaction*
    * Occurrence: the transaction is invalid.
    * RPC code: 1010
    * RPC message: "Invalid Transaction"
  * *UnknownTransaction*
    * Occurrence: the transaction is unknown.
    * RPC code: 1011
    * RPC message: "Unknown Transaction Validity"
  * *TemporarilyBanned*
    * Occurrence: the transaction is temporarily banned.
    * RPC code: 1012
    * RPC message: "Transaction is temporarily banned"
  * *AlreadyImported*
    * Occurrence: the transaction was already imported.
    * RPC code: 1013
    * RPC message: "Transaction Already Imported"
  * *TooLowPriority*
    * Occurrence: the transaction has too low priority.
    * RPC code: 1014
    * RPC message: "The transaction has too low priority to replace another transaction already in the pool"
  * *CycleDetected*
    * Occurrence: a cycle has been detected.
    * RPC code: 1015
    * RPC message: "Cycle Detected"
  * *ImmediatelyDropped*
    * Occurrence: transaction pool limit reached.
    * RPC code: 1016
    * RPC message: "The transaction couldn't enter the pool because of the limit"
* *UnsupportedKeyType*
  * Occurrence: key type ID has some unsupported crypto.
  * RPC code: 1017
  * RPC message: "The crypto for the given key type is unknown, please add the public key to the request to insert the key successfully"

### 1.8.2. author_submitExtrinsic

**Warning**: This method is [UNSAFE](#Safety).

Submit an extrinsic for inclusion into a block.

#### 1.8.2.1. Parameter

* `HEX` - A SCALE encoded extrinsic.

#### 1.8.2.2. Response

* `HASH` - The resulting transaction hash of the extrinsic.

### 1.8.3. author_pendingExtrinsics

Returns all pending extrinsics, potentially grouped by sender.

#### 1.8.3.1. Parameter

None.

#### 1.8.3.2. Response

* `ARRAY`
  * `HEX` - (OPTIONAL) SCALE encoded extrinsic.

### 1.8.4. author_removeExtrinsic

**Warning**: This method is [UNSAFE](#Safety).

Remove given extrinsic from the pool and temporarily ban it to prevent reimporting.

#### 1.8.4.1. Parameter

Either one of the following types:
* `ARRAY`
  * `HASH` - The hash of the extrinsic to be removed.
* `HASH` - The hash of the extrinsic to be removed.

#### 1.8.4.2. Response

* `ARRAY` - (OPTIONAL)
  * `HEX` - The extrinsic that was removed.

### 1.8.5. author_insertKey

**Warning**: This method is [UNSAFE](#Safety).

Insert a key into the keystore.

#### 1.8.5.1. Parameter

* `STRING` * [Key type](#Key-types).
* `HEX` - The seed.
* `HEX` - The public key.

#### 1.8.5.2. Response

None.

#### 1.8.5.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_insertKey", "params":["dumy", "0x3fb882f70b4ddf5f8923f4a2d3b30a20f79bc0c5de212c1a8977f4972272db8d", "0x5ebf69cfbb4914711f70ff3b9e7455f7d5006b15f220d011387038cf4fb1593e"]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": null,
    "id": 1
}
```

### 1.8.6. author_rotateKeys

**Warning**: This method is [UNSAFE](#Safety).

Generate new session keys and returns the corresponding public keys.

#### 1.8.6.1. Parameter

None.

#### 1.8.6.2. Response

* `HEX` - The SCALE encoded, concatenated keys.

#### 1.8.6.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "0x8c0baa0b4cf42e669a3805c1d6405926c9adf0691f854a6ddaffde3abc1dbd6b7c70cc7d2a731186ec54e26e0b0509667376d818643a5969549e44a76dc42f5a041b0120d2fc6d686e1bec66f596ddcce78da4029a23b3c213c55d2a064e9c26a20ab338080835b845e71573c3197795b729a1015b504a8352ee7dcbce92296c28bcc393e3cb1b18f597c597a458e21f706374e75f35445132977a66519d731d",
    "id": 1
}
```

### 1.8.7. author_hasSessionKeys

Checks if the keystore has private keys for the given session public keys.

#### 1.8.7.1. Parameter

* `HEX` - The SCALE encoded, concatenated keys.

#### 1.8.7.2. Response

* `BOOL` - Returns `true` if all private keys could be found, `false` if otherwise.

#### 1.8.7.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_hasSessionKeys", "params":["0x8c0baa0b4cf42e669a3805c1d6405926c9adf0691f854a6ddaffde3abc1dbd6b7c70cc7d2a731186ec54e26e0b0509667376d818643a5969549e44a76dc42f5a041b0120d2fc6d686e1bec66f596ddcce78da4029a23b3c213c55d2a064e9c26a20ab338080835b845e71573c3197795b729a1015b504a8352ee7dcbce92296c28bcc393e3cb1b18f597c597a458e21f706374e75f35445132977a66519d731d"]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

### 1.8.8. author_hasKey

Checks if the keystore has private keys for the given public key and key type.

#### 1.8.8.1. Parameter

* `HEX` - The public key.
* `STRING` * [Key type](#Key-types).

#### 1.8.8.2. Response

* `BOOL` - Returns `true` if a private key could be found, `false` if otherwise.

#### 1.8.8.3. Example

Request:

```bash
// Request
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_hasKey", "params":["0x5ebf69cfbb4914711f70ff3b9e7455f7d5006b15f220d011387038cf4fb1593e", "dumy"]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

### 1.8.9. author_submitAndWatchExtrinsic (pubsub)

Submit an extrinsic and watch.

This endpoint communicates over the Websocket protocol (`author_extrinsicUpdate` subscription).

#### 1.8.9.1. Parameter

* `HEX` - The SCALE encoded extrinsic.

#### 1.8.9.2. Response

Return **one** of the following extrinsic statuses:

* `"future"` - The extrinsic is part of the future queue.
* `"ready"` - The extrinsic is part of the ready queue.
* `MAP` - The extrinsic has been broadcast to the given peers.
  * `"broadcast"`: `ARRAY`
    * `STRING` - The PeerId.
* `MAP` - The extrinsic has been included in block with given hash.
  * `"inBlock"`: `HASH` - The hash of the block.
* `MAP` - The block this extrinsic was included in has been retracted.
  * `"retracted"`: `HASH` - The hash of the block.
* `MAP` - Maximum number of finality watchers has been reached, old watches are being removed.
  * `"finalityTimeout"`: `HASH` - The hash of the block.
* `MAP` - The extrinsic has been finalized by GRANDPA.
  * `"finalized"`: `HASH` - The hash of the block.
* `MAP` - The extrinsic has been replaced in the pool, by another extrinsic that provides the same tags (e.g. same sender/nonce).
  * `"usurped"`: `HASH` - The hash of the extrinsic.
* `"dropped"` - The extrinsic has been dropped from the pool because of the limit.
* `"invalid"` - The extrinsic is no longer valid in the current state.

#### 1.8.9.3. Example

Request:

```bash
{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "author_submitAndWatchExtrinsic",
    "params": [
        "0x01d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000bae32a8130ab7966a82a8a025e24d23f0244e606f34375f66855b105d1d2e25eca2e21855ba44e90bd48833638220a4a9ddd1b6ffa08a2424df1a8ffbd8b0d8f00"
    ]
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "method": "author_extrinsicUpdate",
    "params": {
        "result": "ready",
        "subscription": 1
    }
}

{
    "jsonrpc": "2.0",
    "method": "author_extrinsicUpdate",
    "params": {
        "result": {
            "usurped": "0x01d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d0000000000000000000000000000000000000000000000000000000000000000050000000000000000000000000000001a19b6a160cd4f6ba18f9432255c018252e79759248b57e8e907ba3e87642e287581e10a1ced3004831908bb31d16740d6d35b536204540bd82fa23ebfd2608b00"
        },
        "subscription": 1
    }
}
```

### 1.8.10. author_unwatchExtrinsic (pubsub)

Unsubscribe extrinsic watching.

#### 1.8.10.1. Parameter

* `STRING` or `U32` - The subscriber ID, depending on subscription initialization.

#### 1.8.10.2. Response

* `BOOL` - `true` if the subscriber was unsubscribed, `false` if there was no such provided subscriber.

#### 1.8.10.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "author_unwatchExtrinsic",
    "params": [
        10
    ],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

## 1.9. Chain

Blockchain RPC API.

### 1.9.1. Chain Errors

RPC error codes are in the `3000` - `3999` range. No known error types are specified.

### 1.9.2. chain_getHeader

Get header of a relay chain block. If no block hash is provided, the latest block header will be returned.

#### 1.9.2.1. Parameter

* `HASH` - (OPTIONAL) Hex encoded block hash.

#### 1.9.2.2. Response

* `MAP` - (OPTIONAL).
  * `"parentHash"`: `HEX` - The parent hash.
  * `"number"`: `HEX` - The block number.
  * `"stateRoot"`: `HASH` - The state trie merkle root.
  * `"extrinsicsRoot"`: `HASH` - The merkle root of the extrinsics.
  * `"digest"`: `MAP` - Chain-specific digest of data useful for light clients or auxiliary data.
    * `"logs"`: `ARRAY` - A list of logs in the digest.
      * `HEX` - Digest logs of opaque nature.

#### 1.9.2.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "chain_getHeader", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "digest": {
            "logs": [
                "0x064241424534020b000000497fb90f00000000",
                "0x05424142450101f4b34c2a153dc4912c859ac9542c4345911ed0c1cae18c9704d6566f2689ac158681adfd0410a9e1c97a50b0c455af1b4d5447cbd4c4da1a6ebcd53ee5d3ee89"
            ]
        },
        "extrinsicsRoot": "0x01f31561613c605d96d0572b7dda3884ccc2f9f890538974007eb610784216d9",
        "number": "0x12d2a9",
        "parentHash": "0x281270fff803f40aad2f5e6bc113f6814060cc2f435bf6e4526795ec9d73a62d",
        "stateRoot": "0xc51f09252a24f7965c892a4f57d5ec5985830ac90b32da0a045b92457ebc5b39"
    },
    "id": 1
}
```

### 1.9.3. chain_getBlock

Get header and body of a relay chain block. If no block hash is provided, the latest block body will be returned.

#### 1.9.3.1. Parameter

* `HASH` - (OPTIONAL) The block hash.

#### 1.9.3.2. Response

* `MAP` - (OPTIONAL)
  * `"block"`: `MAP`
    * `"extrinsics"`: `ARRAY`
      * `HEX` - An extrinsic.
    * `"header"`: `MAP`
      * `"parentHash"`: `HASH` - The parent hash.
      * `"number"`: `HEX` - The block number.
      * `"stateRoot"`: `HASH` - The state trie merkle root.
      * `"extrinsicsRoot"`: `HASH` - The merkle root of the extrinsics.
      * `"digest"`: `MAP` - Chain-specific digest of data useful for light clients or auxiliary data.
        * `"logs"`: `ARRAY` - A list of logs in the digest.
          * `HEX` - Digest logs of opaque nature.
  * `"justification"`: `HEX` - (OPTIONAL) The justification.

#### 1.9.3.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "chain_getBlock", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "block": {
            "extrinsics": [
                "0x280402000bb0019c8b7001",
                "0x1c0409006a4b4b00",
                "0x1004140000"
            ],
            "header": {
                "digest": {
                    "logs": [
                        "0x06424142453402610000007d7fb90f00000000",
                        "0x0542414245010166b01406e966b6604ea996cda5355c5f24fffee6cb2082802105f136b456c9412855a0cce4058f08ccf9ce822cd94a6ae786037a8566310765a043a09514488a"
                    ]
                },
                "extrinsicsRoot": "0x1ce1236c8262cead39a2a758fc4cdba0b918f309e5a8572c288c5ba2c16aa7a7",
                "number": "0x12d2dd",
                "parentHash": "0x78b7af6942c220ec33945f228f1e17383403561f8d7bbc02da2c79650f22fdbe",
                "stateRoot": "0xa6387842acac9315ad4424828f571c348a40bdfb9f9b4cb4951bcb76e43dacfb"
            }
        },
        "justification": null
    },
    "id": 1
}
```

### 1.9.4. chain_getBlockHash

Get hash of the 'n-th' block in the canon chain. If no parameters are provided, the latest block hash gets returned.

Alias: `chain_getHead`

#### 1.9.4.1. Parameter

One of the following types is allowed:

* `HEX` - (OPTIONAL) The value indicating the 'n-th' block in the chain.
* `U32` - (OPTIONAL) The value indicating the 'n-th' block in the chain.
* `ARRAY` - (OPTIONAL) Multiple values, where each value can be one of the following types:
  * `HEX` - The value indicating the 'n-th' block in the chain.
  * `U32` - The value indicating the 'n-th' block in the chain.

#### 1.9.4.2. Response

One of the following types gets returned, depending on the provided parameters:

* `HASH` - (OPTIONAL) The block hash.
* `ARRAY`
  * `HASH` - (OPTIONAL) A block hash

#### 1.9.4.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "chain_getBlockHash", "params":[[50, "0x64", 200]]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": [
        "0x4fee9b1803132954978652e4d73d4ec5b0dffae3832449cd5e4e4081d539aa22",
        "0x46781d9a3350a0e02dbea4b5e7aee7c139331a65b2cd736bb45a824c2f3ffd1a",
        "0x0f82403bcd4f7d4d23ce04775d112cd5dede13633924de6cb048d2676e322950"
    ],
    "id": 1
}
```

### 1.9.5. chain_getFinalizedHead

Get hash of the last finalized block in the canon chain.

Alias: `chain_getFinalisedHead`

#### 1.9.5.1. Parameter

None.

#### 1.9.5.2. Response

* `HASH` - The hash of the last finalized block.

#### 1.9.5.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "chain_getFinalizedHead", "params":[]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "0x00b8360db070d20fb2bc700a73240c65acb590d26ae8c477b0add48a3695bd35",
    "id": 1
}
```

### 1.9.6. chain_subscribeAllHeads (pubsub)

Subscription for all block headers (new blocks and finalized blocks).

This endpoint communicates over the Websocket protocol (`chain_allHead` subscription).

#### 1.9.6.1. Parameter

None.

#### 1.9.6.2. Response

* `MAP`
  * `"digest"`: `MAP`
    * `"logs"`: `ARRAY`
      * `HEX` - Digest item.
  * `"extrinsicsRoot"`: `HASH` - The merkle root of extrinsics.
  * `"number"`: `HEX` - The block number.
  * `"parentHash"`: `HASH` - The parent hash.
  * `"stateRoot"`: `HASH` - The state root.

#### 1.9.6.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "chain_subscribeAllHeads",
    "params": [],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "method": "chain_allHead",
    "params": {
        "result": {
            "digest": {
                "logs": [
                    "0x0642414245b50101a300000082accc0f0000000056b7c762126decd0f4ea57564d468873831e92905fdd0b398c53240bb9df7e642b47b8af28e025257219acaa4c5065daa9e48d23451a0e046aba07be1666080de5889c4e698919bb735905ceafcb6d0af2d64377ad7a7dd6c0c85e3a6009780f",
                    "0x054241424501015e0cf4e51a0395d88303ccdaeefec4c5384f7f8fa7dbd1dc57503b9d344c436891c9140baa0f65060f1f0a9e192fd4e8268475f8caeae7ce13490f34c9e5678d"
                ]
            },
            "extrinsicsRoot": "0x165cb92127cd5eef85ace0af1dbefd952b913858909b9cc4b35fa33d457a7c9c",
            "number": "0x25997e",
            "parentHash": "0xa777e05033d255b84180e96a0d1b0b04b7d118969b4d60f78750900278ed4774",
            "stateRoot": "0xd330701dc41b9c2c97cbb4cfaed3e59e6f94e3d896e4d9c904235c657091175b"
        },
        "subscription": 1
    }
}
```

### 1.9.7. chain_unsubscribeAllHeads (pubsub)

Unsubscribe from watching all block headers.

#### 1.9.7.1. Parameter

* `STRING` or `U32` - The subscriber ID, depending on subscription initialization.

#### 1.9.7.2. Response

* `BOOL` - `true` if the subscriber was unsubscribed, `false` if there was no subscriber.

#### 1.9.7.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "chain_unsubscribeAllHeads",
    "params": [
        3
    ],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

### 1.9.8. chain_subscribeNewHeads (pubsub)

Subscription for new block headers.

This endpoint communicates over the Websocket protocol (`chain_newHead` subscription).

#### 1.9.8.1. Parameter

None.

#### 1.9.8.2. Response

* `MAP`
  * `"digest"`: `MAP`
    * `"logs"`: `ARRAY`
      * `HEX` - A digest item.
  * `"extrinsicsRoot"`: `HASH` - The merkle root of extrinsics.
  * `"number"`: `HEX` - The block number.
  * `"parentHash"`: `HASH` - The parent hash.
  * `"stateRoot"`: `HASH` - The state root.

#### 1.9.8.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "chain_subscribeNewHeads",
    "params": [],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "method": "chain_newHead",
    "params": {
        "result": {
            "digest": {
                "logs": [
                    "0x06424142453402be00000024adcc0f00000000",
                    "0x05424142450101eec7df1cd6e522d9e753a971f25bffd07646632a2cccc24ab8d6ca08d4313f0c8afc9fd4ec741ad677eeb373ad66c5a55e31f7f74d76d83d10763c30357f2883"
                ]
            },
            "extrinsicsRoot": "0xe83be686dc673d81624e105b4def1773631ba9b5d0bb21aff683cfe402c092d9",
            "number": "0x259a1e",
            "parentHash": "0x2bdf3aea35b4fbc9fae4a001821973d872bf7fae9acfd7d15deb86b7a31ccba3",
            "stateRoot": "0x1735078a466512f5c1275d13e5f2634e00b505aee5d673a4fd167630bf108a99"
        },
        "subscription": 2
    }
}
```

### 1.9.9. chain_unsubscribeNewHeads (pubsub)

Unsubscribe from watching new block headers.

#### 1.9.9.1. Parameter

* `STRING` or `U32` - The subscriber ID, depending on subscription initialization.

#### 1.9.9.2. Response

* `BOOL` - `true` if the subscriber was unsubscribed, `false` if there was no subscriber.

#### 1.9.9.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "chain_unsubscribeNewHeads",
    "params": [
        7
    ],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

### 1.9.10. chain_subscribeFinalizedHeads (pubsub)

Subscription for finalized block headers.

This endpoint communicates over the Websocket protocol (`chain_finalizedHead` subscription).

#### 1.9.10.1. Parameter

None.

#### 1.9.10.2. Response

* `MAP`
  * `"digest"`: `MAP`
    * `"logs"`: `ARRAY`
      * `HEX` - A digest item.
  * `"extrinsicsRoot"`: `HASH` - The merkle root of extrinsics.
  * `"number"`: `HEX` - The block number.
  * `"parentHash"`: `HASH` - The parent hash.
  * `"stateRoot"`: `HASH` - The state root.

#### 1.9.10.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "chain_subscribeFinalizedHeads",
    "params": [],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "method": "chain_finalizedHead",
    "params": {
        "result": {
            "digest": {
                "logs": [
                    "0x0642414245340251000000d2adcc0f00000000",
                    "0x05424142450101c26bd718dc93364f14911ad1a9f946238b34ecc38723e00f3819a0440ad4c331a4a40009fffd5364998bbbcd85cec755156c09e45a36276ee0317c4720396385"
                ]
            },
            "extrinsicsRoot": "0xbd09c7fc3a39c11cd87992eb29c263c84f71c3bf42134a14834bc0b70ca69d02",
            "number": "0x259acc",
            "parentHash": "0xe06552be3f8126a6bc3c8bf5f6a7541f357eb7be86f0be48198356dd58c723c8",
            "stateRoot": "0x8f6fd73fb852009bfcce5df4b769f77a42eb401c545ba17e3fd183ac4decb5cf"
        },
        "subscription": 3
    }
}
```

### 1.9.11. chain_unsubscribeFinalizedHeads (pubsub)

Unsubscribe from watching finalized block headers.

#### 1.9.11.1. Parameter

* `STRING` or `U32` - The subscriber ID, depending on subscription initialization.

#### 1.9.11.2. Response

* `BOOL` - `true` if the subscriber was unsubscribed, `false` if there was no subscriber.

#### 1.9.11.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "chain_unsubscribeFinalizedHeads",
    "params": [
        5
    ],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

## 1.10. Offchain

Blockchain RPC API.

### 1.10.1. Offchain Errors

RPC error codes are in the `5000` - `5999` range. No known error types are specified.

### 1.10.2. Storage kinds

The following Storage kinds are available:

* "PERSISTENT" - is non-revertible and not fork-aware. It means that any value set by the offchain worker is persisted even if that block (at which the worker is called) is reverted as non-canonical (meaning that the block was surpassed by a longer chain). The value is available for the worker that is re-run at the new (different block with the same block number) and future blocks. This storage can be used by offchain workers to handle forks and coordinate offchain workers running on different forks.
* "LOCAL" - is revertible and fork-aware. It means that any value set by the offchain worker triggered at a certain block is reverted if that block is reverted as non-canonical. The value is NOT available for the worker that is re-run at the next or any future blocks.

### 1.10.3. offchain_localStorageSet

Set offchain local storage under given key and prefix.

#### 1.10.3.1. Parameter

* `STRING` - [Storage kind](#Storage-kinds).
* `HEX` - The key.
* `HEX` - The value.

#### 1.10.3.2. Response

None.

#### 1.10.3.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "offchain_localStorageSet", "params":["PERSISTENT","0x4B6579","0x56616C7565"]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": null,
    "id": 1
}
```

### 1.10.4. offchain_localStorageGet

Get offchain local storage under given key and prefix.

#### 1.10.4.1. Parameter

* `STRING` - [Storage kind](#Storage-kinds).
* `HEX` - The key.

#### 1.10.4.2. Response

* `HEX` - (OPTIONAL) The value.

#### 1.10.4.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "offchain_localStorageGet", "params":["PERSISTENT","0x4B6579"]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "0x56616c7565",
    "id": 1
}
```

## 1.11. State

State RPC API.

### 1.11.1. State Errors

RPC error codes are in the `4000` - `4999` range. No known error types are specified.

### 1.11.2. Common types

#### 1.11.2.1. State Errors

*NOTE: This type is future-reserved, specification will be adjusted.*

### 1.11.3. state_call

*NOTE: This API is future-reserved, specification will be adjusted.*

Call a contract at a block's state.

### 1.11.4. state_getPairs

**Warning**: This method is [UNSAFE](#Safety).

Returns the keys with prefix, leave empty to get all the keys.

#### 1.11.4.1. Parameter

* `HEX` - The prefix.
* `HEX` - (OPTIONAL) The block hash.

#### 1.11.4.2. Response

* `ARRAY`
  * `ARRAY` - (OPTIONAL)
    * `HEX` - The key.
    * `HEX` - The value.

#### 1.11.4.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getPairs", "params":["0x", null]}' http://localhost:9933
```

Response (shortened):

```json
{
    "jsonrpc": "2.0",
    "result": [
        [
            "0x0b76934f4cc08dee01012d059e1b83eebbd108c4899964f707fdaffb82636065",
            "0x00"
        ],
        [
            "0x11f3ba2e1cdd6d62f2ff9b5589e7ff816254e9d55588784fa2a62b726696e2b1",
            "0x7b000000"
        ]
    ],
    "id": 1
}
```

### 1.11.5. state_getKeysPaged

**Warning**: This method is [UNSAFE](#Safety).

Returns the keys with prefix with pagination support.

#### 1.11.5.1. Parameter

* `HEX` - (OPTIONAL) The prefix.
* `U32` - Amount of keys to be return.
* `HEX` - (OPTIONAL) The storage key after which the next keys should be returned in lexicographic order.
* `HASH` - (OPTIONAL) The block hash.

#### 1.11.5.2. Response

* `ARRAY`
  * `HEX` - (OPTIONAL) Storage key

#### 1.11.5.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getKeysPaged", "params":[null, 2]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": [
        "0x0b76934f4cc08dee01012d059e1b83ee5e0621c4869aa60c02be9adcc98a0d1d",
        "0x0b76934f4cc08dee01012d059e1b83eebbd108c4899964f707fdaffb82636065"
    ],
    "id": 1
}
```

### 1.11.6. state_getStorage

Returns a storage entry at a specific block's state. If not block hash is provided, the latest value is returned.

#### 1.11.6.1. Parameter

* `HEX` - The storage key.
* `HASH` - (OPTIONAL) The block hash.

#### 1.11.6.2. Response

* `HEX` - (OPTIONAL) The storage value.

#### 1.11.6.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getStorage", "params":["0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b458ad08561bd8f502d2ba488697d10b58aaa7c4097d4abb1c8861495348fd6970", null]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "0x00e40b54020000000000000000000000",
    "id": 1
}
```

### 1.11.7. state_getStorageHash

Returns the hash of a storage entry at a block's state. If no block hash is provided, the latest value is returned.

#### 1.11.7.1. Parameter

* `HEX` - The storage key
* `HASH` - (OPTIONAL) The block hash.

#### 1.11.7.2. Response

* `HEX` - (OPTIONAL) The hash of the storage entry.

#### 1.11.7.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getStorageHash", "params":["0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b458ad08561bd8f502d2ba488697d10b58aaa7c4097d4abb1c8861495348fd6970", "0x579deccea7183c2afedbdaea59ad23e970458186afc4d57d5577842d4a219925"]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": "0x7155ca82321b189fc7b9009d9a0c2ebffba28aad222e28986543f266a65064eb",
    "id": 1
}
```

### 1.11.8. state_getStorageSize

Returns the size of a storage entry at a block's state. If no block hash is provided, the latest value is used.

#### 1.11.8.1. Parameter

* `HEX` - The storage key.
* `HASH` - (OPTIONAL) The block hash.

#### 1.11.8.2. Response

* `U64` - (OPTIONAL) The size of the storage entry in bytes.

#### 1.11.8.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getStorageSize", "params":["0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b458ad08561bd8f502d2ba488697d10b58aaa7c4097d4abb1c8861495348fd6970", null]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": 16,
    "id": 1
}
```

### 1.11.9. state_getMetadata

Returns the runtime metadata.

#### 1.11.9.1. Parameter

* `HASH` - (OPTIONAL) The block hash indicating the state. `NULL` implies the current state.

#### 1.11.9.2. Response

* `HEX` - The runtime metadata as an opaque blob.

#### 1.11.9.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getMetadata", "params":[null]}' http://localhost:9933
```

Response (shortened):

```json
{
    "jsonrpc": "2.0",
    "result": "0x6d6574610c701853797374656...c696461746541747465737473",
    "id": 1
}
```

### 1.11.10. state_getRuntimeVersion

Get the runtime version at a given block. If no block hash is provided, the latest version gets returned.

#### 1.11.10.1. Parameter

* `HEX` - (OPTIONAL) The block hash.

#### 1.11.10.2. Response

* `MAP`
  * `"specName"`: `STRING` - Name of the runtime.
  * `"implName"`: `STRING` - Name of the implementation of the specification.
  * `"authoringVersion"`: `U32` - Version of the authorship interface.
  * `"specVersion"`: `U32` - Version of the runtime specification.
  * `"implVersion"`: `U32` - Version of the implementation of the specification.
  * `"apis"`: `ARRAY` - List of supported API features along with their version.
    * `ARRAY` - (OPTIONAL)
      * `HEX` - The feature name.
      * `U32` - Version of the feature.

#### 1.11.10.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getRuntimeVersion", "params":[]}' http://localhost:9933
```

Response (shortened):

```json
{
    "jsonrpc": "2.0",
    "result": {
        "apis": [
            [
                "0xdf6acb689907609b",
                2
            ],
            [
                "0x37e397fc7c91f5e4",
                1
            ]
        ],
        "authoringVersion": 2,
        "implName": "parity-kusama",
        "implVersion": 0,
        "specName": "kusama",
        "specVersion": 1045
    },
    "id": 1
}
```

### 1.11.11. state_queryStorage

**Warning**: This method is [UNSAFE](#Safety).

Query historical storage entries (by key) starting from a block given as the second parameter.

**Note**: This first returned result contains the initial state of storage for all keys. Subsequent values in the array represent changes to the previous state (diffs).

#### 1.11.11.1. Parameter

* `ARRAY` - List of storage keys to query.
  * `HEX` - The storage key.
* `HEX` - The block hash from where to start the query.
* `HASH` - (OPTIONAL) The block hash to where to end the query

#### 1.11.11.2. Response

* `ARRAY`
  * `MAP` - (OPTIONAL)
    * `"block"`: `HASH` - The block hash at which a change occurred.
    * `"changes"`: `ARRAY` - Mappings between keys and their corresponding (changed) values.
      * `ARRAY`
        * `HEX` - The storage key.
        * `HEX` - The value.

#### 1.11.11.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_queryStorage", "params":[["0xf2794c22e353e9a839f12faab03a911bf68967d635641a7087e53f2bff1ecad3c6756fee45ec79ead60347fffb770bcdf0ec74da701ab3d6495986fe1ecc3027"], "0xa32c60dee8647b07435ae7583eb35cee606209a595718562dd4a486a07b6de15", null]}' http://localhost:9933
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": [
        {
            "block": "0xe99c87d6efab57a98706c10fa4bc4d39feaf51ca62ac3bae7a67bf17c8c305ec",
            "changes": [
                [
                    "0xf2794c22e353e9a839f12faab03a911bf68967d635641a7087e53f2bff1ecad3c6756fee45ec79ead60347fffb770bcdf0ec74da701ab3d6495986fe1ecc3027",
                    "0xa80400fffe112179ba7e7a1549e6c45522901ab7ef7ee373c1797b0f58191c6b53c7831c0b00a0724e1809603153efe4c60f146d61b66d5c9f4a9b469291aa260899bd99083d755a28923d00ea56fa000000000000000000000000490f0600"
                ]
            ]
        }
    ],
    "id": 1
}
```

### 1.11.12. state_getReadProof

Returns the proof of storage entries.

#### 1.11.12.1. Parameter

* `ARRAY`
  * `HEX` - The storage key.
* `HASH` - (OPTIONAL) The block hash indicating the sate. `NULL` implies the current state.

#### 1.11.12.2. Response

* `MAP`
  * `"at"`: `HASH` - The block has used to generate the proof.
  * `"proof"`: `ARRAY`
    * `HEX` - The proofs of the storage entries.

#### 1.11.12.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getReadProof", "params":[["0x1a736d37504c2e3fb73dad160c55b2918ee7418a6531173d60d1f6a82d8f4d51c16ee72ac33a6a9e5e887792c26526f9cc080000"], null]}' http://localhost:9933
```

Response (shortened):

```json
{
   "jsonrpc":"2.0",
   "result":{
      "at":"0x4f6b72413a242fb63b3637d2102d3f5d64319d9f9dd60cdc39aade2a093dfab5",
      "proof":[
         "0x650ee72ac33a6a9e5e887792c26526f9cc080000c4664dbd21a50bec286ed2ae25da8f41634778154b3ae6dbd93290bcae58f1dd600000000000000000000000000000000000",
         "0x80801080126002ea89025d2ff7332e6e13b01319e19154993e60db82ac22ce1c7084e19980fe72d495f75a1317c844a2bd7d3ec5c99ac7acf5d8e61096b8d2fe6152574ef2"
      ]
   },
   "id":1
}
```

### 1.11.13. state_subscribeRuntimeVersion (pubsub)

Runtime version subscription. Creates a message for current version and each upgrade.

This endpoint communicates over the Websocket protocol (`state_runtimeVersion` subscription).

#### 1.11.13.1. Parameter

None.

#### 1.11.13.2. Response

* `MAP`
  * `"specName"`: `STRING` - Name of the runtime.
  * `"implName"`: `STRING` - Name of the implementation of the specification.
  * `"authoringVersion"`: `U32` - Version of the authorship interface.
  * `"specVersion"`: `U32` - Version of the runtime specification.
  * `"implVersion"`: `U32` - Version of the implementation of the specification.
  * `"apis"`: `ARRAY` - List of supported API features along with their version.
    * `ARRAY` - (OPTIONAL)
      * `HEX` - The feature name.
      * `U32` - Version of the feature.

#### 1.11.13.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "state_subscribeRuntimeVersion",
    "params": [],
    "id": 1
}
```

Response (shortened):

```json
{
    "jsonrpc": "2.0",
    "method": "state_runtimeVersion",
    "params": {
        "result": {
            "apis": [
                [
                    "0xdf6acb689907609b",
                    3
                ],
                [
                    "0x37e397fc7c91f5e4",
                    1
                ]
            ],
            "authoringVersion": 2,
            "implName": "parity-kusama",
            "implVersion": 0,
            "specName": "kusama",
            "specVersion": 1062,
            "transactionVersion": 1
        },
        "subscription": 4
    }
}
```

### 1.11.14. state_unsubscribeRuntimeVersion (pubsub)

Unsubscribe from watching the runtime version.

#### 1.11.14.1. Parameter

* `STRING` or `U32` - The subscriber ID, depending on subscription initialization.

#### 1.11.14.2. Response

* `BOOL` - `true` if the subscriber was unsubscribed, `false` if there was no subscriber.

#### 1.11.14.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "state_unsubscribeRuntimeVersion",
    "params": [
        5
    ],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

### 1.11.15. state_subscribeStorage (pubsub)

Storage subscription. If storage keys are specified, it creates a message for each block which changes the specified storage keys. If none are specified, then it creates a message for every block.

This endpoint communicates over the Websocket protocol (`state_storage` subscription).

#### 1.11.15.1. Parameter

* `ARRAY` (OPTIONAL)
  * `HEX` - The storage key.

#### 1.11.15.2. Response

* `MAP`
  * `"block"`: `HASH` - The block hash.
  * `"changes"`: `ARRAY`
    * `ARRAY`
      * `HEX` - The storage key.
      * `HEX` - (OPTIONAL) The new value.

#### 1.11.15.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "state_subscribeStorage",
    "params": [],
    "id": 1
}
```

Response (shortened):

```json
{
    "jsonrpc": "2.0",
    "method": "state_storage",
    "params": {
        "result": {
            "block": "0x7c29399516740ddbbb0d6bd9b34ed67bafe6bee31405d72be3b6b8db72172a0f",
            "changes": [
                [
                    "0x0b76934f4cc08dee01012d059e1b83eebbd108c4899964f707fdaffb82636065",
                    "0x00"
                ],
                [
                    "0x26aa394eea5630e07c48ae0c9558cef78a42f33323cb5ced3b44dd825fda9fcc",
                    null
                ],
                [
                    "0xf68f425cf5645aacb2ae59b51baed9049b58374218f48eaf5bc23b7b3e7cf08a",
                    "0x62952500"
                ]
            ]
        },
        "subscription": 11
    }
}
```

### 1.11.16. state_unsubscribeStorage (pubsub)

Unsubscribe from watching storage.

#### 1.11.16.1. Parameter

* `STRING` or `U32` - The subscriber ID, depending on subscription initialization.

#### 1.11.16.2. Response

* `BOOL` - `true` if the subscriber was unsubscribed, `false` if there was no subscriber.

#### 1.11.16.3. Example

Request:

```bash
{
    "jsonrpc": "2.0",
    "method": "state_unsubscribeStorage",
    "params": [
        14
    ],
    "id": 1
}
```

Response:

```json
{
    "jsonrpc": "2.0",
    "result": true,
    "id": 1
}
```

## 1.12. Child State

### 1.12.1. childstate_getKeys

**Warning**: This method is [UNSAFE](#Safety).

Returns the keys from the specified child storage. The keys can also be filtered based on a prefix.

#### 1.12.1.1. Parameter

* `HEX` - The child storage key.
* `HEX` - The prefix of the child storage keys to be filtered for. Leave empty ("") to return all child storage keys.
* `HASH` - (OPTIONAL) The block hash indicating the state. `NULL` implies the current state.

#### 1.12.1.2. Response

* `ARRAY`
  * `HEX` - (OPTIONAL) Storage key.

### 1.12.2. childstate_getStorage

Returns a child storage entry.

#### 1.12.2.1. Parameter

* `HEX` - The child storage key.
* `HEX` - The key within the child storage.
* `HASH` - (OPTIONAL) The block hash indicating the state. `NULL` implies the current state.

#### 1.12.2.2. Response

* `HEX` - (OPTIONAL) Storage data, if found.

### 1.12.3. childstate_getStorageHash

Returns the hash of a child storage entry.

#### 1.12.3.1. Parameter

* `HEX` - The child storage key.
* `HEX` - The key within the child storage.
* `HASH` - (OPTIONAL) The block hash indicating the state. `NULL` implies the current state.

#### 1.12.3.2. Response

* `HASH` - (OPTIONAL) The hash of the child storage entry, if found.

### 1.12.4. childstate_getStorageSize

Returns the size of a child storage entry.

#### 1.12.4.1. Parameter

* `HEX` - The child storage key.
* `HEX` - The key within the child storage.
* `HASH` - (OPTIONAL) The block hash indicating the state. `NULL` implies the current state.

#### 1.12.4.2. Response

* `U64` - (OPTIONAL) The size of the storage entry in bytes, if found.

## 1.13. Engine

### 1.13.1. Engine Errors

*NOTE: This type is future-reserved, specification will be adjusted.*

Engine RPC API.

### 1.13.2. engine_createBlock

*NOTE: This API is future-reserved, specification will be adjusted.*.

Instructs the manual-seal authorship task to create a new block.

### 1.13.3. engine_finalizeBlock

*NOTE: This API is future-reserved, specification will be adjusted.*.

Instructs the manual-seal authorship task to finalize a block.

## 1.14. Payment

Payment PRC API.

### 1.14.1. Payment Errors

*NOTE: This type is future-reserved, specification will be adjusted.*

### 1.14.2. payment_queryInfo

*NOTE: This API is future-reserved, specification will be adjusted.*.

## 1.15. Contracts

Contracts RPC API.

### 1.15.1. Contracts Errors

*NOTE: This type is future-reserved, specification will be adjusted.*

### 1.15.2. contracts_call

*NOTE: This API is future-reserved, specification will be adjusted.*

### 1.15.3. contracts_getStorage

*NOTE: This API is future-reserved, specification will be adjusted.*

Returns the value under a specified storage key in a contract given by an address at a certain block. If no block chash is provided, the latest values gets returned.

#### 1.15.3.1. Parameter

* `HEX` - The public key.
* `HEX` - The storage key.
* `HASH` - The block hash.

#### 1.15.3.2. Response

* `HEX` - (OPTIONAL) The value of the storage key.

### 1.15.4. contracts_rentProjection

*NOTE: This API is future-reserved, specification will be adjusted.*

## 1.16. BABE

BABE RPC API.

### 1.16.1. BABE Errors

*NOTE: This type is future-reserved, specification will be adjusted.*

### 1.16.2. babe_epochAuthorship

*NOTE: This API is future-reserved, specification will be adjusted.*

Returns data about which slots (primary or secondary) can be claimed in the current epoch with the keys in the key store.

#### 1.16.2.1. Parameter

None.

#### 1.16.2.2. Response

* `ARRAY`
  * `ARRAY`
    * `HEX` - The public key that can claim the slots.
    * `MAP`
      * `"primary"`: `ARRAY` - array of primary slots that can be claimed.
        * `U64` - (OPTIONAL)
      * `"secondary"`: `ARRAY` - array of secondary slots that can be claimed.
        * `U64` - (OPTIONAL)

## 1.17. Sync

Sync State RPC API.

### 1.17.1. sync_state_genSyncSpec

Returns the JSON serialized chain specification running the node (i.e. the current state state), with a sync state.

#### 1.17.1.1. Parameter

*TODO*: What is this used for?

* `BOOL` - ...

#### 1.17.1.2. Response

* `MAP`
  * `"name"`: `STRING` - The name of the chain.
  * `"id"`: `STRING` - The Id fo the chain.
  * `"chainType"`: `STRING` - The chain type.
  * `"bootNodes"`: `ARRAY` - A list of default boot nodes.
    * `STRING` - A boot node multiaddress.
  * `"telemetryEndpoints"`: `ARRAY` - A list of default telemetry endpoints.
    * `ARRAY`
      * `STRING` - An URL to the telemetry endpoint.
      * `U32` - The maximum verbosity level.
  * `"protocolId"`: `STRING`- The protocol Id.
  * `"properties"`: `MAP` - General properties.
    * `"ss58Format"`: `U32`
    * `"tokenDecimals"`: `U32`
    * `"tokenSymbol"`: `STRING`
  * `"forkBlocks"`: `ARRAY` - (OPTIONAL) Block number with known hashes.
    * `ARRAY`
      * `U32` - The block number.
      * `HASH` - The block hash
  * `"badBlocks"`: `ARRAY` - (OPTIONAL) Known bad block hashes.
    * `HASH` - The block hash.
  * `"consensusEngine"`: `NULL` - Never used, left for backward compatibility.
  * `"genesis"`: `MAP`
    * `"raw"`: `MAP`
      * `"top"`: `MAP`
        * `HEX`: `HEX` - A key/value storage entry.
      * `"childrenDefault"`: `MAP`- (OPTIONAL)
        * `HEX`: `MAP` - The child starge key.
          * `HEX`: `HEX` - A key/value storage entry.
  * `"lightSyncState"`: `MAP` - (OPTIONAL)
    * `"babeEpochChanges"`: `HEX` - The epoch changes tree for BABE.
    * `"babeFinalizedBlockWeight"`: `HEX` - The BABE weight of the finalized block.
    * `"finalizedBlockHeader"`: `HEX` - The header of the best finalized block.
    * `"grandpaAuthoritySet"`: `HEX` - The authority set for GRANDPA.

#### 1.17.1.3. Example

Request:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "sync_state_genSyncSpec", "params":[true]}' http://localhost:9933
```

Response (shortened):

```json
{
    "jsonrpc": "2.0",
    "result": {
        "badBlocks": null,
        "bootNodes": [
            "/dns/p2p.cc1-0.polkadot.network/tcp/30100/p2p/12D3KooWEdsXX9657ppNqqrRuaCHFvuNemasgU5msLDwSJ6WqsKc",
            "/dns/p2p.cc1-1.polkadot.network/tcp/30100/p2p/12D3KooWAtx477KzC8LwqLjWWUG6WF4Gqp2eNXmeqAG98ehAMWYH",
            "/dns/p2p.cc1-2.polkadot.network/tcp/30100/p2p/12D3KooWAGCCPZbr9UWGXPtBosTZo91Hb5M3hU8v6xbKgnC5LVao",
        ],
        "chainType": "Live",
        "consensusEngine": null,
        "forkBlocks": null,
        "genesis": {
            "raw": {
                "childrenDefault": {},
                "top": {
                    "0x1a736d37504c2e3fb73dad160c55b2918ee7418a6531173d60d1f6a82d8f4d51000772c5f44ff1ae5233c073e4b0b85d7f070000": "0x66b1ebbaeb6b2beee0ef60ab899c6a6ccfc7e3cbf820e5be26f561b44a56832f00000000000000000000000000000000",
                    "0x1a736d37504c2e3fb73dad160c55b2918ee7418a6531173d60d1f6a82d8f4d51009404c9922aa254ae3e28cda4d8d01efb050000": "0x7c3c0e9543220809e9207ece95c504006574f50c42361068e846dc51f7e44c7100000000000000000000000000000000",
                    "0x1a736d37504c2e3fb73dad160c55b2918ee7418a6531173d60d1f6a82d8f4d5100abaa29210df0d8c56d796350a2d268cb060000": "0xfc5d04e7ff3965c8285a2c23aa573117deeed886bbe5e3be0974f1cf0a2ff21600000000000000000000000000000000",
                }
            }
        },
        "id": "polkadot",
        "lightSyncState": {
            "babeEpochChanges": "0x040911d09...000000002",
            "babeFinalizedBlockWeight": 652541,
            "finalizedBlockHeader": "0xdc1cfc709...6ad97958e",
            "grandpaAuthoritySet": "0xbd0331bba...ee7270000"
        },
        "name": "Polkadot",
        "properties": {
            "ss58Format": 0,
            "tokenDecimals": 10,
            "tokenSymbol": "DOT"
        },
        "protocolId": "dot",
        "telemetryEndpoints": [
            [
                "/dns/telemetry.polkadot.io/tcp/443/x-parity-wss/%2Fsubmit%2F",
                0
            ]
        ]
    },
    "id": 1
}
```

## 1.18. Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
