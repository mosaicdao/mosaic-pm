---
title: 'Bootstrap clique POA network'
disqus: https://hackmd.io/-9gZLFxbQY-YIkkC63aNRg
---

Bootstrap clique POA network
===

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 08/01/2020    | Bootstrap clique POA network |

**Meeting date/time:** N/A

**Editor:** Deepesh Kumar Nath

**Team:** N/A

---

## Overview
When a new metachain is created, the initial validators that joined the core (`joinedBeforeOpen`) , generates the genesis file locally. The details of genesis file generation can be found [here](https://github.com/mosaicdao/mosaic-pm/blob/master/specification-discussions/20191209-genesis-contracts.md)

The auxiliary chain can be started using this genesis file. Each validator will start the auxiliary chain locally. The sealer addresses are already provided in the genesis file, so for the block generation to start atleast 51 percent of the sealers should join the POA network.

At this point the auxiliary chain node do not have any information about the peer nodes, so chain will wait for discovery of peer node untill atleast 51% of sealers are connected to the network.

Each auxiliary chain node that is run by the validators will publish the bootnode endpoints so that it is discoverable by the other nodes and they can join the POA network.

The bootnode endpoints are published in `Consensus` contract by calling `publishEndpoint` function. This function will read the inputs and emit the `EndpointPublished` event. The validators will listen/subscribe to this event and when the event is received the validators can add the published node as peers. 

The block generations on the auxiliary chain will start, once the 51 percent of auxiliary chain nodes joins the POA network.


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
