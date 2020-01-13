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

![Contract architecture](https://i.imgur.com/pxra0Gn.jpg)

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

```js
<H(T(s)), s, t, h_s, h_t>
```

which is hashed to produce a `VoteMessageHash`; and a vote message where `h_t - h_s = epochLength`, can be proposed as a `Metablock` in `Core`.

Define now a `Link` as the object

```js
struct Link {
    bytes32 parentVoteMessageHash;
    uint256 openMetablockHeight;
    bytes32 targetBlockHash;
    uint256 targetBlockHeight;
    uint256 targetDynasty;
    bytes32 sourceTransitionHash;
    uint256 forwardVoteCount;
    uint256 rearVoteCount;
    CheckpointFinalizationStatus targetFinalization;
}
```

```js
enum CheckpointFinalizationStatus {
    Undefined,
    Registered,
    Justified,
    Finalised,
    Committed
}
```

## User Stories

- As a co consensus contract I should be able to call setup function.
- As a user I should be able to propose a valid link.
- As an active validator I should be able to vote for the registered link.

## Proposed interfaces

`Protocore` knows about `Link`s and `Validator`s who can vote on these links.
Links become supermajority links under certain conditions specified later.

```js

contract Protocore
    is MasterCopyUpgradable, GenesisProtocore, MosaicVersion, ValidatorSet {

    /**
     * @notice setup() function initializes the genesis link from values
     *         defined in GenesisProtocore.
     *         `targetFinalization` of the genesis link will be set to finalised.
     *         `targetDynasty` of the genesis link will be set to 0.
     *
     * @dev The following fields of the gensis link will be initialized with
     *          null (dummy) values:
     *              - `parentVoteMessageHash` will be set to 0
     *              - `sourceTransitionHash` will be set to 0
     *              - `forwardVoteCount` will be set to 0
     *              - `rearVoteCount` will be set to 0
     *
     * \pre The function can be called only by co consensuse contract.
     * \pre The function can be called only once.
     * \pre `targetBlockHash` value read from GenesisProtocore should not be 0.
     */
    function setup()
        external
    {
        // ...
    }

    /**
     * @notice proposeLink() function proposes a valid link to be voted later by
     *          active validators.
     *
     * \pre `parentVoteMessageHash` is not 0.
     * \pre `parentVoteMessageHash` refers to an already proposed link which
     *      `targetFinalizationStatus` is at least justified.
     * \pre `targetBlockHash` is not 0
     * \pre `targetBlockHeight` is a multiple of the epoch length.
     * \pre `targetBlockHeight` is bigger than a targetBlockHeight pointed
     *      by `_parentVoteMessageHash` link (source block height of the newly
     *      proposed link)
     * \pre If the proposed link length in blocks is equal to one epoch length
     *      (finalization link) current block number is strictly less than
     *      target block height + epoch length (inclusion principle).
     * \pre voteMessageHash calculated from the input parameters does not exist.
     */
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
        // ...
    }

    /**
     * @notice registerVote() function registers a vote for a link specified
     *         by vote message hash.
     *
     * \pre `_voteMessageHash` is not 0.
     * \pre The metablock height of the link pointed by `_voteMessageHash` is
     *      equal to the open metablock height.
     * \pre Only an active (staked and non-slashed) validator can call.
     * \pre If a link (specified by `_voteMessageHash`) length is equal to one
     *      epoch length (finalization link) the inclusion principle should
     *      be satisfied:
     *          current block number is strictly less than
     *          target block height + epoch length
     *
     * \post If the validator is in forward validators' set increments
     *       forwardVoteCount.
     * \post If the validator is in rear validators' set increments
     *       rearVoteCount.
     * \post If quorum reached for both forward and rear validator set
     *       then targetFinalization is set to justified.
     *       if the link length is equal to the epoch length (finalization link)
     *       then marks targetFinalization of the parent parent as finalised.
     * \post Calls co consensus once checkpoint is finalised.
     */
    function registerVote(
        bytes32 _voteMessageHash,
        bytes32 _r,
        bytes32 _s,
        uint8 _v
    )
        external
    {
        // ...
    }

    /**
     * @notice commitMetablock() function marks the specified metablock
     *         as committed.
     *
     * \post Increments open metablock height.
     */
    function commitMetablock(
        bytes32 _voteMessageHash
    )
        external
    {
        // ...
    }
}

```

## ValidatorSet

---

## Meeting 2

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
