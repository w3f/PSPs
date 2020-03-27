# PSP-2: Polkadot DKG threshold multisig wallet

* **PSP Number:** 2
* **Author(s):** Everstake
* **Status:** PoC
* **Created:** [2020-02-13]
* **Reference Implementation** [-]


## Summary
Multisignature (multisig) refers to requiring more than one key to authorize a transaction. It is generally used to divide up responsibility for possession of assets. Using a multisig wallet users are able to prevent the loss or theft of their private keys. But still if one of the keys is compromised, the funds are safe.
A multisig wallet can be implemented using cryptographic methods or non cryptographic ones with the help of smart contracts. With cryptography there are also couple of ways to do it:
 - Shamir Secret Sharing
 - Verifiable Secret Sharing
 - Naive multisignatures(like in Bitcoin)
 - Aggregated signatures
 - Distributed Key Generation

Distributed Key Generation (DKG) is a way for a group of nodes to collectively agree on a public/private key pair without any single party knowing the private key. Everyone just knows the public key. 
This is actually very hard to achieve but it relies on the fact that lagrange interpolated shares are homomorphic (in that operations can be performed on shares even without knowing the full value). For example, you can add A{share1}+B{share1} to get C{share1} that you can add to someone else’s C{share2} to get the full value of C (assuming A and B were split into 2 shares each).


## Motivation
So far Polkadot has two implementations: one is a non-cryptographic multisignature pallet and the other is the cryptographic Schnorrkel multisignature implementation that sits in the Polkadot Host. The goal is to expand the list of multsignature options within Polkadot and to eventually agree upon a standard. Eventually this will allow for the creation of wallets that have robust and well-reviewed code. 

If possible, it would be ideal to investigate the possibilities of standarising the use of DKG, such that a wallet can take advantage of DKG for the creation of Threshold Multisignature addresses. 

As compared to Shamir Secret Sharing or Verifiable Secret Sharing, by using DKG we avoid a single point of failure problem since all the participants generate their keys themselves and no one, except for a creator knows them.


## Specification
For the key generation we want to implement the Rabin DKG protocol. That’s a secure protocol for distributed key generation in discrete-log based cryptosystems with threshold *t*, for any *t < n/2*.

For signing we decided to use Distributed Schnorr Signature(DSS) which also gives us threshold functionality.

Here are links on papers to learn more about these protocols:
 - DKG - https://www.researchgate.net/publication/227327292_Secure_Distributed_Key_Generation_for_Discrete-Log_Based_Cryptosystems
 - DSS - http://cacr.uwaterloo.ca/techreports/2001/corr2001-13.ps

To implement these protocols we can have asynchronous communication but with adding some term bound to sending messages to avoid situations when transaction is stuck because of one participant is offline during days. We can achieve this with a web server through which participants will exchange information and where will store wallet and transactions statuses.

All the commitments and partial signatures will go through the server, so the web server in our case is like a bridge between all the participants. Since the server doesn’t hold any private keys or other secret information it won’t reveal a distributed secret key even if it’s corrupted. All the secret information will be stored on the client side.

In general procedure will look like this:
 1. Everyone generate own priv and pub keys;
 2. All participants exchange pub keys to everyone have list of all participant’s pub keys;
 3. Users start to generate general distributed secret key by broadcasting commitments and responses;
 4. When general secret is generated and everyone have pub key of secret, users can create multisig transaction.

To make communication between users secure we will use AES256-GCM encryption to encode some messages.

Here is sequence diagram to see all the steps in distributed key generating process. Suppose that wallet client connect to web-server through websocket and continuously receive new status. "Rust DKG lib" on diagram - Rust code compiled to WebAssembly to use it in JS. Also there are on diagrams "found message" symbols, it means that Wallet APP got new status from web-server through the websocket.

![](sequence_wallet_diagram.png)

To sign transactions we will use the DSS protocol that was mentioned above. It’s secure 3 - round Schnorr signature scheme where signature is creating distributively. Final signature is compatible with the EdDSA verification function against the distributed secret key. When transaction is signed there are will be couple of options to send it. Transaction can be send by the last partisipant who add his partial signature or by the first who initiate transaction.

![](dss_sequence_diagram.png)

## Web-server wallet API

Here is short web-server's API specification. Some endpoints can be changed with time.

Version: 1.0.0

### /dkg/wallet/new

#### POST
##### Summary:

Create new multisig wallet
***
### /dkg/wallet/join

#### POST
##### Summary:

Join to multisig wallet
***
### /dkg/deal/broadcast

#### POST
##### Summary:

Broadcast Deal for some participant
***
### /dkg/deal/response

#### POST
##### Summary:

Broadcast Response of processed Deal
***
### /dkg/deal/respose/complaint

#### POST
##### Summary:

Complain on Deal/participant
***
### /dkg/secretCommit/broadcast

#### POST
##### Summary:

Broadcast secretCommit for participants
***
### /dkg/secretCommit/response

#### POST
##### Summary:

Broadcast complain of secretCommit if something is wrong
***
### /dss/transaction/new

#### POST
##### Summary:

Create(initiate) new multisig transaction
***
### /dss/transaction/status/randPub/generated

#### POST
##### Summary:

Announce that random key was generated and send to server pub key
***
### /dss/transaction/status/randKey/generated

#### POST
##### Summary:

Announce that random key was generated with DKG protocol
***
### /dss/transaction/partSignature/broadcast

#### POST
##### Summary:

Broadcast partial signature that was generated with DSS protocol
***
### /dss/transaction/partSignature/check

#### POST
##### Summary:

Broadcast check of partial signature
***
### /dss/transaction/status/finalSignature/reconstruct

#### POST
##### Summary:

Broadcast status of reconstruction final signature
***
### /dss/transaction/status/sent

#### POST
##### Summary:

Announce that transaction was sent
***
## Tests
*If applicable, please include a list of potential test cases to validate an implementation.*

## Copyright

Each PSP must be labeled as placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).

## Have feedback?

You can let us know what you think using one of the channels below:

Email: inbox@everstake.one

Telegram: @vit_park

Riot: @mrvatka32:matrix.org 