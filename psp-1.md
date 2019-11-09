# PSP-1: Polkadot Domain Name Service

* **PSP Number:** 1
* **Authors:** Hyungsuk Kang(hskang9)
* **Status:** PoC
* **Created:** [2019-10-26]
* **Reference Implementation** 
[EIP-137: Ethereum Domain Name Service](https://eips.ethereum.org/EIPS/eip-137) 

## Summary

As the Ethereum name service did, this draft PSP describes the details of the Polkadot Name Service, a proposed protocol and runtime module function that provides flexible resolution of short, human-readable names to service and resource identifiers. This permits users and developers to refer to human-readable and easy to remember names, and permits those names to be updated as necessary when the underlying resource (contract, content-addressed data, etc.) changes.

The goal of domain names is to provide stable, human-readable identifiers that can be used to specify network resources. In this way, users can enter a memorable string, such as ‘gavin.wallet’ or ‘www.mysite.ipfs’, and be directed to the appropriate resource. The mapping between names and resources may change over time, so a user may change wallets, a website may change hosts, or a swarm document may be updated to a new version, without the domain name changing. Further, a domain need not specify a single resource; different record types allow the same domain to reference different resources. For instance, a browser may resolve ‘mysite.ipfs’ to the IP address of its server by fetching its A (address) record, while a mail client may resolve the same address to a mail server by fetching its MX (mail exchanger) record.

## Motivation
As Nick Johnson said,
Addresses in blockchains suffer several shortcomings that will significantly limit their long-term usefulness:
- A single global namespace for all names with a single 'centralised' resolver.
- Limited or no support for delegation and sub-names/sub-domains.
- Only one record type, and no support for associating multiple copies of a record with a domain.
- Due to a single global implementation, no support for multiple different name allocation systems.
- Conflation of responsibilities: Name resolution, registration, and whois information.

Use-cases that these features would permit include:
- Support for subnames/sub-domains - eg, live.mysite.tld and forum.mysite.tld.
- Multiple services under a single name, such as a DApp hosted in IPFS, a Whisper address, and a mail server.
- Support for DNS record types, allowing blockchain hosting of 'legacy' names. This would permit an Ethereum client such as Mist to resolve the address of a traditional website, or the mail server for an email address, from a blockchain name.
- DNS gateways, exposing ENS domains via the Domain Name Service, providing easier means for legacy clients to resolve and connect to blockchain services.

The first two use-cases, in particular, can be observed everywhere on the present-day internet under DNS, and we believe them to be fundamental features of a name service that will continue to be useful as the Polkadot develops and matures.

The normative parts of this document do not specify an implementation of the proposed system; its purpose is to document a protocol that different resolver implementations can adhere to in order to facilitate consistent name resolution. An appendix provides sample implementations of resolver contracts and libraries, which should be treated as illustrative examples only.

Likewise, this document does not attempt to specify how domains should be registered or updated, or how systems can find the owner responsible for a given domain. Registration is the responsibility of registrars, and is a governance matter that will necessarily vary between top-level domains.

Updating of domain records can also be handled separately from resolution. Some systems, such as IPFS, may require a well defined interface for updating domains, in which event we anticipate the development of a standard for this.

## Specification

## Overview

ENS in Polkadot just needs one part:
- resolver

The resolver is the state where it manages all domains with addresses. Think of it as an Ethereum world state just for the DNS system.

## Data model

The struct of Domain data is:

```rust
// AccountId, Balacne, Moment are the generic types that are used in Substrate Runtime Module Library(SRML).
pub struct Domain<AccountId, Balance, Moment> {
    /// the account which the domain points
    source: AccountId,
    /// the current price of the domain
    price: Balance,
    /// Time to live: the time the domain lives starting from the registered date
    ttl: Moment,
    /// Date of registration from update or creation
    registered_date: Moment,
    /// Whether domain is available for the auction
    available: bool,
    /// Highest bid price in the auction phase
    highest_bid: Balance,
    /// The highest bidder
    bidder: AccountId,
    /// Auction closing date in timestamp
    auction_closed: Moment
}
```

The resolver records the auction status and domain information for each item.
If someone registers a domain which did not exist before, they get domain paying the initilization price(1 milli dot) and have it for 1 year.
The format of the domain(e.g. .io, .com, .dot, etc) can be limited using regular expression in frontend library or in the module using rust `regex` lib in `no_std` mode.

For example, suppose you wish to register the address of Bob.
```javascript
var pns = require('polkadot-name-service');
var account = pns.getAccountFromJSON("5EtCcn1pim8ty6kzCgEyJNLDXNRuqzMjGGavNh9y6qKUkst2.json");
pns.setAccount(account);
// namehash runs regex if the domain passes the regex, hash with blake2b. Otherwise, throw err. 
var node = pns.namehash("Bob.dot");
pns.register(node).then((event) => {
    if event.status == "registered" {
        console.log("the name \"Bob.dot\" has been registered by" + account.address);
    }
})
```

the owner can renew the domain before it expires with the price he or she bought. 

The auction starts when anyone who want the domain claim auction after ttl from registered date passes or when the owner wants to sell it.

The resolver will record the highest bidder only.

Bidding is available before the auction is closed.

The auction will finalize when anyone submits a request to finalize auction after the auction closing date.

The recorded highest bidder will take the domain and register it new with the new price and have it for one year. 

## Name Syntax

// TODO: I am making a vote to find out which form would be better: email? wechat? riot? website?
PNS names must conform to the following syntax:

```
<domain> ::= <label> | <domain> "." <label>
<label> ::= 
```


## Namehash algorithm

Namehash algorithm in Polkadot does recursive hashing with blake2b split with ".".

the output of the namehash algorithm is referred to as a 'node' as ENS does.

Pseudocode for the namehash algorithm is as follows:

```
def namehash(name):
    if name == '':
        return '\0' * 32
    else:
        label, _, remainder = name.partittion('.')
        return blake2(blake2(remainder) + blake2(label))
```

## Resolver specification

The resolver module hash the following functions

```
fn register_domain(origin, domain_hash: T::Hash) -> Result
```

registers domain in the storage as namehash as key and information as value
and emits event `DomainRegistered`

```
fn resolve(origin, domain_hash: T::Hash) -> Result
```

resolves the domain and emits event `DomainResolved`

```
fn renew(origin, domain_hash: T::Hash) -> Result
```

renews the domain and emits event `DomainRenewal`

```
fn claim_auction(origin, domain_hash: T::Hash) -> Result 
```

claims the name to be set in auction

```
fn new_bid(origin, domain_hash: T::Hash, bid: T::Balance) -> Result
```

bids to the name which is in auction until the auction finalizes

```
fn finalize_auction(origin, domain_hash: T::Hash) -> Result
```

finalizes auction to select the winner and transfer the domain


### Appendix A: Resolver Implementation
```rust
use support::{decl_module, decl_storage, decl_event, dispatch::Result, ensure};
use support::traits::{Currency, WithdrawReason, ExistenceRequirement};
use system::{ensure_signed};
use codec::{Encode, Decode};
use rstd::prelude::*;

// 1 year in seconds
const YEAR: u32 =  31556952;

#[derive(Encode, Decode, Default, Clone, PartialEq)]
pub struct Domain<AccountId, Balance, Moment> {
	source: AccountId,
	price: Balance,
	ttl: Moment,
	registered_date: Moment,
	available: bool,
	highest_bid: Balance,
	bidder: AccountId,
	auction_closed: Moment,
}


/// The module's configuration trait.
pub trait Trait: system::Trait + balances::Trait + timestamp::Trait {
	/// The overarching event type.
	type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
	
}


// This module's storage items.
decl_storage! {
	trait Store for Module<T: Trait> as NamingServiceModule {
		/// Total number of domains
		Domains get(total_domains): u64;
		/// Hash is the blake2b hash of the domain name
		/// In Javascript, use @polkadot/util-crypto's blake2AsHex("<domain name you want>" 256) and put the hexstring in the polkadot.js apps param.
		/// Or use blakejs with this example.
		/// > var blake = require('blakejs');
		/// > console.log(blake.blake2sHex('hyungsukkang.dot', 256))
		/// fecf3628563657233c1d29fd6589bcb792d1ce7611892490c3dd5857647006d7
		Resolver get(domain): map T::Hash => Domain<T::AccountId, T::Balance, T::Moment>;
	}
}

// The module's dispatchable functions.
decl_module! {
	/// The module declaration.
	pub struct Module<T: Trait> for enum Call where origin: T::Origin {
		// Initializing events
		// this is needed only if you are using events in your module
		fn deposit_event() = default;
		
		fn offchain_worker(_now: T::BlockNumber) {
			#[cfg(feature = "std")]
			Self::validate_format();
		}
		
		// Register domain with 1 year ttl(31556926000 milliseconds) and 1 milli DEV(0.001 DEV) base price
		pub fn register_domain(origin, domain_hash: T::Hash, test: Vec<u8>) -> Result {
			let sender = ensure_signed(origin)?;
			ensure!(Self::validate_format(test) == true, "Test failed");
			ensure!(!<Resolver<T>>::exists(domain_hash), "The domain already exists");
			// Convert numbers into generic types which codec supports
			// Generic types can process arithmetics and comparisons just as other rust variables
			let ttl = Self::to_milli(T::Moment::from(YEAR));
			let init_price = Self::to_balance(1, "milli");
			let reg_date: T::Moment = <timestamp::Module<T>>::now();
			
			// Try to withdraw registration fee from the user without killing the account
			let _ = <balances::Module<T> as Currency<_>>::withdraw(&sender, init_price, WithdrawReason::Reserve, ExistenceRequirement::KeepAlive)?;			

			// make new Domain struct
			let new_domain = Domain{
				source: sender.clone(),
				price: init_price,
				ttl: ttl,
				registered_date: reg_date,
				available: false,
				highest_bid: T::Balance::from(0),
				bidder: sender.clone(),
				auction_closed: T::Moment::from(0)
				};

			// Insert new domain to the Resolver state
			<Resolver<T>>::insert(domain_hash, new_domain);

			// Increment domain number	
			let mut domains = Self::total_domains();
			domains = domains.wrapping_add(1);

			// Store domain number to Domains state
			Domains::put(domains);			

			// Deposit event
			Self::deposit_event(RawEvent::DomainRegistered(sender.clone(), init_price, ttl, reg_date));
			
			Ok(())
		}

		pub fn resolve(origin, domain_hash: T::Hash) -> Result {
			ensure!(<Resolver<T>>::exists(domain_hash), "The domain does not exist");
			let domain = Self::domain(domain_hash);
			Self::deposit_event(RawEvent::DomainResolved(domain_hash, domain.source));

			Ok(())
		}

		pub fn renew(origin, domain_hash: T::Hash) -> Result {
			let sender = ensure_signed(origin)?;

			let mut new_domain = Self::domain(domain_hash.clone());
			let now = <timestamp::Module<T>>::now();
			// Ensure the sender is the source of the domain and its ttl is not expired
			ensure!(new_domain.source == sender && now < new_domain.registered_date + new_domain.ttl, "You are either not the source of the domain or the domain is expired");
			
			// Extend domain TTL by a year
			let ttl = Self::to_milli(T::Moment::from(YEAR));
			new_domain.ttl += ttl;		

			// Try to withdraw price from the user account to renew the domain 
			let _ = <balances::Module<T> as Currency<_>>::withdraw(&sender, new_domain.price, WithdrawReason::Reserve, ExistenceRequirement::KeepAlive)?;			


			// mutate domain with new_domain struct in the Domain state
			<Resolver<T>>::mutate(domain_hash.clone(), |domain| *domain = new_domain.clone());
			Self::deposit_event(RawEvent::DomainRenewal(domain_hash, sender, new_domain.registered_date + new_domain.ttl));


			Ok(())
		}

		pub fn claim_auction(origin, domain_hash: T::Hash) -> Result {
			let sender = ensure_signed(origin)?;
			// Ensure that
			// Domain does already exist
			ensure!(<Resolver<T>>::exists(domain_hash), "The domain does not exist");
			// But wait, get domain data and time
 			let mut new_domain = Self::domain(domain_hash.clone());
			let now = <timestamp::Module<T>>::now();
			// Ensure the sender is the source of the domain or its ttl is expired
			ensure!(sender == new_domain.source || new_domain.registered_date + new_domain.ttl < now, "You are neither the source of the domain or the claimer after the domain's TTL");

			
			// Set domain available for selling
			new_domain.available = true;

			// Set auction to be closed after 1 hour(60* 60 seconds) * 1000(milliseconds conversion) using timestamp 
			let converted = Self::to_milli(T::Moment::from(3600));
			new_domain.auction_closed = now + converted;

			// mutate domain with new_domain struct in the Domain state
			<Resolver<T>>::mutate(domain_hash.clone(), |domain| *domain = new_domain.clone());
			Self::deposit_event(RawEvent::NewAuction(sender, domain_hash, now, new_domain.auction_closed));


			Ok(())
		}

		
		pub fn new_bid(origin, domain_hash: T::Hash, bid: T::Balance) -> Result {
			let sender = ensure_signed(origin)?;
			// Ensure that
			// Domain does already exist
			ensure!(<Resolver<T>>::exists(domain_hash), "The domain does not exist");
			// But wait, get domain data
			let mut new_domain = Self::domain(domain_hash.clone());
			// The auction is available
			ensure!(new_domain.available, "The auction for the domain is currently not available");
			// The auction is not closed
			let now = <timestamp::Module<T>>::now();
			ensure!(new_domain.auction_closed > now, "The bid for the auction is already closed");
			// The bid price is higher than the current highest bid
			ensure!(new_domain.highest_bid < bid.clone(), "Bid higher");
			

			// Set new domain data
			new_domain.bidder = sender.clone();
			new_domain.highest_bid = bid.clone();
			
			// mutate domain with new_domain struct in the Domain state
			<Resolver<T>>::mutate(domain_hash.clone(), |domain| *domain = new_domain.clone());
			Self::deposit_event(RawEvent::NewBid(sender, domain_hash, bid));

			Ok(())
		}

		pub fn finalize_auction(origin, domain_hash: T::Hash) -> Result {
			let sender = ensure_signed(origin)?; 
			// Ensure that
			// Domain does already exist
			ensure!(<Resolver<T>>::exists(domain_hash), "The domain is not registered yet");
			// But wait, get domain data and time
			let mut new_domain = Self::domain(domain_hash);
			let now = <timestamp::Module<T>>::now();
			// The auction is available
			ensure!(new_domain.available, "The auction for the domain is currently not available");
			// The auction is finalized or the source wants to finalize the auction(test)
			// TEST: If you want to test auction functions without waiting for 1 hour, just add '|| sender == new_domain.source in ensure! macro
			ensure!(now > new_domain.auction_closed, "The auction has not been finalized yet");

			let _ = <balances::Module<T> as Currency<_>>::transfer(&new_domain.bidder, &new_domain.source, new_domain.highest_bid);

			// Set new domain data to bidder as source, highest_bid as price, and reinitialize rest of them 
			new_domain.source = new_domain.bidder.clone();
			new_domain.price = new_domain.highest_bid;
			new_domain.available = false;
			let ttl = Self::to_milli(T::Moment::from(YEAR));
			new_domain.ttl = ttl;
			new_domain.registered_date = now;
			new_domain.available = false;
			new_domain.highest_bid = T::Balance::from(0);
			new_domain.auction_closed = T::Moment::from(0);

			// mutate domain with new_domain struct in the Domain state
			<Resolver<T>>::mutate(domain_hash.clone(), |domain| *domain = new_domain.clone());
			Self::deposit_event(RawEvent::AuctionFinalized(new_domain.bidder, domain_hash, new_domain.highest_bid));

			Ok(())
		}
	}
}

decl_event!(
	pub enum Event<T> where AccountId = <T as system::Trait>::AccountId, <T as system::Trait>::Hash, <T as balances::Trait>::Balance, <T as timestamp::Trait>::Moment
 {
		DomainRegistered(AccountId, Balance, Moment, Moment),
		NewAuction(AccountId, Hash, Moment, Moment), 
		NewBid(AccountId, Hash, Balance),
		AuctionFinalized(AccountId, Hash, Balance),
		DomainResolved(Hash, AccountId),
		DomainRenewal(Hash, AccountId, Moment),
	}
);

// Module's function
impl<T: Trait> Module<T> {

	pub fn to_milli(m: T::Moment) -> T::Moment {
		m * T::Moment::from(1000)
	}

	pub fn to_balance(u: u32, digit: &str) -> T::Balance {
		let power = |u: u32, p: u32| -> T::Balance {
			let mut base = T::Balance::from(u);
			for _i in 0..p { 
				base *= T::Balance::from(10)
			}
			return base;
		};
		let result = match digit  {
			"femto" => T::Balance::from(u),
			"nano" =>  power(u, 3),
			"micro" => power(u, 6),
			"milli" => power(u, 9),
			"one" => power(u,12),
			"kilo" => power(u, 15),
			"mega" => power(u, 18),
			"giga" => power(u, 21),
			"tera" => power(u, 24),
			"peta" => power(u, 27),
			"exa" => power(u, 30),
			"zetta" => power(u, 33),
			"yotta" => power(u, 36),
			_ => T::Balance::from(u)
		}; 
		result 
	}

}
```
## Tests

Current working Substrate runtime module:
[substrate-naming-service](https://github.com/hskang9/substrate-naming-service/blob/master/runtime/src/naming_service.rs)

## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).

TODOs:

- Provide interchain use case
- Compare with blockstack and extend this idea
