# Substrate Uniform Resource Identifier

* **PSP Number:** [To be assigned]
* **Authors:** xlc
* **Status:** Draft
* **Created:** 2019-10-01
* **Reference Implementation** [To be implemented]

## Summary

A URI format that identifies a resource in the Polkadot ecosystem, which can be an independent chain, a module in a chain or one of the resource provided by a module in a chain (e.g. A smart contract in a chain with contracts module).

## References

- https://hackmd.io/gQKQGf42TeOODid3hM4_1w#Identifying-and-utilising-a-fungible-asset-on-Polkadot
- https://tools.ietf.org/html/rfc3986

## Motivation

In the cross-chain Substrate / Polkadot ecosystem, there will be multiple chains interacting with each other. Therefore it is esstenital to have a uniformed way to address each chain and the resources / services hosted by different chain.

Substrate Uniform Resource Identifier (SRUI) provides a way to uniformally address a cross-chain resource.

A resource can be:

- A blockchain. e.g. Polkadot, Ethereum
- A runtime module in a chain. e.g. Balances in Polkadot (DOT), Crowdfunding module in Polkadot
- A resource provided by a runtime module in a chain. e.g. An instance of a smart contract in a contracts module in a smart contract chain

More examples:

- A native fungible token of a chain. e.g. DOT on Polkadot, BTC on Bitcoin, ETH on Ethereum
- A non-fungible token provided by a chain.
- An instance of a non-fungible token. e.g. A particular Crypto Kitty
- A bridgged token provided by a chain. e.g. XBTC on ChainX
- A utility token provided by runtime modules or smart contracts. e.g. MKR on Ethereum, SDOT on ChainX
- A service provided by a chain. e.g. DEX on exchange chain
- A traiding pair in a exchange that provided by a chain. e.g. BTCETH pair on a DEX chain

## Specification

### URI format

A URI is made of various parts:

```
URI = scheme:[//authority]path[?query][#fragment]
authority = [userinfo@]host[:port]
```

For SRUI, the format is defined as

- schema: TBD
  - `polkadot` or `substrate` or something more generalized?
- port: N/A
- userinfo: the parachain ID
- host: the relay chain index or address
  - Leave empty to point to current chain
- path: the resource path in a chain
  - Begin with the runtime moudle name and followed by runtime module specific path
- query & fragment: unspecified

### Encoding

There are two encoding format for SUI.

- Standard encoding
  - Encoded as a URI string

- Custom SCALE ecnoding to avoid unnecessary space usage
  - ```rust
  struct SRUI {
    userinfo: Option<ParaId>,
    host: Option<ChainId>,
    path_components: Vec<RuntimeStr>,
    query: Option<RuntimeStr>,
    fragment: Option<RuntimeStr>
  };
  type RuntimeStr = Vec<u8>;
  type ParaId = u32; // per Polkadot implementation
  enum ChainId {
    Id(u32), // numeric chain id registry is out of scope of this proposol
    Name(RuntimeStr)
  };
  ```

## Tests

(using `polkadot` as the schema)

- Polkadot DOT: `polkadot://polkadot/balances`
- The Default Inatnace Balances token on parachain 1: `polkadot://1@polkadot/srml-balances`
- The Instance2 Balances token on parachain 2: `polkadot://2@polkadot/balances.2`
- Generic Asset Asset ID 5 token on a indenpent chain: `polkadot://chain-name/srml-generic-asset/5`
- The smart contract module on parachain 3 on Kusama: `polkadot://3@kusama/srml-contracts`
- A NFT with ID of  `UniqueWeapon001` with provided by a smart contract with address of `0x1234abcd` on a Instance2 contracts module on parachain 4 on Kusama: `polkadot://4@kusama/srml-contracts.2/0x1234abcd/UniqueWeapon001`
- The balances of user `0x1111aaaa` of a ERC20-like token contract with address of `0x2222bbbb` on a smart contract chain: `polkadot://smart-contract-chain-name/srml-contracts/0x2222bbbb/0x1111aaaa`


## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
