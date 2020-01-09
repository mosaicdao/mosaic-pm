---
title: 'Protocore'
disqus: https://hackmd.io/hNL_Co0lQ4Wbn_rpQeCVdQ
---

# Mosaic Implementation Discussion: Protocore

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 6/12/2019    | Protocore contract |

Meeting date/time: Friday 6 december 2019

Editor: Benjamin Bollen

## Contract architecture
![](https://i.imgur.com/pxra0Gn.jpg)

## Protocore contract flow

tracking an initial stub here:
https://github.com/benjaminbollen/mosaic-1/blob/protocore/contracts/protocorev2/ProtoCorev2.sol


## Dual description between checkpoints and links

The objective of `Protocore` is to finalise a unique history of proposed blocks.

First we coarse-grain the history into a history of checkpoint blocks,
where for the consensus we only allow votes on blocks at
a block number which is an integer multiple of a `constant epochLength`.

However, if we implement a consensus logic with blocks or checkpoint blocks as fundamental objects, then the code will become complex.
We can delegate responsibility of ancestry checks to the layer-1 code to verify an unbroken chain of headers.

We can further construct a dual picture of `Link`s.

We do this in two steps. We define a `VoteMessage` as the known object
```
<H(T(s)), s, t, h_s, h_t>
```
which is hashed to produce a `VoteMessageHash`; and a vote message where `h_t - h_s = epochLength`, can be proposed as a `Metablock` in `Core`.

Define now a `Link` as the object
```
struct Link {
    bytes32 parentVoteMessageHash;
    bytes32 targetBlockHash;
    uint256 targetBlockHeight;
    bytes32 sourceTransitionHash;
    uint256 forwardVoteCount;
    uint256 rearVoteCount;
    CheckpointFinalisationStatus targetFinalisation;
}
```

```js
enum CheckpointFinalisationStatus
    Undefined,
    Registered,
    Justified,
    Finalised
```

Protocore calls on coconsensus when a checkpoint is finalised.

## Preconditions on proposing Link

- `parentVoteMessage` must refer to a link which `targetFinalisationStatus` is at least justified
- calculate VoteMessageHash, shouldnt already exist
- 

## Finalisation status of target checkpoints

## Proposed function signatures and summary

`Protocore` knows about `Link`s and `Validator`s who can vote on these links. Links become supermajority links under certain conditions specified later.


```js
contract Protocore is MasterCopyUpgradable, GenesisProtocore, MosaicVersion, ValidatorSet {


    // TODO: This can be setup() instead of registerGenesisLink
    function registerGenesisLink()
        external
    {

    }

    function proposeLink(
        bytes32 _parentVoteMessageHash,
        bytes32 _targetBlockHash,
        uint256 _targetBlockHeight,
        bytes32 _sourceKernelHash,
        bytes32 _sourceOriginObservation,
        uint256 _sourceDynasty,
        uint256 _sourceAccumulatedGas,
        bytes32 _sourceCommitteeLock
    )
        external
    {

    }

    function registerVote(
        bytes32 _voteMessageHash,
        bytes32 _r,
        bytes32 _s,
        uint8 _v
    )
        external
    {
}

```

## ValidatorSet

---
## meeting 2
attendees: A G J D S P B
date 9/1/2020

Pro: base protocore for ProtocoreSelf and OriginObserver

Abhay: 
- Does ProtoCore has state machine logic like Core state machine(Core status) on origin chain?

Sarvesh:- genesis link: 
    1. There will be setup function which will initialize geneisis link. Some of the fields will be dummy values like (forward count and rear count will be zero.)
    
Task: Specification for defining metachain id, we need to define it for origin chain too.
    
Ben: Should the protocore (self) and origin protocore (origin observer) should inherit the validator set? or it can just reference to the coreputation contract to get the core validators ?

Deepesh: Will the protocores for other auxiliary chain will have the same validators sets that are for self protocore ? 