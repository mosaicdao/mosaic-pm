---
title: "Mosaic-1 Graph Service"
disqus: "https://hackmd.io/POS1wkpzTuKu5QW8TQJPkQ"
---

# Mosaic1 Graph Service

| version | Last updated | Component              |
| ------- | ------------ | ---------------------- |
| 0.14    | 13/01/2020   | Mosaic-1 Graph Service |

**Editor: Jayesh Bairagi**

---

## Overview

The Graph Service will be responsible to sync the mainnet and save the required event data for the contracts(Core and the Consensus) to the PostgreSQL. The validator will be able to retrieve all the relevant data through GraphQL queries.

## User Stories

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
