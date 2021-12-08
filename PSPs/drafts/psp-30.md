# Augmented ERC20s

- **PSP Number:** 30
- **Authors:** Hyungsuk Kang(@hskang9)
- **Status:** Draft, Call For Feedback
- **Created:** 2021-11-25
- **Reference Implementation** https://github.com/digitalnativeinc/eip-4469

## Summary

Augmented ERC20s propose a wrapper method to bring ethereum token primitives into xcm-compatible asset class.

## Motivation

There is definitely a friction for developers when it comes to making erc20 compatible to substrate native runtime use cases. This can be solved via a precompile contract which calls transfer functions in evm, but this just provides a virtual controller, not a compatible primitive which it can use full potential of each runtime. Hence, I propose a wrapper to provide each primitives.

## Specification

Similar to the design of [reference implementation](https://github.com/digitalnativeinc/eip-4469), an entry point contract which creates augmentor contract for each erc20 token is provided with `create2` opcode. An augmentor links to the ethereum precompile contract where it executes issuing asset primitives in `pallet-asset` module. 

An evm precompile module is provided with three functions.

`createAsset(name, symbol, decimal)`: `createAsset` function executes `pallet-asset` module to register an asset with arguments received from the evm smart contract(e.g. name, symbol, decimal, etc)

`mint(account, amount)`: `mint` function requires the caller to be the augmentor contract registered in the entry point contract and mints certain amount of asset in given account's balance.

`burn(account, amount)`: `burn` function requires the caller to be the augmentor contract registered in the entry point contract and burns certain amount of asset in given account's balance.

An entry point contract has storage 
```
mapping(address -> address) augmentors;
```
An entry point contract for augmentors should have one function.

`createAugment(token) returns (address augmentor)`: `createAugment` function checks whether the sender is the owner of the erc20:
```
require(msg.sender == IERC20(token).owner(), "ACCESS_INVALID");
```
then it executes `createAsset` function from `pallet-asset` after registering augmentor contract in `augmentors` storage. augmentor contract address is returned.

An augmentor contract has two functions to augment/decrease tokens:

`augment(amount)`: Through an augmentor contract, an ERC20 token holder deposits token into the augmentor contract, then the augmentor contract executes `mint` function to the sender with the same amount deposited.

`decrease(amount)`: Through an augmentor contract, the augmentor executes `burn` function to the sender account with the given amount, then transfers ERC20 tokens to the sender account

## Tests

# Test case 1: CreateAugment is initiated by only owner of the contract


# Test case 2: Augment/Decrease function works within a parachain runtime


# Test case 3: Test with xcm tx to store token information in `pallet-asset` module in statemint 

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
