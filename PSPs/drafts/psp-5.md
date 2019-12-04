# Structure for genesis parameters

* **PSP Number:** 001
* **Authors:** Fabio Lama <github.com/lamafab, fabio@web3.foundation>
* **Status:** Call for Feedback
* **Created:** 2019-12-04
* **Reference Implementation:** https://github.com/w3f/polkadot-spec/blob/master/genesis-state/kusama/ksmcc3/kusama.json

## Summary

This standard describes the basic JSON structure (refered to as "genesis file") of how genesis parameters including relevant data and metadata of the network should be distributed and processed.

## Motivation

The [Polkadot specification](https://github.com/w3f/polkadot-spec) describes how genesis parameters should be inserted into the state storage, but networks contain additional useful information such as boot nodes, protocol identifiers, telemetry endpoints and properties that are not part of the genesis state. A standardized JSON structure allows a portable integration for different Polkadot implementations, nodes and and other tools, allowing these software to correctly use and process the data.

## Specification

The specification should describe the feature as detailed as possible. The proposal should be complete, consistent, unambiguous, quantitative, and feasible.

## Tests

If applicable, please include a list of potential test cases to validate an implementation. 

## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
