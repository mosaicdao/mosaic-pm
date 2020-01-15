---
title: 'Protocore'
disqus: https://hackmd.io/hNL_Co0lQ4Wbn_rpQeCVdQ
---

# Mosaic Implementation Discussion: Protocore

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 14/01/2020   | Protocore contract |

Meeting date/time: Friday 6 december 2019

Editor: Benjamin Bollen, Paruyr Gevorgyan

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
    bytes32 targetBlockHash;
    uint256 targetBlockNumber;
    bytes32 sourceTransitionHash;
    uint256 proposedMetablockHeight;
    uint256 forwardVoteCount;
    uint256 forwardVoteCountNextHeight;
    uint256 forwardVoteCountPreviousHeight;
    CheckpointFinalisationStatus targetFinalisation;
}
```

```js
enum CheckpointFinalisationStatus {
    Undefined,
    Registered,
    Justified,
    Finalised
}
```

## User Stories

- As a coconsensus contract I should be able to call `setup` function.
- As a user I should be able to propose a valid link.
- As an active validator I should be able to vote for the registered link.
- As a coconsensus contract I should be able to open a new metablock.

## Origin Protocore

`OriginProtocore` Casper FFG game differs from `SelfProtocore` one with absence
of the **inclusion principle**. There is no kernel and transition object for origin observations, so the VoteMessage should be simplified.

Metablock height of the `OriginProtocore` is the metablock height of the `SelfProtocore`. This determines the active validators that can justify and finalise the links (and associated checkpoints).

OriginProtocore reads validator set from SelfProtocore.

## Proposed interfaces

`Protocore` knows about `Link`s and `Validator`s who can vote on these links.
Links become supermajority links under certain conditions specified later.

```js
contract Protocore is MosaicVersion, ValidatorSet
{
    mapping(bytes32 /* vote message hash */ => Link) public links;

    /**
     * @notice openMetablock() function marks the specified metablock
     *         as opened.
     *
     * \pre `_metablockHeight` is plus one of the current metablock height of
     *      the protocore.
     * \pre `_kernelHash` is not 0.
     *
     * \post Increments open metablock height.
     * \post Updates stored kernel hash.
     */
    function openMetablock(
        uint256 _metablockHeight,
        bytes32 _kernelHash
    )
        external
    {
        // ...
    }

    /**
     * @notice _proposeLink() function proposes a valid link to be voted later
     *          by active validators.
     *
     * \pre `parentVoteMessageHash` is not 0.
     * \pre `parentVoteMessageHash` refers to an already proposed link which
     *      `targetFinalisation` is at least justified.
     * \pre `targetBlockHash` is not 0
     * \pre `targetBlockNumber` is a multiple of the epoch length.
     * \pre `targetBlockNumber` is bigger than a targetBlockNumber pointed
     *      by `_parentVoteMessageHash` link.
     * \pre A vote message hash (calculated with input params) does not exist.
     *
     * \post The link is saved in `links` mapping with currently
     *       open metablock height as `proposedMetablockHeight`.
     * \post `targetFinalisation` is set to 'Undefined'.
     * \post forwardVoteCount -s set to 0.
     */
    function _proposeLink(
        bytes32 _parentVoteMessageHash,
        bytes32 _sourceTransitionHash,
        bytes32 _targetBlockHash,
        uint256 _targetBlockNumber
    )
        internal
    {
        // ...
    }

    /**
     * @notice registerVote() function registers a vote for a link specified
     *         by vote message hash.
     *
     * \pre `_voteMessageHash` is not 0.
     * \pre A link mapping to the `_voteMessageHash` exists.
     * \pre The metablock height of the link pointed by `_voteMessageHash` is
     *      equal to the open metablock height or open metablock height minus 1.
     * \pre Only an active validator can call.
     *
     * \post If quorum reached then for the link pointed by `_voteMessageHash`
     *       targetFinalisation is set to justified. If the link length is
     *       equal to the epoch length (finalisation link) then marks
     *       targetFinalisation of the link pointed by `parentVoteMessageHash`
     *       as finalised.
     * \post Calls coconsensus if the source checkpoint of the link is finalised.
     */
    function _registerVote(
        bytes32 _voteMessageHash,
        bytes32 _r,
        bytes32 _s,
        uint8 _v
    )
        internal
    {
        // ...
    }

    /**
     * @notice Takes vote message parameters and returns the typed vote
     *         message hash.
     */
    function hashVoteMessage(
        bytes32 _sourceTransitionHash,
        bytes32 _sourceBlockHash,
        bytes32 _targetBlockHash,
        uint256 _sourceBlockNumber,
        uint256 _targetBlockNumber
    )
        private
        view
        returns (bytes32 voteMessageHash_)
    {
        // ...
    }
}

