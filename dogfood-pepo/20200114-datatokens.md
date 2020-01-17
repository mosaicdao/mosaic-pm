---
title: "Pepo data tokens"
disqus: https://hackmd.io/9VS3xSdxQ_OUViHpbwOIfw
---

# Pepo: create videos as data tokens

| version | Last updated | Component   |
| ------- | ------------ | ----------- |
| Gen1    | 17/01/2020   | Data tokens |

**Editor:** Benjamin Bollen

## Overview

Non-fungible data tokens introduce a writable data field of 32 bytes non-fungible token. Described in [EIP-1948](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1948.md), the owner of the data token can write to the data field.

We can create a Pepo videos contract that inherits EIP-1948. Ownership transfers of Pepo videos should be a two-step process `initiateOwnershipTransfer` from the old owner and `confirmOwnershipTransfer` by the new owner.

Videos uploaded through the Pepo app can be uploaded on IPFS using an IPLD format. The format should allow specifying metadata of the video and the CID of the video itself, append-only to edit, and ability to link to other videos (replies would be the first relationship).

A `Pepos` data token in the contract can be created by a `host` hosting service (here Pepo app backend) and can initiate ownership transfer from the host to the Pepo user (`TokenHolder`). Currently the user cannot (yet) accept the ownership transfer, but this way the host can already indicate the intended ownership, while still maintaining the ability to update the data token.

### Removing content

Beyond editing metadata of a video, the owner should be able to indicate a desire for the content to no longer be hosted. Honest services can listen to this and stop hosting the IPFS content related to this data token.

The content can be "removed" in this sense, by burning the data token by the owner.

## Milestones

1. Developer Experience: Make Pepo hackable

Create an environment to encourage developers to build and have ownership in the Pepo ecosystem: data, and logic should be increasingly decentralised and owned by the Pepo users. In the first milestone we want to register also the video data on-chain to open up all hacking on Pepo.

## User Stories

### Gen1-DX

Constraint: Pepo devs should be able to deploy their DApps that interact with real Pepo data on Goerli testnet.
The Pepo app should not yet need to call on new contracts in this generation.

1. As a pepo dev, I should be able "to know" all pepo videos that are published and the publisher
1. As a pepo dev, I should be able to get all pepo video data (from IPFS).
1. As a pepo dev, I should be able to read metadata and relationships/replies of videos.
1. As a pepo dev, I should be able to know all rewards of all videos.
1. As a pepo dev, I should be able to deploy my DApp to Goerli and interact with a (replay / live) simulation of mainnet Pepo data.

### GenX-DX

TBD: for the Pepo users to claim ownership of their Pepos on-chain, the Pepo app must sign transaction for new data token
contracts and other custom token rules. Assuming this constraint is aligned:

- As a pepo user, I should be able to own my videos
- As a pepo user, I should be able to transfer ownership of my videos
- As a pepo user, I should be able to "delete" my videos (obeyed by all honest IPFS pinning services)

## Bounty proposals

### Gen1-DX
-

## Proposed implementation

```solidity
contract Pepos is EIP1948 {
    function registerPepo(bytes32 _pepoCid) returns (uint256 pepoTokenId_);
    function initiateOwnershipTransfer(uint256 _pepoTokenId, address _proposedOwner);
    function confirmOwnershipTransfer(uint256 _pepoTokenId);
    function burn(uint256);
}
```

```solidity
contract PepoRewards is EIP1948 {
    function registerReward(uint256 _pepoTokenId, bytes32 _txHash) returns (uint256 rewardTokenId_)
}
```

---
## Meeting Notes

Date: 16/01/2020

Attendees: Abhay, Gulshan, Jayesh, Deepesh, Pro, Ben
Note taker: Jayesh, Pro

### User Stories

- As a pepo dev, I should be able to read metadata and relationships/replies of videos. (Gen-1)
- As a pepo dev, I should be able to know all rewards of all videos. (PepoReward is NDT) (Gen-1)
- As a pepo dev, I can deploy contracts (interacts) to Goerli (Gen-1)
- As a pepo dev, I can run simulations (Gen-1)
- As a pepo dev, I should be able "to know" all pepo videos (Gen-1)
- As a pepo dev, I should be able to get all pepo videos (Gen-1)
- As a pepo user, I should be able to own my videos (Gen-2)
- As a pepo user, I should be able to transfer ownership of my videos (Gen-2)
- As a pepo user, I should be able to "delete" my videos (Gen-2)
