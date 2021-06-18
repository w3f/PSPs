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
If someone registers a domain which did not exist before, they get the domain by paying the initialization price (1 milli dot) and have it for 1 year.
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

The auction starts when anyone who wants the domain claims an auction after `ttl` from registered date passes or when the owner wants to sell it.

The resolver will record the highest bidder only.

Bidding is available before the auction is closed.

The auction will finalize when anyone submits a request to finalize auction after the auction closing date.

The recorded highest bidder will take the domain and register it with the new price and have it for one year. 

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


### Appendix A: Runtime Implementation
```rust
use support::{decl_module, decl_storage, decl_event, dispatch::Result, ensure};
use support::traits::{Currency, WithdrawReason, ExistenceRequirement};
use system::{ensure_signed};
use codec::{Encode, Decode};
use rstd::prelude::*;

// 1 year in blockseconds
// each block is assumed to be generated in 6 seconds. divide that with 31556952(1 year) seconds and you get 5259492 blocks to represent 1 year in blockchain. 
const YEAR: u32 =  5259492;
pub type IPV4 = [u8; 4];
pub type IPV6 = [u16; 6];
pub type BYTES = Vec<u8>;

#[derive(Encode, Decode, Default, Clone, PartialEq)]
pub struct Domain<AccountId, Balance, BlockNumber> {
	/// domain name in bytestring e.g. b'hyungsukkang.eth'
	name: BYTES,
	/// source of this domain a.k.a. the address of the blockchain
	source: AccountId,
	/// the current domain price
	price: Balance,
	/// Time to claim the ownership
	ttl: BlockNumber,
	/// Registered date in block height
	registered_date: BlockNumber,
	/// whether it is available for purchase or sale
	available: bool,
	/// highest bid in the auction stage
	highest_bid: Balance,
	/// bidder who bidded highest
	bidder: AccountId,
	/// Auction closing date
	auction_closed: BlockNumber,

	/// TODO: Try to make browser engine which asks for this with Servo fork
	/// IPV4 in case where the owner wants to put IP address
	ipv4: IPV4,
	/// IPV6 in case where the owner wants to put IP address for his or her IoT device 
	ipv6: IPV6,
}

#[derive(Encode, Decode, Default, Clone, PartialEq)]
pub struct DataPoint<AccountId> {
	/// Array of accounts that are able to get access to the data point
	access: Vec<AccountId>,
	/// Whether the data is public and can be shown to anyone
	public: bool,
	/// Encrypted/Decrypted data in byte array(e.g. b"twt://@{twitter username}", b"ipfs://{some IPFS hash}", b"jpg://{some blob of image}" )
	data: BYTES,
}

// Module's function and Methods of custom struct to be placed here
impl<T: Trait> Module<T> {

	pub fn new_domain(domain_name: BYTES, source: T::AccountId) -> Domain<T::AccountId, T::Balance, T::BlockNumber> {
		// Convert numbers into generic types which is mapped to native type in lib.rs
		// Generic types can process arithmetics and comparisons just as other rust variables
		let ttl = T::BlockNumber::from(YEAR);
		let init_price = Self::to_balance(1, "milli");
		let reg_date: T::BlockNumber = <system::Module<T>>::block_number(); 
		
		Domain {
			name: domain_name,
			source: source.clone(),
			price: init_price,
			ttl: ttl,
			registered_date: reg_date,
			available: false,
			highest_bid: T::Balance::from(0),
			bidder: source,
			auction_closed: T::BlockNumber::from(0),
			ipv4: [0,0,0,0],
			ipv6: [0,0,0,0,0,0],
		}
	}

	// TODO: Add this to <balances::Module<T>> and test with u128
	/// Convert u32 to u128 generic type Balance type
	pub fn to_balance(u: u32, digit: &str) -> T::Balance {
		// Power exponent function
		let pow = |u: u32, p: u32| -> T::Balance {
			let mut base = T::Balance::from(u);
			for _i in 0..p { 
				base *= T::Balance::from(10)
			}
			return base;
		};
		let result = match digit  {
			"femto" => T::Balance::from(u),
			"nano" =>  pow(u, 3),
			"micro" => pow(u, 6),
			"milli" => pow(u, 9),
			"one" => pow(u,12),
			"kilo" => pow(u, 15),
			"mega" => pow(u, 18),
			"giga" => pow(u, 21),
			"tera" => pow(u, 24),
			"peta" => pow(u, 27),
			"exa" => pow(u, 30),
			"zetta" => pow(u, 33),
			"yotta" => pow(u, 36),
			_ => T::Balance::from(u),
		}; 
		result 
	}

	pub fn remove_domain(domain_hash: T::Hash, domains: Vec<T::Hash>) -> Vec<T::Hash> {
		let mut new_reverse_list: Vec<T::Hash> = vec!{};

		for i in domains {
			if i != domain_hash {
				new_reverse_list.push(i);
			}
		}

		return new_reverse_list;
	}
}


/// The module's configuration trait.
pub trait Trait: system::Trait + balances::Trait {
	/// The overarching event type.
	type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
	
}


// This module's storage items.
decl_storage! {
	trait Store for Module<T: Trait> as NameServiceModule {
		/// Total number of domains
		Domains get(total_domains): u64;
		/// Hash is the blake2b hash of the domain name
		/// In Javascript, use @polkadot/util-crypto's blake2AsHex("<domain name you want>" 256) and put the hexstring in the polkadot.js apps param.
		/// Or use blakejs with this example.
		/// > var blake = require('blakejs');
		/// > console.log(blake.blake2s('hyungsukkang.dot'))
		/// fecf3628563657233c1d29fd6589bcb792d1ce7611892490c3dd5857647006d7
		Resolver get(domain): map T::Hash => Domain<T::AccountId, T::Balance, T::BlockNumber>;
		/// Reverse resolver for account => domain_hash
		Reverse get(account): map T::AccountId => Vec<T::Hash>;
	}
}

// The module's dispatchable functions.
decl_module! {
	/// The module declaration.
	pub struct Module<T: Trait> for enum Call where origin: T::Origin {
		// Initializing events
		// this is needed only if you are using events in your module
		fn deposit_event() = default;

////////////////////////////////////////////////////////////////////////////////////////////////
/// domain and reverse logics //////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////	
			
		/// Register domain with estimated 1 year ttl blocktime(31556926000 milliseconds) and 1 milli DEV(0.001 DEV) base price
		pub fn register_domain(origin, domain_hash: T::Hash, domain_name: BYTES) -> Result {
			let sender = ensure_signed(origin)?;
			ensure!(!<Resolver<T>>::exists(domain_hash), "The domain already exists");
			// Make new Domain struct
			let new_domain = Self::new_domain(domain_name, sender.clone());

			// Try to withdraw registration fee from the user without killing the account
			let _ = <balances::Module<T> as Currency<_>>::withdraw(&sender.clone(), new_domain.price, WithdrawReason::Reserve, ExistenceRequirement::KeepAlive)?;			

			<Reverse<T>>::insert(sender.clone(), vec![domain_hash]);
			
			// Insert new domain to the Resolver state
			<Resolver<T>>::insert(domain_hash, new_domain.clone());

			// Increment domain number	
			let mut domains = Self::total_domains();
			domains = domains.wrapping_add(1);

			// Store domain number to Domains state
			Domains::put(domains);			

			// Deposit event
			Self::deposit_event(RawEvent::DomainRegistered(sender.clone(), new_domain.price, new_domain.ttl, new_domain.registered_date));
			
			Ok(())
		}

		/// Set IPV4 for existing domain
		pub fn set_ipv4(origin, domain_hash: T::Hash, ipv4: IPV4) -> Result {
			// Ensure that 
			// domain exists
			ensure!(<Resolver<T>>::exists(domain_hash), "The domain does not exist");
			// the sender is the source of the domain
			let sender = ensure_signed(origin)?;
			let mut new_domain = Self::domain(domain_hash);
			ensure!(sender == new_domain.source, "you are not the source of the domain");
			
			// Set ipv4 for new domain
			let old_ipv4 = new_domain.ipv4;
			new_domain.ipv4 = ipv4;

			// Change domain data with the new one and emit event
			<Resolver<T>>::mutate(domain_hash.clone(), |d| *d = new_domain.clone());
			Self::deposit_event(RawEvent::SetIPV4(domain_hash, old_ipv4.to_vec(), new_domain.ipv4.to_vec()));

			Ok(())
		}

		pub fn resolve(_origin, domain_hash: T::Hash) -> Result {
			ensure!(<Resolver<T>>::exists(domain_hash), "The domain does not exist");
			let domain = Self::domain(domain_hash);
			Self::deposit_event(RawEvent::DomainResolved(domain_hash, domain.source, domain.price, domain.available, domain.highest_bid, domain.bidder, domain.auction_closed));

			Ok(())
		}

		pub fn renew(origin, domain_hash: T::Hash) -> Result {
			let sender = ensure_signed(origin)?;

			let mut new_domain = Self::domain(domain_hash.clone());
			let now = <system::Module<T>>::block_number();
			// Ensure the sender is the source of the domain and its ttl is not expired
			ensure!(new_domain.source == sender && now < new_domain.registered_date + new_domain.ttl, "You are either not the source of the domain or the domain is expired");
			
			// Extend domain TTL by a year
			let ttl = T::BlockNumber::from(YEAR);
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
			let now = <system::Module<T>>::block_number();
			// Ensure the sender is the source of the domain or its ttl is expired
			ensure!(sender == new_domain.source || new_domain.registered_date + new_domain.ttl < now, "You are neither the source of the domain or the claimer after the domain's TTL");

			
			// Set domain available for selling
			new_domain.available = true;

			// Set auction to be closed after 1 hour(60* 60 seconds) * 1000(milliseconds conversion) using timestamp 
			let converted = T::BlockNumber::from(600);
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
			// The auction is not finalized
			let now = <system::Module<T>>::block_number();
			ensure!(new_domain.auction_closed > now, "The bid for the auction is already finalized");
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
			let _sender = ensure_signed(origin)?; 
			// Ensure that
			// Domain does already exist
			ensure!(<Resolver<T>>::exists(domain_hash), "The domain is not registered yet");
			// But wait, get domain data and time
			let mut new_domain = Self::domain(domain_hash);
			let now = <system::Module<T>>::block_number();
			// The auction is available
			ensure!(new_domain.available, "The auction for the domain is currently not available");
			// The auction is finalized or the source wants to finalize the auction(test)
			// TEST: If you want to test auction finalization without waiting for 1 hour, just add '|| sender == new_domain.source in ensure! macro
			ensure!(now > new_domain.auction_closed, "The auction has not been finalized yet");

			let _ = <balances::Module<T> as Currency<_>>::transfer(&new_domain.bidder, &new_domain.source, new_domain.highest_bid);


			let ttl = T::BlockNumber::from(YEAR);

			// Remove domain hash from the prior owner's reverse registrar
			let old_reverse = Self::account(new_domain.source.clone());
			
			let new_reverse = Self::remove_domain(domain_hash.clone(), old_reverse);

			// Mutate reverse with new_reverse arrray in the Reverse state
			<Reverse<T>>::mutate(new_domain.source.clone(), |account| *account = new_reverse.clone());
		
			// Set reverse for the new owner
			// if the account is in reverse registrar
			if <Reverse<T>>::exists(new_domain.bidder.clone()) {
				let mut new_reverse: Vec<T::Hash> = Self::account(new_domain.bidder.clone());
				new_reverse.push(domain_hash.clone());
				// Mutate reverse with new_reverse arrray in the Reverse state
				<Reverse<T>>::mutate(new_domain.bidder.clone(), |reverses: &mut Vec<T::Hash>| *reverses = new_reverse.clone());
			} else {
				let new_reverse = vec![domain_hash];
				<Reverse<T>>::insert(new_domain.bidder.clone(), new_reverse.clone());
			}

			// Set new domain data to bidder as source, highest_bid as price, and reinitialize rest of them 
			new_domain.source = new_domain.bidder.clone();
			new_domain.price = new_domain.highest_bid;
			new_domain.available = false;
			new_domain.ttl = ttl;
			new_domain.registered_date = now;
			new_domain.available = false;
			new_domain.highest_bid = T::Balance::from(0);
			new_domain.auction_closed = T::BlockNumber::from(0);



			// Mutate domain with new_domain struct in the Domain state
			<Resolver<T>>::mutate(domain_hash.clone(), |domain| *domain = new_domain.clone());
			
			Self::deposit_event(RawEvent::AuctionFinalized(new_domain.bidder, domain_hash, new_domain.highest_bid));

			Ok(())
		}

		pub fn reverse_resolve(_origin, account_id: T::AccountId) -> Result {
			ensure!(<Reverse<T>>::exists(account_id.clone()), "The account have not registered or owned any domain");
			let domains = Self::account(account_id.clone());
			Self::deposit_event(RawEvent::ReverseResolved(account_id, domains));

			Ok(())			
		}
	}
}

decl_event!(
	pub enum Event<T> where AccountId = <T as system::Trait>::AccountId, <T as system::Trait>::Hash, <T as balances::Trait>::Balance, <T as system::Trait>::BlockNumber
 {
		DomainRegistered(AccountId, Balance, BlockNumber, BlockNumber),
		SetIPV4(Hash, Vec<u8>, Vec<u8>),
		NewAuction(AccountId, Hash, BlockNumber, BlockNumber), 
		NewBid(AccountId, Hash, Balance),
		AuctionFinalized(AccountId, Hash, Balance),
		DomainResolved(Hash, AccountId, Balance, bool, Balance, AccountId, BlockNumber),
		ReverseResolved(AccountId, Vec<Hash>),
		DomainRenewed(Hash, AccountId, BlockNumber),
	}
);
```
## Tests

Current working Substrate runtime module:
[substrate-name-service](https://github.com/hskang9/substrate-name-service/blob/blocknumber/runtime/src/name_service.rs)

## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).

TODOs:

- Provide interchain use case
- Compare with blockstack and extend this idea
