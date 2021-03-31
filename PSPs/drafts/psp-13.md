# Voting Standard

- **PSP Number:** 13
- **Authors:** @ETeissonniere
- **Status:** Draft
- **Created:** 2021-03-31
- **Reference Implementation** https://github.com/NucleiStudio/governance-os

## Summary

Uniformize various voting systems implementations behind a common trait to allow for easy change between runtimes or support for decentralized organizations with pluggable voting systems.

## Motivation

Thanks to a set of W3F grant, I was able to focus on building a set of pallets to supports multiple decentralized organizations in one runtime (instead of the traditional "chain dao" model as common in the substrate ecosystem). Once I started building various voting pallets I realized that a uniformized standard would be needed to allow for the easy integration of multiple voting systems in one runtime without having to create yet a new pallet with yet a new set of extrinsics or coupling traits every time.

## Specification

We propose the creation of a coupling trait that must be supported by any voting implementations. Other pallets can require this trait to be loosely coupled with any voting pallet.

Our proposed trait is as follows, we have included the relevant documentation in the function's comments. We have also included the definition of a `ProposalResult` type to correctly flag a proposal's result.
```rust
use codec::{Decode, Encode};
#[cfg(feature = "std")]
use serde::{Deserialize, Serialize};
use sp_runtime::{DispatchError, DispatchResult, RuntimeDebug};
use sp_std::result;

/// End result of a proposal being closed.
#[derive(Encode, Decode, Clone, PartialEq, Eq, RuntimeDebug)]
#[cfg_attr(feature = "std", derive(Serialize, Deserialize))]
pub enum ProposalResult {
    Passing,
    Failing,
}

/// A common trait accross all voting implementations to make it easy to change
/// between voting models or implementations.
/// A pallet implementing this trait is not necessarily in charge of storing
/// proposals but could stick to only to support the actual decison making
/// code while proposal storage is delegated to another pallet.
pub trait StandardizedVoting {
    /// How we represent the a proposal as passed to the underlying functions.
    /// This can be used to fetch any state associated to the proposal.
    type ProposalId;

    /// How the parameters of a voting system are represented and set at the
    /// organization level.
    type Parameters;

    /// How voting data is passed to the underlying pallet.
    type VoteData;

    /// How accounts are represented, used to identify voters.
    type AccountId;

    /// A proposal is being created. Handle any eventual registration and trigger
    /// an error if any preconditions are not met. Shall be called before any other
    /// state changes so that it is safe to fail here. It is the caller's responsibility
    /// to try and prevent overwrites or duplicated proposals.
    fn initiate(proposal: Self::ProposalId, parameters: Self::Parameters) -> DispatchResult;

    /// Special function to handle the case when a proposal is being vetoed. This
    /// should clean any storage or state associated to the given proposal.
    fn veto(proposal: Self::ProposalId) -> DispatchResult;

    /// Handle the reception of a new vote for the given proposal. This should mutate any
    /// state linked to the proposal accordingly.
    fn vote(
        proposal: Self::ProposalId,
        voter: &Self::AccountId,
        data: Self::VoteData,
    ) -> DispatchResult;

    /// Handle the closure of a proposal or return an error if it cannot be closed because
    /// some conditions are not met. Shall return an indicator on wether the proposal is
    /// passing (should be executed) or not (should be discarded).
    fn close(proposal: Self::ProposalId) -> result::Result<ProposalResult, DispatchError>;
}
```

## Tests

[To be created once more feedback is collected]

## Copyright

This document is placed in the
[public domain](https://creativecommons.org/publicdomain/zero/1.0/).
