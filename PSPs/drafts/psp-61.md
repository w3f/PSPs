# Standard Interface Detection

- **PSP Number:** 61
- **Authors:** Artem Fomiuk <artem.lech@727.ventures>, Bohdan Ohorodnii <varex.silve@727.ventures>, Dominik Krizo <dominik.krizo@727.ventures>, Markian Ivanichok <markian@727.ventures>
- **Status:** Draft
- **Created:** 2023-07-20
- **Reference Implementation** https://github.com/Brushfam/openbrush-contracts/pull/112

## Summary

A standard for a standard interface detection for WebAssembly smart contracts which run on Substrate's [`contracts` pallet](https://github.com/paritytech/substrate/tree/master/frame/contracts).

This proposal aims to define the interface detection standard for WebAssembly smart contracts, inspired by [EIP-165](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-165.md) for the Ethereum ecosystem.

## Motivation

Motivation of this proposal is to provide a standard way for smart contracts to detect the interface of the smart contract they are interacting with. 
This is useful for smart contracts which interact with other smart contracts, as it allows them to detect the interface of the smart contract they are interacting with and adjust their behavior accordingly, 
without having any ABI of that contracts.

## Specification

### 1. Interface ID (TRAIT_ID)

Interface ID is a 4-byte identifier of the interface, stored in `u32` big-endian format. It is calculated as the first 4 bytes of the blake2b hash of the string, 
that is concatenation of all messages, sorted in lexicographical order. Such as in the example below:

```rust
message_selectors.sort_unstable();

let trait_id = ::ink_ir::Selector::compute(&message_selectors.join("").into_bytes()).into_be_u32();
```

Where `message_selectors` is a vector of all message selectors of the interface in the format `<trait-name>::<message-name>`, 
and `::ink_ir::Selector::compute` that computes the BLAKE-2 256-bit based selector from the given input bytes.

```rust
pub fn compute(input: &[u8]) -> Self {
    let mut output = [0; 32];
    blake2b_256(input, &mut output);
    Self::from([output[0], output[1], output[2], output[3]])
}
```

### 2. Interface Detection

Interface detection is provided as standard trait `PSP61`, that looks like this:

```rust
#[openbrush::trait_definition]
pub trait PSP61 {
    #[ink(message)]
    fn supports_interface(&self, interface_id: u32) -> bool;

    #[ink(message)]
    fn supported_interfaces(&self) -> Vec<u32>;
}
```

With the abi of messages:
```json
{
        "args": [],
        "default": false,
        "docs": [],
        "label": "PSP61::supported_interfaces",
        "mutates": false,
        "payable": false,
        "returnType": {
          "displayName": [
            "ink",
            "MessageResult"
          ],
          "type": 14
        },
        "selector": "0xea0cf0ba"
},
{
  "args": [
    {
      "label": "interface_id",
      "type": {
        "displayName": [
          "psp61_external",
          "SupportsInterfaceInput1"
        ],
        "type": 3
      }
    }
  ],
  "default": false,
  "docs": [],
  "label": "PSP61::supports_interface",
  "mutates": false,
  "payable": false,
  "returnType": {
    "displayName": [
      "ink",
      "MessageResult"
    ],
    "type": 16
  },
  "selector": "0x0d0f1b07"
}
```

### 3. Interface Detection Implementation

As an example, we provide an implementation of the `PSP61` using `Openbrush` library:


Trait definition:
```rust
// contracts/traits/psp61/mod.rs
use ink::prelude::vec::Vec;

#[openbrush::trait_definition]
pub trait PSP61 {
    #[ink(message)]
    fn supports_interface(&self, interface_id: u32) -> bool;

    #[ink(message)]
    fn supported_interfaces(&self) -> Vec<u32>;
}
```

