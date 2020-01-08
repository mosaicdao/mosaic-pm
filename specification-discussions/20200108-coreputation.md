---
title: 'CoReputation'
disqus: https://hackmd.io/bhxONpnpRZOTw3SgYr3TiQ
---

CoReputation
===

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 08/01/2020    | CoReputation |

**Meeting date/time:** N/A

**Editor:** Deepesh Kumar Nath

**Team:** N/A

---

## Overview
`CoReputation` contract on the auxiliary chain does the bookkeeping of validators. This contract on the auxiliary chain is in sync with the `Reputation` contract on origin for each metablock creation cycle. This contract tracks the following for validators
- Validator status
    - Undefined
    - Slashed
    - Staked
    - Deregistered,
    - Withdrawn
- Reputation

The reputation and the status of the validators will be updated when the opening of the kernel is confirmed on the auxiliary chain and the proposed metablock is committed on the auxiliary chain.

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
