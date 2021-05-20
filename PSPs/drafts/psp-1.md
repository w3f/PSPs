# XP.network Protocol

- **PSP Number:** 1
- **Authors:** DimaBryuhanov, dima@xp.network, VKint, kint@xp.network
- **Status:** Draft
- **Created:** 2021-05
- **Reference Implementation**:

[XCMP Pallet example](https://github.com/xp-network/xcmp_pallet-poc/tree/master/xmessage)

[Protocol Blob Serialisation example](https://github.com/xp-network/serde_xp_protocol)

## Summary

Since, using the existing XCMP protocol, it is hard to trace whether an incoming message is related to any previous transaction or request, we are elaborating a protocol that will enable such tracking by the "TopicId", which is especially useful when multiple transactions(TXs) are executed between two blockchains (Lx, Ly). Besides, when a message is sent from one parachain to another the initiating parachain is not kept updated about the progress of a transaction, while our protocol will take care of that. The XP.network Protocol will be supported by a group of pallets, each acting like a “post office” in a post office network. Any parachain, parathread or bridge will be able to attach our pallet and use the protocol for tracking the states of the transactions (TXs) it committed to the other parachains, parathreads or bridged blockchains.

## Motivation

The XP.network protocol allows parachains to communicate in a connectionless but ordered and reliable way. The protocol allows to store the State of a negotiated TX and enables the functionality required to inform the User accordingly.

At the moment, XCMP is a work in progress and SPREE only exists in documentation and will only be developed after the parachains become available. With all the advantages of the two above mentioned protocols, neither of them offers the precision XP.network Protocol does.

Our protocol allows to mark and store the current state of every transaction in a blob with a unique TopicID. Even though the future implementations of XCMP will allow to create channels between two parachains, TopicIDs will still be useful when multiple transactions are being executed between two parachains simultaneously. Another XCMP feature - ordered delivery - does not provide the same precision as the TopicId does. The TopicId does not rely on the order of messages. It relies on the unique identifier which is the safest pointer.
Once the state of a transaction changes or an error occurs, the requesting parachain gets notified. Some of such messages help keep the end user properly informed about the state of events with his/her transaction, others inform the pallets that the transaction has terminated and the memory must be freed from the blob, which might be expensive to store, especially when it is no longer needed.
Another feature uses the fact that a pallet, implementing our protocol, is a part of its parent parachain. Therefore, it has no additional overhead in tracking whether the transaction succeeded or failed. It listens to events related to the transaction and notifies the requesting parachian of the result and provides the data of the outcome when it becomes available without being additionally requested about this information.
As a transport layer of our protocol we’re using the existing version of XCMP. When the new version of XCMP is developed we will migrate our protocol there. Once SPREE becomes available we will move our protocol there, since it will provide more security in its own storage, that a parachain cannot forge or alter. 

## Specification

**XP Relay Chain Protocol** will be supported by a number of pallets, each acting as a “post office” for its parathread. A typical message will include:
```terminal
{
ID:                 id,               //required to identify that the other blockchain’s reply is related to this request,
To:                 dest,             // indicates the destination parachain / parathread,
Payload:            blob              // A binary representation of the "intention"
}
```

The runtime [storage](https://substrate.dev/rustdocs/v3.0.0/frame_support/storage/trait.StorageValue.html#required-methods) & the message inside the binary payload will be structured as follows:

![img](https://github.com/xp-network/w3f_application/blob/main/xp.network_blob.png)

Even the complete message adds only 64 additional bits to the original TX binary code. 16 bits for the TopicId, 16 bits for the flags and 32 bits for the length of the TX binary. In order to reduce the overhead:
 
1. When notifying about errors or that a TX has been submitted for execution only the first 32 bits are attached to the message (16 bits of the TopicId and 16 bits for flags). They will be used to update the blob, where the state of a TX is stored during the number of seconds when a TX is executed in a target parachain. The extrinsics bytecode is not moved when it is not required.
2. Should sometime in the future even such tiny notifications create a noticable overhead, we will </br>a. join them into batches </br>b. remove the AKN notification, which is required till SPREE becomes avaliable to confirm that the message has been received with the same TX code.

The **XP.network Decision Tree**, regulating the efficiency of the data flow between the two pallets, will roughly look like this:

![img](https://github.com/xp-network/w3f_application/blob/main/XP.network%20Protocol-3.png)

The above scheme is a work in progress and subject to change.

Apart from standard setup, a pallet implementing XP.network Relay Chain Protocol consists of:

1. **Message Listener** - it listens to the incoming messages and passes them to the Decision Tree.
2. **Message Deserialiser** - it reads the contents of the binary file and populates the fields of the Message struct.
3. **Message Serialiser** - it packs the values of the Message struct into a binary representation.
4. **Message Sender** - it uses the Relay Chain callback mechanism to communicate with the other parachains and parathreads using XP Network protocol.
5. **Runtime Storage** - it stores the binaries with the current state of the corresponding transaction. Each blob can be accessed like so: ```sender[TopicId]```.
6. **Decision Tree** - it controls the efficiency of the data flow between the pallets.


## Tests

Run our projects like so:
```
cargo test -- --nocapture
```

## Copyright

Public Domains:

[PoC Documentation](https://xp-network.github.io/poc-documentation/) Method names, parameters with types, return types and description.

[XCMP Pallet example](https://github.com/xp-network/xcmp_pallet-poc/tree/master/xmessage)

[Protocol Blob Serialisation example](https://github.com/xp-network/serde_xp_protocol)