Helper traits and macros:
```rust
// contracts/src/utils/psp61.rs
pub use crate::{
    psp61,
    traits::psp61::*,
};
use ink::prelude::{
    vec,
    vec::Vec,
};

pub trait PSP61Internal {
    fn _interfaces(&self) -> Vec<u32> {
        vec![]
    }
}

pub trait PSP61InternalOB {
    fn _interfaces_ob(&self) -> Vec<u32> {
        vec![]
    }
}

pub trait PSP61Impl: PSP61Internal + PSP61InternalOB {
    fn supports_interface(&self, interface_id: u32) -> bool {
        self._interfaces().contains(&interface_id) || self._interfaces_ob().contains(&interface_id)
    }

    fn supported_interfaces(&self) -> Vec<u32> {
        let mut interfaces = self._interfaces();
        interfaces.append(&mut self._interfaces_ob());
        interfaces
    }
}

/// Macro for implementing PSP61Internal trait
#[macro_export]
macro_rules! supported_interfaces {
    ($contract:ident => $($interface_id:expr),*) => {
        impl ::openbrush::contracts::psp61::PSP61Internal for $contract {
            fn _interfaces(&self) -> ::ink::prelude::vec::Vec<u32> {
                ::ink::prelude::vec![$($interface_id),*]
            }
        }
    };
    ($contract:ident) => {
        impl ::openbrush::contracts::psp61::PSP61Internal for $contract {}
    };
}
```

And implementation of this trait is done via `#[openbrush::implementation]` macro:
```rust
// examples/psp61/lib.rs
#![cfg_attr(not(feature = "std"), no_std, no_main)]

#[openbrush::implementation(PSP61, Ownable)]
#[openbrush::contract]
pub mod my_psp61 {
    use ink::prelude::vec;
    use openbrush::contracts::supported_interfaces;
    use openbrush::traits::Storage;

    #[ink(storage)]
    #[derive(Default, Storage)]
    pub struct Contract {
        #[storage_field]
        pub ownable: ownable::Data,
    }

    supported_interfaces!(Contract);

    impl Contract {
        #[ink(constructor)]
        pub fn new() -> Self {
            Self::default()
        }

        #[ink(message)]
        pub fn ownable_id(&self) -> u32 {
            ownable::ownable_external::TRAIT_ID
        }
    }
}
```

### Additional Information

- Common TRAIT_IDs for PSPs:

| Trait name | TRAIT_ID      |
|------------|---------------|
| PSP22      | 2_746_973_080 |
| PSP34      | 4_286_560_456 |
| PSP37      | 2_478_854_925 |
| PSP61      | 3_629_194_953 |

## Tests

Tests that are created for the previous example are located in the `examples/psp61/lib.rs`, and have the following cases:

```rust
#[cfg(test)]
mod tests {
    use super::Contract;
    use openbrush::contracts::psp61::PSP61;
    use openbrush::contracts::timelock_controller;
    use openbrush::contracts::upgradeable;
    use openbrush::contracts::{access_control, ownable, pausable, psp61};

    #[ink::test]
    fn assure_ids_are_proper() {
        let contract = Contract::new();

        assert_eq!(contract.ownable_id(), ownable::ownable_external::TRAIT_ID);
        assert_eq!(
            contract.access_control_id(),
            access_control::accesscontrol_external::TRAIT_ID
        );
        assert_eq!(contract.pausable_id(), pausable::pausable_external::TRAIT_ID);
        assert_eq!(
            contract.timelock_controller_id(),
            timelock_controller::timelockcontroller_external::TRAIT_ID
        );
        assert_eq!(contract.upgradeable_id(), upgradeable::upgradeable_external::TRAIT_ID);
        assert_eq!(contract.psp61_id(), psp61::psp61_external::TRAIT_ID);
    }

    #[ink::test]
    fn check_for_interfaces() {
        let contract = Contract::new();

        assert_eq!(contract.supports_interface(ownable::ownable_external::TRAIT_ID), true);
        assert_eq!(
            contract.supports_interface(access_control::accesscontrol_external::TRAIT_ID),
            true
        );
        assert_eq!(contract.supports_interface(pausable::pausable_external::TRAIT_ID), true);
        assert_eq!(
            contract.supports_interface(timelock_controller::timelockcontroller_external::TRAIT_ID),
            true
        );
        assert_eq!(
            contract.supports_interface(upgradeable::upgradeable_external::TRAIT_ID),
            true
        );
        assert_eq!(contract.supports_interface(psp61::psp61_external::TRAIT_ID), true);
    }

    #[ink::test]
    fn check_for_interfaces_batch() {
        let contract = Contract::new();

        let ids = contract.id_batch();
        let mut interfaces = contract.supported_interfaces();

        let mut ids: Vec<_> = ids
            .into_iter()
            .map(|(_, id)| {
                assert_eq!(contract.supports_interface(id), true);
                id
            })
            .collect();

        ids.sort_unstable();
        interfaces.sort_unstable();

        assert_eq!(ids, interfaces);
    }
}
```

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
