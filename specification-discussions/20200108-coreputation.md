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
`CoReputation` contract on the auxiliary chain does the bookkeeping of validators. This contract on the auxiliary chain mirrors a subset of the total validator set (minimally the core validators) of the `Reputation` contract on origin. With each metablock opening cycle, an array of updated validators and their reputation synchronises the origin reputation to the coreputation contract. This contract tracks the following for validators
- Validator status
    - Undefined
    - Slashed
    - Staked
    - Deregistered
- Reputation

The reputation and the status of the validators will be updated when the opening of the kernel is confirmed on the auxiliary chain and the proposed metablock is committed on the auxiliary chain.

New validators with a non-zero reputation in the kernel become `Staked` (`Undefined -> Staked`).
Staked validators with a zero reputation in the kernel are logged out, which in coreputation is marked as `deregistered` (`Staked -> Deregistered`).
Staked validators that violate slashing conditions will be instantly marked as `Slashed` (`Any -> Slashed`)

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