contract OriginProtocore is MasterCopyUpgradable, GenesisOriginProtocore, Protocore
{
    /**
     * @notice setup() function initializes the genesis link from values
     *         defined in GenesisOriginProtocore.
     *
     * @dev The following fields of the genesis link will be initialized with
     *      constant placeholder values:
     *          - `parentVoteMessageHash` will be set to GEN_PARENT_VOTE_MESSAGE_HASH
     *          - `targetBlockHash` from GEN_TARGET_BLOCK_HASH
     *          - `targetBlockNumber` from GEN_TARGET_BLOCK_NUMBER
     *      `targetFinalisation` of the genesis link will be set to finalised.
     *      `proposedMetablockHeight` will be set to 0.
     *      `forwardVoteCount` -s will be set to 0.
     *
     * \pre The function can be called only by coconsensus contract.
     * \pre The function can be called only once.
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
     * @dev Satisfies \pre and \post conditions of Protocore::_proposeLink().
     *
     */
    function proposeLink(
        bytes32 _parentVoteMessageHash,
        bytes32 _targetBlockHash,
        uint256 _targetBlockNumber
    )
        external
    {
        // ...

        _proposeLink(
            _parentVoteMessageHash,
            bytes32(0),
            _targetBlockHash,
            _targetBlockNumber
        );
    }

    /**
     * @notice registerVote() function registers a vote for a link specified
     *         by vote message hash.
     *
     * @dev Satisfies \pre and \post conditions of Protocore::_registerVote().
     */
    function registerVote(
        bytes32 _voteMessageHash,
        bytes32 _r,
        bytes32 _s,
        uint8 _v
    )
        external
    {
        _registerVote(
            _voteMessageHash,
            _r,
            _s,
            _v
        );
    }
}

contract SelfProtocore is MasterCopyUpgradable, GenesisSelfProtocore, Protocore
{
    /**
     * @notice setup() function initializes the genesis link from values
     *         defined in GenesisSelfProtocore.
     *
     * @dev The following fields of the genesis link will be initialized with
     *      constant placeholder values:
     *          - `sourceTransitionHash` will be set to GEN_SOURCE_TRANSITION_HASH
     *          - `parentVoteMessageHash` will be set to GEN_PARENT_VOTE_MESSAGE_HASH
     *          - `targetBlockHash` from GEN_TARGET_BLOCK_HASH
     *          - `targetBlockNumber` from GEN_TARGET_BLOCK_NUMBER
     *      `targetFinalisation` of the genesis link will be set to finalised.
     *      `proposedMetablockHeight` will be set to 0.
     *      `forwardVoteCount` -s will be set to 0.
     *
     * \pre The function can be called only by coconsensus contract.
     * \pre The function can be called only once.
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
     * @dev Satisfies \pre and \post conditions of Protocore::_proposeLink().
     *
     * \pre If the proposed link length in blocks is equal to one epoch length
     *      (finalisation link) current block number is strictly less than
     *      target block number + epoch length (inclusion principle).
     * \pre `_sourceKernelHash` should match with the open kernel hash.
     */
    function proposeLink(
        bytes32 _parentVoteMessageHash,
        bytes32 _targetBlockHash,
        uint256 _targetBlockNumber,
        bytes32 _sourceOriginObservation,
        bytes32 _sourceKernelHash,
        uint256 _sourceDynasty,
        uint256 _sourceAccumulatedGas,
        bytes32 _sourceCommitteeLock
    )
        external
    {
        // ...

        // checks inclusion principle

        // calculates source transition hash

        _proposeLink(
            _parentVoteMessageHash,
            sourceTransitionHash,
            _targetBlockHash,
            _targetBlockNumber
        );
    }

    /**
     * @notice registerVote() function registers a vote for a link specified
     *         by vote message hash.
     *
     * @dev Satisfies \pre and \post conditions of Protocore::_registerVote().
     *
     * \pre If a link (specified by `_voteMessageHash`) length is equal to one
     *      epoch length (finalisation link) the inclusion principle should
     *      be satisfied:
     *          current block number is strictly less than
     *          target block height + epoch length
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

        // checks inclusion principle

        _registerVote(
            _voteMessageHash,
            _r,
            _s,
            _v
        );
    }
}

```

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



## meeting 3
date: 13/1/2020

note taker: deepesh

- Pro runs over user stories, Ben scrolls the page
- Ben explained about forward and rear validator concept  on white board (check the video)
- Deepesh: Proposing a link and votion on the link should be different? or the proposal should also increase the vote count? 
    - It will be different, anyone can propose, the validators will vote.
- Deepesh: Is validatorSet a contract? Yes, its will be contract and core and protocore will inherit it.