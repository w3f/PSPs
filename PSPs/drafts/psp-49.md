# Title

- **PSP Number:** 49
- **Authors:** cheme
- **Status:** Draft
- **Created:** 2022-11-07
- **Reference Implementation** https://github.com/cheme/trie/tree/compact2

## Summary

A compact merkle proof format for merkle radix trie.

This is for replacing current so called compact proof for the substrate patricia merkle trie.
Current proof is strongly related to codec and if the original purpose was no specific encoding,
recent change to the trie did break this rules slightly.
Additionaly, from my recent conversation the proof format is not easy to explain and even if can
require very small implementation, this is mainly true for some given implementation only, and can
feel rather conter productive for others.

## Motivation

Merkle proof contains a subset of a merkle trie. Therefore they can just be

Proof should only contains the needed content to build this state trie subset.

As a radix tree the algorithm should only really need a single stack of tree nodes in memory and
could run over a stream of content.

## Specification

The proof is a sequence of encoded instructions.

The instructions are describing the different element of the partial trie we want to build.

Ordering is following a lexicographical ordered iteration on trie nodes.

The proof can be build from this iteration by only keeping in memory a stack of nodes and a cursor to the current key.

Some instruction are added to the proof when the node is first reach (value and valuehash), some over instruction are
added only when the node leave the stack (children hash).

When decoding, the stream of instruction, one can build the trie or just check the proof by creating all node in a similar
order (nodes are only processed when leaving the stack).

Two instruction are optionally used, the `Version` as the first element and the `EndProof`.

The proof is afterward a sequence of instruction with an optional terminal end node instruction.



Inline node: for simplicity they are treated as standard content, data will be inline back when building.

### Instruction for sequence

#### Version

Two version would be present initialy, substrate trie v0 as version number 0 and substrate trie v1 as version number 1.

The instruction is optional, this is something that can be already known and static in many context.

But this is good for extensibility and ensure no ambiguity.

A variant of these rule is to always have a default version (could be the fist version declared) and only allow `Version` when it differs from it (and use `EndProof` otherwhise).

##### Action

Indicate which variant to use for the proof, currently all version are compatible.

##### Sequence

Can be in first stream position.

After `EndProof` or another `Version`, previous proof is empty trie.

In other position it indicates the start of another proof sequence.

If last item of the stream this is an error.

#### EndProof

It can be seen as a special `Version` that contains no element.

##### Action

The instruction is optional, this is for indicating the end of a proof when multiple contents are on a stream.

##### Sequence

Can be in first stream position (empty trie).

After `EndProof` or another `Version`, previous proof is empty trie.

### Instruction for single proof

##### KeyPush

With current trie it is part of the key we should add to the current cursor position.
It contains the key nibbles to add.

##### Action

##### Sequence


### Misc

#### Empty trie

Proof on empty trie contains no nodes.

#### Proof that do not touch state

Proof contains the root node (allowing to calculate root hash).

### Encoding

TBD currently the implementation just use scale encoded, but since there is few variant a bit can be gain

The first byte contains the version. Number of node is not specified, an optional end node is used instead.u

Rest of node is non ambiguous and should only have a single sequence form.

##### KeyPush

With current versions, we write two nibbles per bytes and if the size is unaligned, the last nibble must be 0 (decoding must fail otherwhise).

TODO probably size info in the first byte with some extensible scheme as in trie codec

## Tests

If applicable, please include a list of potential test cases to validate an implementation.

Script for a matching default version and being the only element of the stream will be:




Sequence of proof will be, for a default version 0 (first line), for undefined default version (second line), C being the proof instruction (all but version or end), with a sequence of:

TODO having two alternative does not look proper (could make the default a first call to Version and allow using C only as a subset, single compact).

- v0, v1, empty0, v0

C, Version(1), C, EndProof, EndProof, C

Version(0), C, Version(1), C, Version(0), Version(0), C

- empty0, v0, v1, empty0

EndProof, C, Version(1), EndProof

Version(0), Version(0), C, Version(1), Version(0) 

- empty0, v1

EndProof, Version(1)

Version(0), Version(1)

- empty1, v1

Version(1), Version(1)

Version(1), Version(1)


Anything else being invalid, notice that if default is 0, there will never be a Version(0) instruction.

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
