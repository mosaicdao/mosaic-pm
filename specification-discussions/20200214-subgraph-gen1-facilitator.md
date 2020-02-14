---
title: "Subgraph Gen1 Facilitator"
disqus: https://hackmd.io/J5nwK6zySlm4204yIKnc1w
---

# Subgraph Gen1 Facilitator

| version | Last updated | Component                 |
| ------- | ------------ | ------------------------- |
| 0.14    | 13/02/2020   | Subgraph Gen1 Facilitator |

## Overview

Subgraph will be responsible for indexing and querying data from blockchains. It makes it possible to query data that is difficult to query directly. It provides the facilitator Gen1 with the functionality to subscribe and react to the events of gateways.

## User stories

1. As a user of subgraph, I should be able to subscribe to the origin contracts(ERC20Gateway and Anchor)
1. As a user of subgraph, I should be able to subscribe to the auxiliary contracts(ERC20Cogateway and Coanchor)
1. As a user of subgraph, I should be able to deploy subgraph for origin.
1. As a user of subgraph, I should be able to deploy subgraph for auxiliary.

## Tasks
1. Create new repoository mosaic-subgraph

1.  Define all the entities in schema.graphql
    - DeclaredDepositIntent
      - id
      - token address
      - amount
      - nonce
      - beneficiary
      - fee gas price
      - fee gas limit
      - depositor
      - block number
      - block hash
      - contract address
      - updated time stamp (uts)
    - ConfirmedWithdrawIntent
      - id
      - message hash
      - block number
      - block hash
      - contract address
      - updated time stamp (uts)
    - DeclaredWithdrawIntent
      - id
      - token address
      - amount
      - nonce
      - beneficiary
      - fee gas price
      - fee gas limit
      - withdrawer
      - block number
      - block hash
      - contract address
      - updated time stamp (uts)
    - ConfirmedDepositIntent
      - id
      - message hash
      - block number
      - block hash
      - contract address
      - updated time stamp (uts)
    - ProvenGateway
      - id
      - proven block number
      - storage root
      - gateway address 
      - block number
      - block hash
      - contract address
      - updated time stamp (uts)
    - CreatedUtilityToken
      - id
      - value token
      - utility token
      - block number
      - block hash
      - contract address
      - updated time stamp (uts)

1. Define handlers 
    - in mapping for ERC20Gateway
      - DeclaredDepositIntent, 
      - ConfirmedWithdrawIntent
      - ProvenGateway
    - in mapping for ERC20Cogateway
      - ConfirmedDepositIntent
      - DeclaredWithdrawIntent
      - CreatedUtilityToken, 
      - ProvenGateway
    - in mapping for Anchor.
      - StateRootAvailable
    - Add custom fields for entities in ERC20GatewaySchema and ERC20CogatewaySchema, AnchorSchema.
    - Ref: (https://github.com/mosaicdao/mosaic-chains/blob/develop/graph/origin/src/EIP20GatewayMapping.ts#L28)

1. Add 
    - ABIs of 
      - ERC20Gateway contract
      - ERC20Cogateway contract
      - Anchor contract
    - Events in subgraph.yaml/subgraph.mustache.yaml
      - DepositIntentDeclared event
      - WithdrawIntentConfirmed event
      - GatewayProven event
      - UtilityTokenCreated event
    - Addresses of deployed the contracts.

1. Add command to deploy subgraph for each chain.

## Assumptions

N/A

## Out of scope
N/A
