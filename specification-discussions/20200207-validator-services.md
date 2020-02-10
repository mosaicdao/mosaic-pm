---
title: 'Validator'
disqus: https://hackmd.io/Khym_2U4QoyU9-PZhXcaIQ
---

# Mosaic Implementation Discussion: Validator services and avatars

| version | Last updated | Component                      |
| ------- | ------------ | ------------------------------ |
| 0.14    | 07/02/2020   | Validator services and avatars |

Meeting date/time: Friday 7 december 2019

Editor: Benjamin Bollen

## Architecture

### Models

Models capture the observation of events as they happen on the origin chain and metachains. For now, we assume a simple world-view where we don't have chain-reorgs.  Meaning that all events that have happened, will eventually happen in some reorganised history.

### Subjective

Subjective models build history from the observations of events and predict the actions to be performed by the services. These models are updated by the services. At a particular threshold (time) the services read the data from the subjective models and act.

The subjective models will be needed for the following:
- Propose link
- Register vote
- Propose metablock

### Subgraphs entities

### Transaction/Entity handlers
Validator executable subscribes/queries the subgraph for the entities. The entities from the subgraph are received and handled by the transaction handlers. 
The transaction handler performs the following actions:
- Filter out the entities that are already processed.
- Create the model objects and save it in local DB.
- Call the specific entity handlers.

The entity handlers know what services are required to be triggered, entity handlers call the required services.

### Services
Services are the specific tasks that the validator can perform.
The tasks for the service can be:
- Update the subjective model.
- Perform ethereum transactions.

There can be cases where the services do not perform ethereum transactions immediately, rather it builds a history from the received input and predicts the transactions that can be executed.
After some threshold, the validators can perform an ethereum transaction.

Following are the detail of services:
- Deposit
    - <check if this service is needed, this can be a command>
- Join before creation
    - This service will do the following
        - Approve reputation contract for OST transfer.

### Avatars
The validator executable can hold more than one ethereum account for different purposes. For example, the account for `withdraw`, `deposit`, `validator` can be different. The term avatar is introduced to differentiate between the accounts based on its purpose. The ethereum transactions can be performed by different avatar keys.

___
### Validator configuration

The validator executable can run in 3 modes
- Validator
    - The executable runs in a full validator mode.
- Observer
    - The executable runs as a observers, it observes the metachain and updates the local database.
- Facilitator
    - The executable run as a facilitator and performs the confirmation of deposits, withdraw.

A `yaml` file can be used to configure the validator. Below is an example
```yaml

version: 0.0.1

mode: <observer/facilitator/validator>

origin_metachain_id: <origin_metachain_id>

self_metachain_id: <self_metachain_id>

contracts: 
    axiom: 0xAxiomAddress 
    core: 0xCorAddress

end_points:
    <origin_metachain_id>:
        subgraph: <graph_endpoint>
        web3: <web3_endpoint>
    <self_metachain_id>:
        subgraph: <graph_endpoint>
        web3: <web3_endpoint>

withdrawal_address: 0xabc
validator_address: 0xpqr
depositor_address: 0xxyz

accounts: 
    0xabc: 
        keystore: file://
        password: file://
    0xpqr: 
        keystore: file://
        password: file://
    0xxyz: 
        keystore: file://
        password: file://

```

____
For later discussion
### Validator experience:
- Start a local server at http://localhost:77/
- Visualization of latest observation.
- UI tool to create configuration (yaml) file.
- Display all the endpoints for the metachains.
