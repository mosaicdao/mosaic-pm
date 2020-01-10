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
mapping (metachainId => Protocore) protocores
```



- update validators

---
## Meeting notes
### Meeting 1
date/time:
attendees:


Deepesh: Do we need to track all the finalized checkpoints ?

