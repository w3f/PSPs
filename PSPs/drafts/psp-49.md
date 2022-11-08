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

Two version would be present initialy, substrate trie v0 as version number 0 and substrate trie v1 as version number 1.

The proof is afterward a sequence of instruction with an optional terminal end node instruction.

This versioning is not currently strictly necessary, most proof are being use in a well known version context, but this is
good for extensibility and ensure no ambiguity.


Inline node: for simplicity they are treated as standard content, data will be inline back when building.

### Instruction form

Version: the instruction is optional, this is something that can is known and fix in many context.

End node: the instruction is optional, this acts as a separator when we do not know the end of a proof. Semantically it is the same as using instruction pop up to root.

### Algorithm and limits

Some 
### Encoding

TBD currently the implementation just use scale encoded, but since there is few variant a bit can be gain

The first byte contains the version. Number of node is not specified, an optional end node is used instead.u

Rest of node is non ambiguous and should only have a single sequence form.


## Tests

If applicable, please include a list of potential test cases to validate an implementation.

## Copyright

This PSP is placed in the [public domain](https://creativecommons.org/publicdomain/zero/1.0/).
