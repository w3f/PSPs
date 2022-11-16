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

Value are written as soon as seen in iteration (when entering the node), while sibling hash of a node are only written after iterating on all its children (when exiting the node).

Rational for writing hash when exiting the node is to have a single stack of node in memory when building node.

Rational for writing value immediately is to be able to stream values when decoding the proof in a lexicographical order.

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

Cannot be after `KeyPop` or `KeyPush`.

#### EndProof

It can be seen as a special `Version` that contains no element.

##### Action

The instruction is optional, this is for indicating the end of a proof when multiple contents are on a stream.

##### Sequence

Can be in first stream position (empty trie).

After `EndProof` or another `Version`, previous proof is empty trie.

Cannot be after `KeyPop` or `KeyPush`.

### Instruction for single proof

The stack contains item that are simply node content: an optional value (with versioning variant) and n optional hashes (hashes can be from the proof instruction or from the processing of a children node in proof), n being the arity of the trie.

#### KeyPush

With current trie it is part of the key we should add to the current cursor position.
It contains the key nibbles to add.

##### Action

Append the key content to the cursor.
Add an empty node to the stack.

##### Sequence

Cannot be after another `KeyPush` (two consecutive `KeyPush` are invalid).

#### KeyPop

Contains a number of nibble to remove from the current cursor.

Same action append at end of stream or (`EndProof` or `Version) with number
of nibble equal to current cursor size but including root node in nodes to enact.

When removing nibbles, nodes present between last cursor position and new cursor position
are enacted (they are guaranted to contains all attached values and hash).

##### Action

While current cursor length bigger than target cursor length, remove last recorded item on stack and enact it.

Enacting means encoding the node as a branch or a leaf (leaf when no children) and calculate hash. Possibly the process can use this enacting step to build a in memory trie or do other pre computation.

The hash is then added to the last item of the stack at its matching position.

Except if no item are on stack at target depth, in this case, add an empty item to stack and add last enacted root to it.

##### Sequence

`KeyPop` cannot be after another `KeyPop` (no consecutive `KeyPop`).

`KeyPop` is not allowed to be the last item of a stream (or before `Version` or `EndProof`).

#### Value

Contains a value at the current cursor position.

##### Action

If stack is empty add an empty node at depth 0. This will only append for `Value` as first node (value in a root node with no partial key).

On last item from stack, set the value.

##### Sequence

Either First node (or after `EndProof` or version) or after a `KeyPush`.
Invalid otherwhise.

### ValueForceInline

Same as value, but indicate this is always attached to the node.
This allow to do proof of trie with both node in version 0 and version 1.
That for a current version 1 a value with big length that was previously written in version 0 can stay in version 0 when reading the proof.

This should be encoded in a non efficient way as it is an exception.

This could be remove in future version of the scheme.

### ValueForceHashed

Act as `ValueForceInline`, but force an detached node when it should not have been.
(eg if reverting to state 0 from state 1 or if using a diffrent hashing threshold for node value).

This should be encoded in a non efficient way as it is an exception.

This could be remove in future version of the scheme.

#### HashChild

Contains a hash and its index in branch.

##### Action

If stack is empty add an empty node at depth 0. This will only append for `HashChild` as first node (and proof containing only root node, root node being a branch with no partial key).

Add hash to children in last item from stack.

If there is already a hash in the stack item (either duplicate definition or children content present in proof).

##### Sequence

When after another `HashChild`, the child index must be bigger than for the previous `HashChild`.

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

### Instruction sequence:

For a trie (version 1) build from (key, values):
```
(b"alfa", &[0; 32]),
(b"bravo", b"bravo"),
(b"do", b"verb"),
(b"dog", b"puppy"),
(b"doge", &[0; 33]),
(b"horse", &[1; 33]),
(b"house", b"building"),
```
Resulting in the following structure:
```
Branch { slice: 6, children: [Leaf { index: 1, slice: 6'c'6'6'6'1, value: Hash([41, 13, 236, 217, 84, 139, 98, 168, 214, 3, 69, 169, 136, 56, 111, 200, 75, 166, 188, 149, 72, 64, 8, 246, 54, 47, 147, 22, 14, 243, 229, 99]) }, Leaf { index: 2, slice: 7'2'6'1'7'6'6'f, value: Inline([98, 114, 97, 118, 111]) }, Branch { index: 4, slice: 6'f, children: [Branch { index: 6, slice: 7, children: [Leaf { index: 6, slice: 5, value: Hash([243, 154, 134, 159, 98, 231, 92, 245, 240, 191, 145, 70, 136, 166, 178, 137, 202, 242, 4, 148, 53, 216, 230, 140, 92, 94, 109, 5, 228, 73, 19, 243]) }], value: Inline([112, 117, 112, 112, 121]) }], value: Inline([118, 101, 114, 98]) }, Branch { index: 8, slice: 6'f'7, children: [Leaf { index: 2, slice: 7'3'6'5, value: Hash([217, 220, 23, 8, 219, 226, 132, 166, 177, 126, 66, 193, 193, 176, 3, 255, 40, 226, 183, 201, 226, 44, 153, 61, 169, 250, 213, 191, 111, 27, 138, 203]) }, Leaf { index: 5, slice: 7'3'6'5, value: Inline([98, 117, 105, 108, 100, 105, 110, 103]) }], value: None }], value: None }

TODO try some ascii from it.

```

For given key part of proof:
```
	b"do", b"dog", b"doge", b"bravo",
```
And given accesses done during proof:
```
	b"d",
	b"do\x10",
	b"halp",
```

The instruction sequence will be:
```
KeyPush: b"bravo" all nibbles (10)
Value: b"bravo"
KeyPop: 9
KeyPush: b"do" without first nibble (3)
Value: b"verb"
KeyPush: b"g" all nibbes (2)
Value: b"puppy"
KeyPush: b"e" all nibbes (2)
Value: [0; 32]
KeyPop: 7
KeyPush: b"house" skipping first nibble (5)
Value: b"building" // This is not queried but include because inline value of an included node
KeyPop(5)
HashChild: SomeHash at 2 (second nibble of b"r")
KeyPop(4)
HashChild: SomeOtherHash at 1 (b"alfa" branch)
```

### Multiple proof sequence:

C is a single proof instruction sequence (anything but `Version` or `EndProof`).
Default version 0 is first line.
Undefined default version is second line.

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
