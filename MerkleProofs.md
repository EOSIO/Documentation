# Proofs and Merkles

This document describes how merkle trees are used to enable proofs of user actions to light weight clients. Lightweight clients are the foundation of inter-blockchain communication where one blockchain acts as a light-client to another blockchain.

Every user action can be viewed as someone writing a cheque which gets processed by the bank (blockchain) and if accepted logged. Inter Blockchain communication is about generating proofs that a particular check was accepted and the order of its acceptance relative to other cheques written by the same sender or receiver.

Chain A can receive messages from Chain B by following Chain B’s block headers and processing action proofs. Each action proof has one or more sequence numbers which Chain A uses to make sure there are no gaps in processing.

Where as other chains generate a merkle tree over state (account balances), EOS.IO generates a merkle tree over sequenced actions. A light client can derive the balance of a single account by processing all sequenced actions for that account without having to process the actions for other accounts.

In many cases it is not necessary to know the balance of an account, instead all the light client needs to do is verify payment. This can be verified with the proof of a single action.

Blockchains are built on top of deterministic state machines; therefore, inter blockchain communication is most effective when guarantees of completeness and order can be provided. The EOS.IO protocol creates a TCP like communication channel between chains. This means that missing or out of order proofs can be detected.

While you can generates proofs to guarantee no actions are skipped, it is not possible to prove that you have all actions up to the present moment. To prove completeness to the present moment one must generate a transaction, have it included, then generate a proof that the most recent transaction was confirmed with the proper sequence number.

## Merkle Trees in EOS.IO

### Canonical Trees and Proofs

All of the merkle calculations in EOS.IO share a few conventions that disambiguate entries in a proof without requiring additional space.  Given two leaves with respective digests of `Da` and `Db`.  Their shared parent node in a basic merkle tree would be the digest of the concatenation of `Da` and `Db` (`Da|Db`).  However, unless the chosen digest calculation is commutative, `Da|Db` is not the same as `Db|Da`.  This means that a proof, in the form of a “path” from a leaf to the root of a Merkle tree (sometimes referred to as a “log proof”) must indicate whether the entries in the “path” represent left or right concatenation.

To avoid this extra “path” data in the proof, EOS.IO applies a transform to the leaf digests before concatenating them for the new digest.  This transform, indicates whether the digest is on the left or right side of the concatenation.  The proof “path” then includes pre-transformed digests making the correct concatenation operation easy to infer.

More precisely, EOS.IO enforces that the first bit of the leaf digest is `0` for “left” digests and `1` for “right” digests.  When reading a proof path entry (`Dp`) and applying it to a given digest (`Dn`) if the first bit of `Dp` is `0` then the resulting digest is `hash(Dp|Dn)` otherwise it is `hash(Dn|Dp)`

###### pseudocode for generating a hash from a left and right leaf
```
C_HASH(left,right)
    LET canonical_left := CLEAR_FIRST_BIT(left),
    LET canonical_right := SET_FIRST_BIT(right),
    LET data := CONCATENATE(canonical_left,canonical_right),
    HASH_FN(data);
 ```


In addition, should construction of a merkle tree require hashing a node with an unknown value, for example in a balanced tree of 3 nodes, the resulting digest is the concatenation of the digest with itself.
```
Root:         Dabcc
              /   \
            Dab   Dcc  <- Dc concatenates with itself
            / \   /
Leaves:    Da Db Dc
```

### The Block Merkle

For any Block N, its header includes the root of a balanced merkle tree where the leaves are the block ids for all preceding blocks 0..N-1: the Block Root.  This serves as a commitment to the contents of the blockchain, from genesis on, by each new block.

Given any block id for a block in the blockchain, and the headers in a trusted irreversible block,   it is possible to prove that the block is included in the blockchain.  This proof takes ceil(log2(N)) digests for its path, where N is the number of blocks in the chain.  Given a digest method of SHA256, this means a proof for the existence of any block in a chain which contains 100 million blocks contains 864 bytes of data.  Similarly, if in a chain which contains 100 billion blocks (100,000% increase), the proof is 1,184 bytes ( ~37% increase).  It is therefore easy to see that proof remains succinct even when applied to extremely large and long-lived blockchain.

### The Action Merkle

For any Block N, the headers include the root of a nearly-balanced merkle tree where the leaves are commitments to data generated while processing actions: the Action Root.  This can be used to prove existence and, ordering of actions.  Additionally, it can be used to prove completeness of transcripts of actions that affect a given scope.

