---
title: 'Coconsensus'
disqus: https://hackmd.io/6sbnr54gThi2Na5KjiSCxg
---

Mosaic Implementation Discussion: Coconsensus
===

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 08/01/2020    | Coconsensus contract |


Editor: Benjamin Bollen

## Overview

Coconsensus mirrors the consensus contract on the auxiliary chain;
however, notable differences make it worthwhile to describe it in its own right.

#### Blockchains
In contrast to `Consensus` (which deals with consensus on metablocks), `Coconsenus` focuses on the finalisation and commitment of checkpoints. We can therefore introduce a mapping

```js
mapping (metachainId => mapping(blocknumber => Block)) blockchains;
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
can store a new `block` in `blockchains` if the number strictly increases and the `CheckpointCommitStatus` is `Undefined`; status of the block is set to `Finalised` and the current dynasty of `Self` is stored along with it.

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

Coconsensus must call on `Protocore:openMetablock(uint256 metablockHeight)`, as Protocore must know the current metablock height to 

Coconsensus must iterate over the array of updated validators and call on `Coreputation:upsert(updatedReputation)` and call `Protocore:upsert(updatedReputation)` to "join" or "logout" at the new metablock height. 

#### storing domainseparators
Coconsensus must be able to hash Kernel and VoteMessage with the domain separator of the `Core` validators' contract on origin. We can provide the domain seperators for Self (and other protocores) metachainId through `CoconsensusGenesis` and have
```js
mapping (metachainId => b32) domainSeparators;
```
OriginProtocore VoteMessages should be hashed with the same domain separator as Self, because it concerns the observation by the Core validators of Self. Therefore we want to enforce slashing conditions with a domain separator for this same group of validators.

## User stories for Gen1


---
## Meeting notes
### Meeting 1
date/time: 9/1/2020
attendees:


Deepesh: Do we need to track all the finalized checkpoints ? no, we can agressively prune in later generations



Question Ben fri 10/1/2020:
`Coreputation:upsertValidator` may just not return an enum; because perhaps we should always just join/logout in protocore to identically mirror the validatorSet of `Core` in `Protocore`; if a validator is slashed, then that doesnt matter because the votes dont count. (This is different from the discussion earlier on Fri 10/1/2020)