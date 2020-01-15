---
title: "Mosaic Subgraph"
disqus: "https://hackmd.io/POS1wkpzTuKu5QW8TQJPkQ"
---

# Mosaic Subgraph

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 13/01/2020   | Mosaic-1 subgraphs |

**Editor: Jayesh Bairagi**

---

## Overview

The Graph Service will be responsible to extract (event) data for the origin chain or a metachain into a PostgreSQL. The validator will be able to subscribe and make GraphQL queries to compose a local view of origin and a set of metachains, needed to operate.

`Mosaic-origin-subgraph` repository can contain the subgraph for the subgraph on Ethereum (mainnet) and Goerli (Hadapsar testnet).

`Mosaic-metachain-subgraph` repository can contain the (templated) subgraph for any metachain.

## User Stories for Mosaic-origin-subgraph
(Gen1-BFT)
- as a validator, I want to access (event) data from
    - Axiom
    - Consensus
    - Reputation
    - Cores
    - Committees
    - Anchors
    - ConsensusGateways
    - MOST
   
- As the graph I expect
  - The contract addresses of
    - Axiom
    - Consensus
    - Core
  - The event handlers and the entities for
    - MetachainCreated
    - ValidatorJoinedDuringCreation
    - RootOriginObservationBlockHeight
    - EndpointPublished

## User Stories for Mosaic-metachain-subgraph
(Gen1-BFT)
- as a validator, I want to access (event) data from
    - Coconsensus
    - OriginProtocore
    - SelfProtocore
    - OriginObserver
    - Coreputation
    - ConsensusCogateway
    - utMOST