The data committed to for each action, or Action Commitment, is:

 * **receiver:** the account name whose contract received the given action
 * **scope:** the scope where this action is defined
 * **name:** The name of the action (eg: ‘transfer’)
 * **data:** the binary encoded data payload delivered to the contract
 * **data_access:** an array of sequence numbers for scopes accessed during the processing of this action
   * This can be used to prove complete transcripts by providing actions and proofs for the sequence numbers between two given actions
   * Sequence numbers always increment on write operations and never on reads.  There will be exactly one action with a write for a given (sequence, scope, receiver)
 * **region_id:** the region this action was processed in
 * **cycle_index:** the cycle this action was processed in
   * This can be used to prove ordering constraints
   * Shards inside a cycle can execute in parallel; there is no deterministic ordering between actions in the same cycle but different shards.

###### Example Action Commitment
```
   receiver:inite
      scope:eosio
       name:transfer
       data:{
               "from":"inite",
               "to":"inita",
               "amount":10000,
               "memo":"memo"
            }
data_access:[
               {"type":"write","scope":"inite","sequence":1},
               {"type":"write","scope":"inita","sequence":0}
            ]
     region:0
      cycle:0
```

For each shard (a unit of parallelizable execution in a cycle) a balanced merkle tree is constructed of these action commitments to generate a temporary shard merkle root.  This is done for speed of parallel computation.  The block header contains the root of a balanced merkle tree whose leaves are the roots of these individual shard merkle trees.  This means that the resulting merkle tree represented by the Action Root is not guaranteed to be  a perfectly balanced merkle tree, however, it should be very close in most cases.

Given an action, somewhere in the blockchain, it is possible to succinctly prove the application of that action by first proving that it was committed to by a block’s Action Root, and then that the given block was committed to by a trusted irreversible block header’s Block Root.  These two proofs are referred to as the “Action Path” and the “Block Path” respectively.

The Action Path consists of all the data (sibling digests from the merkle tree) necessary to recalculate the root of a merkle tree with respect to a single Action Commitment.  That merkle tree’s root should exactly match the Action Root committed to by the header of the block that applied the action.

The Block Path consists of all the data necessary to recalculate the root of a merkle tree with respect to a single Block ID.  That merkle tree’s root should exactly match the Block Root committed to by the header of a trusted irreversible block.

For example,

Given the action above, applied in a block with the ID:
```
00000033e4f902358b1d43918d197652fc01baaf0e6ca8ee38cda2e8780b7dbd
```

Whose Action Root was:
```
067ed979eeff38064f10bd77dcd3630ce104ae2685b5d6c2500552b0172e4c57
```

And trusted irreversible block whose Block Root is:
```
1ae092ea41e70982b56b9673f990346876ba535595df16dc1094740c144aaae6
```

That proof data looks like this:
```
Action Path:
   9b75773cc9e2d8d186c408c3f5cdc55cde5b23addb6f25aa924bd3279bd0ca91
   9cb66ff0d78991662c4784afac391a0e5f7104164daed36ba1fd4c6cc4907067
   C65abe772ecb192105cd627bcb170eff8ec8f707a5d68d5c8060fee0236b4167
   4cf39dfbe8a0fdc2540c8f0b68037484051a20d6ca00b891bbfc66d826503677
   A8773db30f3733063352d1abec5fea265115bdf646a69d21545746a4f7cff3f3

Block Path:
   8000003418a608c3704f80cdfed2793a055cc13993eea67ccdd018d9df027101
   2351ee036c6a4052e18c9ded8d35c1b1dfb330531e163039847797d38e9babd2
   De1996e5c23c73fa212abaaf423b9038e289fc7a19be5a7f395c7727a5266168
   Ccd7c6e10b398fd27cfc4d29f339806593d614c736ae1af8a47908e44d2fe924
   1e95b55e98d329bda8f2c79d1f969650716207a03f13f540931ab5a6a80f4d0b
   369d86219b35e493a4fda3649c9f3a546809add40462c4b5ddca185f75cfe05c
   9067c63043027831fbc7ba046d5bd7d9a0c23e1baa6fe0ac16a911ff7c3acd74
```

To verify this proof, first serialize the claimed Action Commitment to EOS.IO’s standard binary format.  That results in the following binary representation (in hex):

```
000000000095dd740000000000ea3055000000572d3ccdcd1d000000000095dd74000000000093dd741027000000000000046d656d6f0000000000000000020100000000000000000000000095dd7401000000000000000100000000000000000000000093dd740000000000000000
```

EOS.IO currently uses SHA256 for its digest method(`HASH_FN`), so the leaf digest is the SHA256 of the above binary representation:

```
1b75773cc9e2d8d186c408c3f5cdc55cde5b23addb6f25aa924bd3279bd0ca91
```

Using that leaf digest, apply the Block Path, referring to the section on canonical paths and signatures to determine if an entry in the path requires left or right concatenation.  After applying the complete path, the resulting root digest should be exactly equal to the Action Root in the header of the block that applied the action:

