# Title

- **PSP Number:** [To be assigned (=number of the initial PR to the PSPs repo)]
- **Authors:** Fabio Lama <fabio.lama@pm.me>
- **Status:** Draft
- **Created:** 2022-07-21
- **Reference Implementation:** https://github.com/paritytech/substrate/pull/10918

## Summary

This standard sets weights on expensive resources such as computation, database
access and bandwidth. The weights are determined by benchmarking the Substrate
implementation that provides those resources to the Runtime and displays the
recommended values that alternative implemenations to Substrate should adhere to
in order to unify the cost of those resources and maintain a sustainable fee
structure.

## Motivation

The Substrate implementation determined weights based on the resources that it
provides to the Runtime for consumption, such as I/O, computation and network
bandwidth. Given that alternative implementations are available and can be
developed in the future, those weights should be reflected by those alternative
implemenations so that the fee structure is justifiable for the diverse network
of participants at large. The goal is to avoid situations where alternative
implemenations are too strict or too loose in terms of costs of their resource
requirements which could lead to bad economic outcomes and a fractured network
of implemenations.

## Specification

The specification should describe the feature as detailed as possible. The proposal should be
complete, consistent, unambiguous, quantitative, and feasible.

## Tests

If applicable, please include a list of potential test cases to validate an implementation.

## Copyright

Each PSP must be labeled as placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
