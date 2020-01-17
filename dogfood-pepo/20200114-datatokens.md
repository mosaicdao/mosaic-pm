---
title: "Pepo data tokens"
disqus: https://hackmd.io/9VS3xSdxQ_OUViHpbwOIfw
---

# Pepo: create videos as data tokens

| version | Last updated | Component   |
| ------- | ------------ | ----------- |
| Gen1    | 13/01/2020   | Data tokens |

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

Create an environment for developers that garantuees that they have ownership in the Pepo ecosystem: all data, and logic should be decentralised and owned by the Pepo users.

## User Stories

### Gen1-DX

Constraint: Pepo devs should be able to deploy their DApps that interact with real 

- As a pepo dev, I should be able to read metadata and relationships/replies of videos.
- As a pepo dev, I should be able to know all rewards of all videos. (PepoReward is NDT)
- As a pepo dev, I can deploy contracts (interacts) to Goerli (Gen-1)
- As a pepo dev, I can run simulations (Gen-1)
- As a pepo dev, I should be able "to know" all pepo videos (Gen-1)
- As a pepo dev, I should be able to get all pepo videos (Gen-1)

## Proposed implementation

```solidity
contract Pepos is EIP1948 {
    function initiateOwnershipTransfer(uint256 _tokenId, address _proposedOwner);
    function confirmOwnershipTransfer(uint256 _tokenId);
    function burn(uint256)
}
```

```solidity
contract PepoRewards
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

