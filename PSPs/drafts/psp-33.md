# Adding open token standards to enable/support institutions and issuers on Polkadot

- **PSP Number:** 33
- **Authors:** Tokensoft.io / Wrapped.com
- **Status:** Draft
- **Created:** 2022-01-13
- **Reference Implementation** Pending

## Summary

Institutions and issuers have complex transactional and reporting requirements.
Initiatives must adhere to corporate, regulatory and compliance policies. Any
formal technical integration or deployment of capital would be subject to these
prerequisites.

## Motivation

Additional open standards and features that institutions and issuers can
leverage on Polkadot will help bring additional institutional liquidity and
drive the issuance of more assets on the network as it scales.

## Specification

### Restrictions

Institutions and issuers need the ability for certain types of token transfers to
be restricted, so that any wallets interacting with a token can first query to
determine if the transfer is allowed, and if not, show the appropriate error to
the user.

### Roles

Institutions and issuers need the ability to be able to specify more granular
roles when it comes to a token’s administrative functions. Below are some
examples of those that are most relevant to these entities, based on our
experience setting up role-based functions on assets deployed onto other
networks / chains:

+ Owner: Owners are responsible for managing permissions of all roles.
+ Burner: These accounts can burn tokens from accounts.
+ Minter: These accounts can mint new tokens to accounts.
+ Pauser: These accounts can halt all transfers of the token.
+ Reclaimer: These accounts can reclaim tokens from other accounts into their
  own.
+ Allowlister: These accounts can configure allowlist rules and add/remove
  accounts from allowlists.
+ Denylister: These accounts can add or remove addresses to or from the
  denylist.

### Allowlists

This feature would ensure that before tokens can be transferred, a transaction
must be validated that the source is allowed to send to that destination and
that the destination can also take receipt funds. If the sender does not check
this in advance and sends an invalid transfer, the transfer functionality will
fail and the transaction will revert.

Owner accounts would have the ability to transfer tokens to any valid recipient,
regardless of the allowlist configuration state.

An Owner could enable and disable the allowlist functionality to remove the
allowlist restrictions on transfers. The default state would be set as
disabled in the initialization of the token.

<img width="632" alt="image" src="https://github.com/w3f/PSPs/blob/master/src/psp-33/allowlist-configuration.png">

Examples:

+ Allowlist A is only allowed to send to itself.
+ Allowlist B is allowed to send to itself and allowlist A.
+ Allowlist C is allowed to send to itself and allowlists A and B.
+ Allowlist D is not allowed to transfer to any allowlist, including itself.

Administrators will have the ability modify a allowlist after a token has been
initialized.

### Denylists

This feature would ensure that before tokens can be transferred, if the
denylisting feature was enabled, it would check to ensure both the source and
destination are not denylisted. If either is denylisted the transfer will fail.

An Owner could enable and disable the denylist functionality to add or remove
restrictions on transfers. The default state would be set as disabled.

Accounts with the Denylister could add or remove accounts to or from the
denylist.

### Pausing

This feature would ensure that Pauser accounts may pause/un-pause a token.

When a token is paused all transfers will be blocked. When deployed a token is
initially un-paused.

### Reclaim

This feature would ensure Reclaimer accounts can reclaim tokens from any
account. Reclaiming tokens would have no effect on the total supply, it
increases the balance of the account reclaiming the tokens and decreases the
balance of the account the tokens are reclaimed from.

## Tests

N/A

## Copyright

[Public domain](https://creativecommons.org/publicdomain/zero/1.0/) (PSP
requirement)
