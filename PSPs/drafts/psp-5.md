# Structure for genesis parameters

* **PSP Number:** 001
* **Authors:** Fabio Lama <github.com/lamafab, fabio@web3.foundation>
* **Status:** Call for Feedback
* **Created:** 2019-12-04
* **Reference Implementation:** https://github.com/w3f/polkadot-spec/blob/master/genesis-state/kusama/ksmcc3/kusama.json

## Summary

This standard describes the basic JSON structure (refered to as "genesis file") in which genesis parameters including relevant data and metadata of the network should be saved and distributed.

The detailed description and requirements for the genesis parameters can be found in the [Polkadot Runtime Environment Specification](https://github.com/w3f/polkadot-spec).

## Motivation

The Polkadot Runtime Environment Specification describes how genesis parameters should be inserted into the state storage, but networks contain additional useful information such as boot nodes, protocol identifiers, telemetry endpoints and properties that are not part of the genesis state. A standardized JSON structure allows a portable integration for different Polkadot implementations, nodes and and other tools, allowing these software to correctly read and process the data.

## Specification

The genesis file must be represented in the following format:

|Name              |Type                     |Mandatory|Description|
|------------------|-------------------------|---------|-----------|
|name              |String                   |YES      |The name of the chain network|
|id                |String                   |YES      |Client-side parameter for logging, directory names, etc |
|bootNodes         |Array[String]            |YES      |List of `libp2p` [Multiaddresses](https://docs.libp2p.io/concepts/addressing/), including protocol id and multihash|
|telemetryEndpoints|Array[Array[String, Int]]|NO       |A list of Websocket telemetry endpoints pairs, where the first value is a address prefixed with a `wss://` schema and the second value is a number indicating the logging verbosity|
|protocolId        |String                   |NO       |The identifier to of the chain network, gets sent with each request|
|properties        |Object->Properties       |NO       |Metadata for the chain network|
|consensusEngine   |null                     |YES      |Never used, left only for backward compatibility|
|genesis           |Object->Genesis          |YES      |Contains the genesis parameters|

**Object:** Properties
|Name              |Type                     |Mandatory|Description|
|------------------|-------------------------|---------|-----------|
|ss58Format        |Int                      |NO       |The format of the [ss58 address format]()|
|tokenDecimals     |Int                      |NO       |The decimal precision of the native token|
|tokenSymbol       |String                   |NO       |Symbol of the native token|

**Object:** Genesis
|Name              |Type                     |Mandatory|Description|
|------------------|-------------------------|---------|-----------|
|raw               |Object[String:String]       |YES      |The key/values pairs required for the runtime|

**Example:**
```
{
  "name": "Kusama CC3",
  "id": "ksmcc3",
  "bootNodes": [
    "/dns4/p2p.cc3-0.kusama.network/tcp/30100/p2p/QmeCit3Nif4VfNqrEJsdYHZGcKzRCnZvGxg6hha1iNj4mk",
    "/dns4/p2p.cc3-1.kusama.network/tcp/30100/p2p/QmchDJtEGiEWf7Ag58HNoTg9jSGzxkSZ23VgmF6xiLKKsZ"
  ],
  "telemetryEndpoints": [
    [
      ["wss://telemetry.polkadot.io/submit/", 0]
    ]
  ],
  "protocolId": "ksmcc3",
  "properties": {
    "ss58Format": 2,
    "tokenDecimals": 12,
    "tokenSymbol": "KSM"
  },
  "consensusEngine": null,
  "genesis": {
    "raw": [
      {
        "0x9c5d795d0297be56027a4b2464e333979c5d795d0297be56027a4b2464e33397974a8f6e094002e424b603628718939b060c4c6305a73d36a014468c29b8b7d7": "0x00c0e1d0612100000000000000000000",
        "0x9c5d795d0297be56027a4b2464e333979c5d795d0297be56027a4b2464e33397997f7003f78328f30c57e6ce10b1956c77d2187fe08441845cc0c18273852039": "0x00703874580800000000000000000000"
      },
      {}
    ]
  }
}
```

## Tests

TODO: If applicable, please include a list of potential test cases to validate an implementation. 

## Copyright

TODO: Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).

## TODO
- Why is the client side `id` mandatory while `protocol_id` is not?
- Shouldn't `id` and `protocol_id` be the same?
- What does "format" of the ss58 address format mean?
- In the (kusama) JSON file the `property` field has certain key/value pairs, but the Substrate code accepts anything. This behaviour makes sense in terms of the genesis parameters, since those values just get inserted into the state storage. But how can `property` be useful if those values are not predefined?
- What does the second object in `"genesis"` > `"raw"` do? See: `"raw": [{},{}]`
- Since Polkadot has not been released yet, why is `"consensusEngine"` a legacy field?
- What should be defined in the header value "Reference Implemenation"?
- What's the logging verbosity for telemetry endpoints?