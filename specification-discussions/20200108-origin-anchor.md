---
title: 'Origin anchor'
disqus: https://hackmd.io/_ycJbLOSSACSTGBGnJWSsw
---

Origin anchor
===

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 08/01/2020    | Origin anchor |

**Meeting date/time:** N/A

**Editor:** Deepesh Kumar Nath

**Team:** N/A

---

## Overview
The gateways like Consensus gateway, ERC20 gateway, etc establishes a channel for protocol messages to be communicated from the origin chain to a metachain/core, and back. 
~~When the proposed metablock is committed on the `CoConsensus` contract, the state root from origin chain is stored in the `OriginAnchor` contract.~~

The origin protocore observes the origin chain and keeps finalizing the blocks using casper FFG. Similarly the protocore observes the auxiliary chain (self chain) and keeps finalizing the blocks. 

Once the block of the origin chain gets finalized by the origin protocore in a block then this block is finalized by the protocore on the self chain, then the state root for such blocks can be anchored in the `OriginAnchor` contract.

An RLP encoded block data should be provided while anchoring the state roots.

## Goals


## Assumptions

## Out of scope


## Open questions

## Approach
---
## Meeting notes
### Meeting 1
date/time:
attendees:
