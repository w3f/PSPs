# Refundable Subscription Contract

- **PSP Number:** -
- **Authors:** 
    * Saber Zafarpoor <github.com/SaberDoTcodeR, szafarpoor@ce.sharif.edu>
    * Hadi Esna <github.com/hadiesna, esnaa@ce.sharif.edu>

- **Status:** Draft
- **Created:** [2021-01-26]
- **Reference Implementation** 
    * [Ink! implementation](https://github.com/oxydev/SubsCrypt-ink)
    * [Solidity implementation](https://github.com/oxydev/SubsCrypt-solidity)


## Summary

With ever growing subscription services in web2, the need to provide trustless platform for subscription managemnet emerges. But to effeciently manage refunds, dynamic refund policies and unlitmited number of providers to use this contract, are obstacles that can be bypassed by this proposed implementation.

## Motivation

We are motiviated to proposed our implementation so it can be easiliy integerated in tokens to be used widely in standrad tokens(altough we are designing it as a seperate contract). This can cause to widespread uasge of Web3.

## Specification

### Contract Specification 

| Function | Description | Params | Returns | State mutability | 
| ------------- | ------------- | ------------- | ------------- | ------------- |
| addPlan | This function is for providers to add their plans; each plan has duration, price, max refund percent that they are willing to lock in contract and withdraw after that the subscription period has finished. | list of durations, list of prices, list of max refund percent | None | change state |
| editPlan | This function is for providers to edit their plan. (Old subscriptions are not affected by this change) | index of their plan, new duration, new max refund percent, new price | None | change state |
| changeDisable | This function is for providers to edit their plan that changes the active or deactivate status of their plan(so people can or can't subscribe in that plan) | plan index| None | change state |
| subscribe | This payable function is for users to subscribe to their desired service and plan; they have to provide a hash of their password (the auth mechanism will be explained thoroughly in Auth Section) and provider address and plan index and some metadata that is encrypted by the public key of the provider(users can trust providers to share their data with but nobody else can know that data) | provider address, plan index, the hash of pass, An optional encrypted metadata| None | change state(payable) |
| refund | This function is for users to refund their subscribe anytime they want and instantly withdraw the rest of their money(maximum amount of refund is indicated by max refund percent that provider had set for that plan) | provider address, plan index| None |  change state |
| withdraw | This function is for providers to withdraw the amount that is now ready to withdraw(this is the money that we lock in the contract when a user subscribes to a plan according to max refund percent, and when their plan is finished, that money can be withdrawn). We used an optimized LinkedList solution, which is really cheap to execute and fast. | None | amount of money you are paid  | change state |
| checkSubscription | This function is for users or anyone to check that if a user has an active subscription in a specific plan of a provider | address of the user, address of provider, plan index| return boolean | view |
| checkAuth | This function is used to check if the given combination of token and passphrase can authenticate a specific user for a provider(the auth mechanism will be explained thoroughly in Auth Section) | address of the user, address of provider, token, passphrase| return boolean | view |
| retrieveWholeDataWithPassword | This function is used to get every subscription record of a user with their token and passphrase, which first have to be set in setSubsCryptPass function(this token and passphrase is only worked to login in SubsCrypt website to have a whole review of your account)  | address of the user, token, passphrase| return whole records of a user | view |  
| retrieveWholeDataWithWallet | This function is the same as the above function with a slight difference that it is used with user wallet to trigger the contract directly | None | return whole records of a user | view |
| retrieveDataWithPassword | This function is used to get every subscription record of a user only related to a specific provider with their token and passphrase is set once they subscribe to their chosen plan of that provider | address of the user, address of provider, token, passphrase| return whole records of a user | view |
| retrieveDataWithWallet | This function is the same as the above function with a slight difference that it is used with user wallet to directly trigger the contract | address of provider| return whole records of a user-related to that provider | view |

### Auth Specification 

Users have to choose a pair of token and passphrase, which we recommend having a common token(more than 16 chars) for every provider that they subscribe but having a different passphrase(can be small). Users will submit sha256(token+passphrase) to the contract, and whenever they want to authenticate themselves, they have to provide these token and passphrase in a view function( which is not a transaction, so it's not on-chain and free).
They also have to once set their sha256(token+passphrase) for using the SubsCrypt dashboard without a wallet. The authentication with a wallet is checked by the blockchain address sender.

### Refund Specification

When providers ask to submit their plan in the contract they can set a custom number (0 to 1000) that is divided by 1000 that shows how much a provider is willing to refund in their plan. After a user subscribed we will lock that refund amount till the plan time is finished or user asked for refund.

## Copyright

This document is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
