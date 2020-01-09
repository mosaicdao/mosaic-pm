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

The origin anchor is an immutable registry for state roots of origin (Ethereum) blocks as they have been observed by the core validator group on the auxiliary chain.

The anchored state roots in the Origin anchor can be consumed by gateway contracts to confirm messages in the inbox on the auxiliary chain when declared in the outbox on the origin chain.

The anchor contract has a cyclical memory layout, overwriting the oldest entries when full. New entries can only be written for increasing block number.

Coconsensus has permission to anchor new state roots into the Origin Anchor.

## Goals
The origin anchor should be able to do the following
- The origin anchor should store the state roots and its block heights.
- The origin anchor should store `n` number of latest state roots. The insertion of the latest state root will remove the oldest stored state root.
- Any contract should be able to do the following
    - Read the latest state root.
    - Read the latest block height.
    - Read the state root for the given block height.
- Only a coconsensus contract should have permission to add the latest state root.
- The origin anchor contract should reject the storage root for the block heights that are equal or less than the latest stored block height.

## Assumptions

## Out of scope


## Open questions

## Approach
---
## Meeting notes
### Meeting 1
date/time:
attendees:
