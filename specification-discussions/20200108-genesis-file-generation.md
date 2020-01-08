---
title: 'Genesis file generation'
disqus: https://hackmd.io/sz-1wtHdTl6xU7aCwDsAsw
---

Genesis file generation
===

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 08/01/2020    | Genesis file generation |

**Meeting date/time:** N/A

**Editor:** Deepesh Kumar Nath

**Team:** N/A

---

## Overview
This will be an executable that will read the data from the core contract and generate the genesis file.

Tech governance can create a new metachain. This can be done by calling the `newMetachain` function on the `Axiom` contract on the origin chain. During this process, the contracts for the metachains i.e `Core`, `ConsensusGateway`, `Anchor` comes to existence on the origin chain but the auxiliary chain does not exist yet.

The auxiliary chain runs on clique POA network, so it becomes important to know the peer nodes so that they can become the sealers and seal the blocks on the same network, rather than running independently.

To form this network following things are required
- A genesis file
- Boot nodes to discover the peers
- Initial sealer addresses in POA network.

In addition to this, the initial validators/sealers will need balance (gas) (utMOST) on the auxiliary chain to perform any transaction.

The validators can join the newly created core (`joinBeforeOpen`). Before joining the core, the validators must deposit `OST/MOST` on consensus gateway so that they can receive equivalant amount of `utMOST` on auxiliary chain.
When the minimum number of validators are joined then the block number (`rootOriginObservationBlockHeight`) is stored in the `Core` contract.

Now the genesis file can be created by the executable by reading the data from the contracts. The following params are written in the genesis file
- `config.chainId`
    This is the metachain id (core contract)
- `timestamp`
    This is the timestamp of `rootOriginObservationBlockHeight` (core contract)
- `extraData`
    This will contain the address of all the validators that joined during the core.
- `alloc`
    This section of genesis file will contain the fillowing
    - precompiled contracts
    - validators address with initial balance, the balance will be equal to the staked amount of OST/MOST.

The address `0x00..000000` is reserved for the `utMOST` contract.
The addresses from `0x00..004d00` to `0x00..004dff` are reserved for the precompiled mosaic contracts.

The precompiled contracts will be allocated at the following addresses

```
0x00..000000 utMOST [proxy]

0x00..004d00 CoConsensus [proxy]
0x00..004d01 CoReputation [proxy]
0x00..004d02 ConsensusCoGateway [proxy]
0x00..004d03 ProtocoreSelf [proxy]
0x00..004d04 OriginObserver [proxy]
0x00..004d05 OriginAnchor [proxy]
...
0x00..004dfa OriginAnchor [mastercopy]
0x00..004dfb OriginObserver [mastercopy]
0x00..004dfc ProtocoreSelf [mastercopy]
0x00..004dfd ConsensusCoGateway [mastercopy]
0x00..004dfe CoReputation [mastercopy]
0x00..004dff CoConsensus [mastercopy]
```
The storage of each contract will have some initial storage data. These initial data for the precompiled contracts will be written in the genesis files.
```
< Storage of each contract to be updated >
```
Once the genesis file is created, the validators can start the chain. 
The validators will publish the endpoint of bootnodes in the `Consensus` contract, so that other nodes can discover the peers.

The template for genesis file is given below:
```
{
  "number": "0x0",
  "config": {
    "chainId": < metachainid >, 
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip150Hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "clique": {
      "period": 3,
      "epoch": 30000
    }
  },
  "nonce": "0x0",
  "timestamp": "< time stamp from block header of rootOriginObservationBlockHeight>",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000< all validator addresses >0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0x989680",
  "difficulty": "0x1",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {
    "0000000000000000000000000000000000000000": {      
      "balance": "< very large number >",
      "code": "0xcode_of_utMOST",
      "nonce": "1",
      "storage": {
        "0x0000000000000000000000000000000000000000000000000000000000000000":" < some value>",
        "0x0000000000000000000000000000000000000000000000000000000000000001":" < some value >"
      }
    },
    "< validator addresss 1 >": {
      "balance": "< Staked amount >"
    },
    "< validator addresss 2 >": {
      "balance": "< Staked amount >"
    },
    "0000000000000000000000000000000000000000000000000000000000004D<00-ff>": {
      "balance": "0",
      "code": "< contract code >"
      "nonce": "1",
      "storage": {
        "0x0000000000000000000000000000000000000000000000000000000000000000":" < some value>",
        "0x0000000000000000000000000000000000000000000000000000000000000001":" < some value >"
    }
  },
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
} 

```

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
