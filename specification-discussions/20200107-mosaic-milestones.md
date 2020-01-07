---
title: 'Mosaic Hadapsar generations'
disqus:
---

Mosaic Hadapsar Testnet Generations
===

| version | Last updated | Component           |
| ------- | ------------ | ------------------- |
| 0.14    | 07/01/2020   | Testnet generations |


**Editor:** Benjamin Bollen

---

## Overview

Hadapsar is the testnet for Mosaic-1. It is set up against Goerli testnet.
In the zero-th generation, it demonstrates the new faciliator to assist users deposit and redeem
ERC20 tokens in and from the metachains. In Gen0 Hadapsar runs with one metachain with metachainId `1405`.

In this document we outline the next milestones for following generations of the Hadapsar testnet.

## Milestones for Hadapsar Gen1

### (`Gen1-DX`) Developer Experience: "first prototype, wire all the piece together"

Outset: re-imagine the developer experience, but let's start with bare metal approach first. Meaning, wire all the new components together for a DApp developer to get started, but don't build any bells and whistles. It can be a bit laborious for a dev to get started at this stage; we'll learn from that, but keep the architecture lean to web3.

Dependencies: `Gen1-DX` has dependencies on `Gen1-BFT`, but takes priority over it. Therefore `Gen1-BFT` is minimised to achieve a validator set to anchor state roots in anchor contracts; this interface enables `Gen1-DX`

    - Gen0 ERC20Gateway and facilitator to connect to new (Gen1) anchors
    - Docs:instructions, example(s) (and architecture explanation for understanding)
    - DevOps:<bootnodes, fac, val, hosted public testnet RPCs>
    - MosaicDAO.org: block explorer and ethstats

### (`Gen1-BFT`) Byzantine Fault-Tolerance: "get a square wheel running"

Outset: if we boil the mechanics of Mosaic down to the essentials and strip away accountability and incentives, then there are three hierarchical supermajority votes. The first two occur by the weak-assumptions core validator group, the third and decisive one by the strong-assumptions committee validator group. Mosaic assumes a low incidence of BFT failures ("optimistic") and we exploit this to build up increasing security guarantees on increasingly longer timescales, enabling parallellisation of computation.

Therefore in `Gen1-BFT` the MVP is the triple hierarchy of "elaborate mutlisigs" with an open set of validators.

    - Contract:consensus
    - Contract:axiom
    - Contract:core
    - Contract:committee
    - Contract:reputation
    - Contract:consensusGateway
    - Contract:anchor
    - Contract:selfProtocore
    - Contract:originProtocore
    - Contract:coconsensus
    - Contract:originAnchor
    - Contract:coReputation
    - Contract:consensusCogateway
    - Contract:utMOST
    - Genesis:generate GenesisFile
    - Genesis:bootstrap Clique network given GenesisFile and published bootnode endpoints
    - Validator:Services -- bare minumum
        - achieve supermaj on reported links
        - achieve supermaj on proposed metablocks
        - achieve supermaj on committee decision

## Milestones for Hadapsar Gen2

### (`Gen2-DX`) "StarGateway" (working title, TBD)

    - Contract:ERC20Stargateway

### (`Gen2-BFT`) (TBD)
