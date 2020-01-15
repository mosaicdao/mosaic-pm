---
title: 'Coconsensus'
disqus: https://hackmd.io/6sbnr54gThi2Na5KjiSCxg
---

Mosaic Implementation Discussion: Coconsensus
===

| version | Last updated | Component            |
| ------- | ------------ | -------------------- |
| 0.14    | 14/01/2020   | Coconsensus contract |


Editor: Benjamin Bollen

## Overview

Coconsensus mirrors the consensus contract on the auxiliary chain;
however, notable differences make it worthwhile to describe it in its own right.

#### Blockchains
In contrast to `Consensus` (which deals with consensus on metablocks), `Coconsenus` focuses on the finalisation and commitment of checkpoints. We can therefore introduce a mapping

```js
mapping (metachainId => mapping(blocknumber => Block)) blockchains;
mapping (metachainId => uint256) blocktips;
```

where a `struct Block` tracks the `CheckpointCommitStatus` and the `dynasty` number of the local metachain at which the status update was included in the chain.

```js
struct Block {
    bytes32 blockhash;
    CheckpointCommitStatus commitStatus;
    uint256 statusDynasty;
}
```

```js
enum CheckpointCommitStatus {
    Undefined,
    Finalised,
    Committed
}
```

Note that the block number is not storing all block numbers, not even all checkpoints; only those that are reported through `coconsensus:finaliseCheckpoint`

#### Protocores
Using the `metachainId` of the local metachain ("self") and the origin chain (Ethereum, "origin"), coconsenus is minimally connected two special `Protocore` contracts for these metachainIds (`SelfProtocore` and `OriginProtocore`).

```js
mapping (metachainId => Protocore) protocores;
```

For all metachainIds tracked (with the exception of `SelfProtocore`) an associated anchor is connected.

```js
mapping (metachainId => Anchor) anchors;
```

In Gen1 there is only one anchor for the origin chain `OriginAnchor`, stored under the "metachainId" of the Origin chain.

#### Finalize checkpoint

```js
function finalizeCheckpoint(b32 metachainId, uint256 number, b32 blockhash)
    onlyRunningProtocore(metachainId)
```
can store a new `block` in `blockchains` if the number strictly increases and the `CheckpointCommitStatus` must be `Undefined`; status of the block is set to `Finalised` and the current dynasty of `Self` is stored along with it.

#### Commit checkpoint

```js
function commitCheckpoint(
    b32 metachainId,
    uint256 kernelHeight,
    address[] updatedValidators,
    uint256[] updatedReputation,
    uint256 gasTarget,
    b32 transitionHash,
    b32 source,
    b32 target,
    uint256 sourceBlockNumber, 
    uint256 targetBlockNumber
)
```

can be called by anyone. Following requirements must pass.

1. The `sourceBlockNumber` and `source` must be registered in `blockchains` as `Finalised`.

2. The parameters that make up the vote message 
```
VoteMessage{transitionHash; source; target; sourceBlockNumber; targetBlockNumber}
```
are for the parent metablock, so at `kernelHeight - 1`, being committed on aux (already committed on origin).

Together they must be hashed to form `b32 parent` for the new Kernel opening (must be hashed following EIP712 with Core contract's address).

From the parameters for the Kernel
```
Kernel{kernelHeight; parent; updatedValidators; updatedReputation; gasTarget}
```
the `kernelHash` can be calculated. In the `ConsensusCogateway` the pair `(kernelHeight, kernelHash)` must be stored; so Coconsensus can query the `kernelHash` for this `kernelHeight` and match with the calculated hash.

Following these checks coconsensus should mark the status of the checkpoint as `Committed`.


Coconsensus must iterate over the array of updated validators and call on `Coreputation:upsert(updatedReputation)` and call `Protocore:upsert(updatedReputation)` to "join" or "logout" at the new metablock height.
**Coconsensus must mirror this list of updated validators as received, so that core and protocore contain the same validator sets. (correction: coreputation doesnt have to return an action-enum)**

Once all validators have been updated in Coreputation and protocore, the new validator set can be effectuated by calling `Protocore:openMetablock(uint256 metablockHeight, b32 kernelHash)`, as Protocore must know the current metablock height for knowing the current validator set.

#### storing domainseparators
Coconsensus must be able to hash Kernel and VoteMessage with the domain separator of the `Core` validators' contract on origin. We can provide the domain seperators for Self (and other protocores) metachainId through `CoconsensusGenesis` and have
```js
mapping (metachainId => b32) domainSeparators;
```
OriginProtocore VoteMessages should be hashed with the same domain separator as Self, because it concerns the observation by the Core validators of Self. Therefore we want to enforce slashing conditions with a domain separator for this same group of validators.

#### observe blocks and anchor state roots

Finalised checkpoints of SelfProtocore do not need to be anchored in an Observer contract. (There is no point to send messages from Self to Self, or to observe oneself.)

Finalised checkpoints of OriginProtocore are stored with the current dynasty of Self. The associated state roots of these checkpoints must be anchored into the `OriginObserver is Anchor` for messages to be received from Origin.

The function `coconsensus:observeBlock(metachainId, rlpBlockHeader)` must decode the RLP encoded bytes to extract the block number, and state root.

It must hash the RLP encoded block header bytes and match it against the block hash stored in `blockchains` under `number`. This block must stored as `Finalised`

It must check that the current dynasty of Self is greater than the dynasty stored with the finalised observation checkpoint from OriginProtocore.

If so, then we can call on `OriginObserver:anchorStateRoot` to store the state root in the contract. (Note that it will revert if the state root was already anchored.)

## User stories for Gen1

(bare bones)

- as a user, I want to call `setup()` which goes over internal functions `setupCoreputation()`, `setupAnchors()`, `setupProtocores()` to setup all contracts from their GenesisContracts.

- as a Protocore, I want to call `finaliseCheckpoint` when I have finalised a checkpoint.

- as a user I want to call `commitMetablock` when a new metablock in the consensusCogateway has been confirmed

- as a user I want to call `anchorStateRoot(metachaindId, number)` when SelfProtocore has increased the dynasty of Self, such that OriginProcotore finalisations at lower dynasty can be anchored into OriginAnchor.


---
## Meeting notes
### Meeting 1
date/time: 9/1/2020
attendees:


Deepesh: Do we need to track all the finalized checkpoints ? no, we can agressively prune in later generations



Question Ben fri 10/1/2020:
`Coreputation:upsertValidator` may just not return an enum; because perhaps we should always just join/logout in protocore to identically mirror the validatorSet of `Core` in `Protocore`; if a validator is slashed, then that doesnt matter because the votes dont count. (This is different from the discussion earlier on Fri 10/1/2020)

## meeting 3
date: 13/1/2020

note taker: deepesh

- Ben runs over user stories
- Terms that we use:
    - block number -> related to blockchain blocks
    - dynasty -> this is the dynasty related to finalization of blocks (Casper FFG)
    - height -> This is height of metablock.

- Deepesh: Why do we need mapping to store domainseparators ?
    - the votes on origin protocore and self protocore will sign with same domain seperators
    - Currently we will have just one value in the mapping.
    - Later we may have other protocores (different metachain id)
- The metachainid will be stored in the mapping in the genesis block.
- Deepesh: Metachain id for origin protocore ?
    - in hashmetachain, use the anchor address as 0x00 to get the metachain id for origin protocore.
    - 