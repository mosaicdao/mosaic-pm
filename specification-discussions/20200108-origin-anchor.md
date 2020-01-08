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


## Assumptions

## Out of scope


## Open questions

## Approach
---
## Meeting notes
### Meeting 1
date/time:
attendees:
