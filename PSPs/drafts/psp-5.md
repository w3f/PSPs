# Structure for genesis parameters

* **PSP Number:** 001
* **Authors:** Fabio Lama <github.com/lamafab, fabio@web3.foundation>
* **Status:** Call for Feedback
* **Created:** 2019-12-04
* **Reference Implementation:** https://github.com/w3f/polkadot-spec/blob/master/genesis-state/kusama/ksmcc3/kusama.json

## Summary

This standard describes the basic JSON structure (refered to as "genesis file") in which genesis parameters including relevant data and metadata of the network should be saved and distributed.

## Motivation

The [Polkadot specification](https://github.com/w3f/polkadot-spec) describes how genesis parameters should be inserted into the state storage, but networks contain additional useful information such as boot nodes, protocol identifiers, telemetry endpoints and properties that are not part of the genesis state. A standardized JSON structure allows a portable integration for different Polkadot implementations, nodes and and other tools, allowing these software to correctly read and process the data.

## Specification

The specification should describe the feature as detailed as possible. The proposal should be complete, consistent, unambiguous, quantitative, and feasible.

Format:
|Name              |Type                     |Mandatory|Description|
|------------------|-------------------------|---------|-----------|
|name              |String                   |YES      |           |
|id                |String                   |YES      |           |
|bootNodes         |Array[String]            |YES      |           |
|telemetryEndpoints|Array[Array[String, Int]]|NO       |           |
|protocolId        |String                   |NO       |           |
|properties        |Map[String:Value]        |NO       |           |
|consensusEngine   |null                     |YES      |           |
|genesis           |Object->Genesis          |YES      |           |

Object: Genesis
|Name              |Type                     |Mandatory|Description|
|------------------|-------------------------|---------|-----------|
|raw               |Map[String:String]       |YES      |           |

Example:
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
        "0x9c5d795d0297be56027a4b2464e333979c5d795d0297be56027a4b2464e33397997f7003f78328f30c57e6ce10b1956c77d2187fe08441845cc0c18273852039": "0x00703874580800000000000000000000",
        ...
      },
      {}
    ]
  }
}
```

## Tests

If applicable, please include a list of potential test cases to validate an implementation. 

## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