<pre><code>
C_HASH(
   <span color="#FFCC00">1b75773cc9e2d8d186c408c3f5cdc55cde5b23addb6f25aa924bd3279bd0ca91</span>,
   9b75773cc9e2d8d186c408c3f5cdc55cde5b23addb6f25aa924bd3279bd0ca91)
=&gt 1cb66ff0d78991662c4784afac391a0e5f7104164daed36ba1fd4c6cc4907067

C_HASH(
   1cb66ff0d78991662c4784afac391a0e5f7104164daed36ba1fd4c6cc4907067,
   9cb66ff0d78991662c4784afac391a0e5f7104164daed36ba1fd4c6cc4907067)
=&gt 465abe772ecb192105cd627bcb170eff8ec8f707a5d68d5c8060fee0236b4167

C_HASH(
   465abe772ecb192105cd627bcb170eff8ec8f707a5d68d5c8060fee0236b4167,
   C65abe772ecb192105cd627bcb170eff8ec8f707a5d68d5c8060fee0236b4167)
=&gt dd205c37e3de758120f50060b052736ba903382904a445fae3a371cfb29820ab

C_HASH(
   4cf39dfbe8a0fdc2540c8f0b68037484051a20d6ca00b891bbfc66d826503677,
   dd205c37e3de758120f50060b052736ba903382904a445fae3a371cfb29820ab)
=&gt ecce9664089221e351aaedafd967a9e19e25ad03d82949b3953de53f8b3b1996

C_HASH(
   ecce9664089221e351aaedafd967a9e19e25ad03d82949b3953de53f8b3b1996,
   a8773db30f3733063352d1abec5fea265115bdf646a69d21545746a4f7cff3f3)
=&gt <span color="#00EF00">067ed979eeff38064f10bd77dcd3630ce104ae2685b5d6c2500552b0172e4c57</span>
</code></pre>

Once we have verified that the block has a commitment retiring the action as described, we must also verify that the retiring block is included in the blockchain using the Block Path and the retiring block’s id.  After applying the complete path, the resulting root digest should be exactly equal to the Block Root in the header of the trusted irreversible block:

<pre><code>
C_HASH(
   <span color="#FFCC00">00000033e4f902358b1d43918d197652fc01baaf0e6ca8ee38cda2e8780b7dbd<span>,
   8000003418a608c3704f80cdfed2793a055cc13993eea67ccdd018d9df027101)
=> da16dd856cbb888545c0ed248f2acbbed57037cf407e258849d4600c8f074739

C_HASH(
   2351ee036c6a4052e18c9ded8d35c1b1dfb330531e163039847797d38e9babd2,
   da16dd856cbb888545c0ed248f2acbbed57037cf407e258849d4600c8f074739)
=> da1a4326c7d83cc86f1154e76000ae94a7579833e75d9378972c6f034e044c11

C_HASH(
   da1a4326c7d83cc86f1154e76000ae94a7579833e75d9378972c6f034e044c11,
   de1996e5c23c73fa212abaaf423b9038e289fc7a19be5a7f395c7727a5266168)
=> d2d0abf4e6d8b771f8de27413ce7c9ca9e443c89fd10c4d42cb37bcc47cb3598

C_HASH(
   d2d0abf4e6d8b771f8de27413ce7c9ca9e443c89fd10c4d42cb37bcc47cb3598,
   ccd7c6e10b398fd27cfc4d29f339806593d614c736ae1af8a47908e44d2fe924)
=> c36045ffee9c98eb419077aa356d65fb6a928914c40ec88aadcf634781e56c23

C_HASH(
   1e95b55e98d329bda8f2c79d1f969650716207a03f13f540931ab5a6a80f4d0b,
   c36045ffee9c98eb419077aa356d65fb6a928914c40ec88aadcf634781e56c23)
=> 4432bce7431f676ed704f2d3367cb33407898b70f51fea2dfa014b97ac37a97b

C_HASH(
   369d86219b35e493a4fda3649c9f3a546809add40462c4b5ddca185f75cfe05c,
   4432bce7431f676ed704f2d3367cb33407898b70f51fea2dfa014b97ac37a97b)
=> 94f5ba720b886515ada9c0f73ab393dffc43beaeac7be6f7f86bce808834dfde

C_HASH(
   94f5ba720b886515ada9c0f73ab393dffc43beaeac7be6f7f86bce808834dfde,
   9067c63043027831fbc7ba046d5bd7d9a0c23e1baa6fe0ac16a911ff7c3acd74)
=> <span color="#00EF00">1ae092ea41e70982b56b9673f990346876ba535595df16dc1094740c144aaae6</span>
</code></pre>

