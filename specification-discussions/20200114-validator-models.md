---
title: "Validator"
disqus: https://hackmd.io/CkFgqDFORvac2AiPFTH0Yw
---

# Validator

| Version | Last Updated | Component |
| ------- | ------------ | --------- |
| 0.14    | 13/01/2020   | Validator   |

**Meeting date/time:** N/A

**Editor:** Deepesh Kumar Nath

**Team:** N/A

---

## Overview

## User stories
1. As a validator, I should be able to subscribe to the following entities from the graph node (origin chain).
    - CreatedMetachain(s) -> metachainid, anchor, core, consensus gateway
    - UpdatedMetachainState(s) -> metachainid, lifetime status.
    - JoinedValidator(s)-> metachainId, core, validator address, begin height, end height, reputation, staked ETH/MOST
    - UpdatedValidatorStatus(es)-> validator address, status
    - UpdatedReputation(s) -> validator address, reputation, 
    - UpdatedCoreStatus(es) -> metachainId, core, status.
    - AnchoredStateRoot(s) -> metachainId, anchor, blocknumber, stateroot
    - DeclaredOpenKernel(s) -> < TBD >
    - ProposedMetablock(s) -> < TBD >
    - RegisteredVote(s) -> < TBD >
    - PrecommittedMetablock(s) -> < TBD >
    - FormedCommittee(s) -> < TBD >
    - EnteredCommittee(s) -> < TBD >
    - SubmittedSealedVote(s) -> < TDB >
    - RevealedCommit(s) -> < TDB >
    - CommitteeDecision(s) -> < TBD >
    - CommittedMetablock(s) -> < TBD >
2. As a validator, I should be able to subscribe to the following entities from the graph node (auxiliary chain).
    - < TBD >

3. As a validator, I should be able to re-subscribe to the graph node if the subscription breaks.

4. As a user, I should be able to deposit OST/MOST in the consensus gateway contract on origin chain, so that I can mint utMOST on the auxiliary chain.

5. As a validator, I should be able to stake OST/MOST and WETH tokens while joining the metachain/core.

6. As a validator, I should be able to join the metachain/core before the core moves to the open status, and deposit OST/MOST.

7. As a validator, I should be able to join the metachain/core when the core is open or precommitted.

8. As a validator, I should be notified when the minimum number of validators joins the metachain/core, so that I can create the genesis file and start the auxiliary chain node. (wait N ethereum blocks)

9. As a facilitator, I should be notified when the message for the opening of the new kernel is declared in the consensus gateway contract on the origin chain. (just store the declared message)

10. As a validator, I should be able to observe the blocks generated on the origin chain so that I can report the origin chain links in the OriginProtocore contract on the auxiliary chain, every epoch length. (source must be justified)

11. As a validator, I should be able to observe the blocks generated in the auxiliary chain so that I can report the auxiliary chain links in the SelfProtocore contract on the auxiliary chain, every epoch length. (source must be justified)

12. As a validator, I should be notified when the gas consumed in the auxiliary chain will reach the gas accumulation target, so that I can propose a new metablock in the core contract on the origin chain. (for now keep this logic simple, like every dynasty mod m)

13. As a validator, I should be notified when the target checkpoint of a link on the OriginProtocore is justified/finalized.


14. As a validator, I should be notified when the target checkpoint of a link on SelfProtocore is finalized, so that I can anchor the origin state root in the origin anchor (if any are present).

15. As a validator, I should be able to propose new metablock in the core contract on the origin chain.

16. As a validator, I should be able to register votes for the new metablock proposals in the core contract (taken from registered votes on Protocore; Ben: should we store these signatures or only which validators have voted?).

17. As a validator, I should be notified when the quorum is reached for a new metablock proposal (metablock is precommitted, store the committee formation height block number).

18. As a validator, I should be able to form a new committee, after the committee formation block height is reached.

19. As a validator, I should be notified when the status of committee changes.

20. As a validator, I should be able to calculate the correct members that should enter the committee.

21. As a validator, I should be able to enter a new committee.

22. As a validator, I should be able to add other validators in to a committee.

23. As a validator, I should be notified the validators that are added in to a committee. (so that I can track and challenge the committee if a validator has not entered)

24. As a validator, I should be notified when a new core is created. (so that I can create and store a genesis file)

25. As a validator, If I entered any committee then I should be able to start the auxiliary chain node for which the committee was formed, and subscribe to the the mosaic subgraph for this auxiliary chain.

26. As a validator, I should be able to challenge the committee if a validator has not entered the committee (during the committee cool down period).

27. As a validator and a member of a committee, I should be able to activate a committee.

28. As a validator and a member of a committee, I should be able to submit a sealed commit. (After verifying the auxiliary chain, till the direct (checkpoint) child of the target checkpoint of the votemessage which precommitted the metablock.)

29. As a validator, I should be able to stop the auxiliary chain after the sealed commit was submitted.

30. As a validator and a member of a committee, I should be able to reveal my position by providing the salt that was used in the sealed commit.

31. As a validator I should be notified about the vote messages reported by other validators on the OriginProtocore, so that I can provide the signatures to slash the validator thats is violating the slashing conditions. (Post GEN1-BFT)

32. As a validator I should be notified about the vote messages reported by other validators on the SelfProtocore, so that I can provide the signaturs to slash the validator thats is violating the slashing conditions. (Post GEN1-BFT)

33. As a validator, I should be able to logout.

34. As a validator, I should only *ever* vote for links I have calculated myself.

35. As a validator, I should only EVER sign vote messages that do not violate slashing conditions with respect to the previously signed vote messages.


## Questions:

1. How will the validator executable start? 
    - Validator can join any metachain by providing the following.
        - metachain id
        - Graph node endpoint.
    - Validator can start the executable without metachain id (interactive)
        - Graph node endpoint.
3. Should validators observe the new metachain chain creation event.

## Models
        
    - Anchor
        - metachainid
        - state root
        - block height
        
    - Core
        - metachain id
        - core status
        - min validators
        - max validator
        - origin observation block height
        
    - Validator
        - metachain id
        - reputation
        - begin height
        - end height
        - status
        - withdrawalAddress
        - lockedEarnings
        - cashableEarnings
        - withdrawalBlockHeight

## Services
- ConfirmDeposit service
    - Anchor block number should be greater than sourceBlockHeight
    - Calcualtes proof
    - Calls ConfirmDeposit transaction on aux chain


## meeting 4
Date: 14/1/2020

Note taker: Jayesh Bairagi

- When there's a precommit we should store the height. This way we will know the block number
- We can have a service/entity to check whatever changes we need at every block/at that particular stage
- Need to keep track of the coldown period to organize the validators that should enter the committee, during this period any one can start the challenge regarding to joining of any validator
- All the validators should keep track of all the other validators in the committee
- There can be pause once the validator start the process of starting the Aux chain as there will be processes like the syncing will be involved
- As a user anyone can challenge if a validator has not joined the committee

## Meeting 5

Date: 14/01/2020

Note Taker: Abhay Singh

### Graph database to explore
    - neo4j
    - https://www.arangodb.com/

### GraphQL entities
- Deposit(s)
    - depositor
    - deposit amount
    - beneficiary
    - feeGasPrice
    - feeGasLimit
    - nonce
  Message hash can be calculated
  
### Handlers
    - DepositHandler
        - Creates deposit model
        - Save Deposit
        - Create message model
        - Save MessageRepository
        
    - StateRootAvailableHandler
        - Saves latest block height in auxiliary chain model


### Models/Repositories
    - Metachain
         - metachainId - PK
         - origin chain id (it can be constant)
         - anchorGA
         - mosaic version
         - consensus address
        
    - Core
         - coreGA - PK
         - metachainId
         - Lifetime status
         - Core status
         - open metablock height
         - minimum validator count
         - maximum validator count
         - gasTarget
         - genesisOriginObservationBlockNumber
         - ? current validator count
         - ? quorum
         
      - UntrustedEndpoint
          - endpoint - PK
          - validatorGA
          - serviceType
          - metachainId
                    
      - Genesis(files)
          - coreGA - PK
          - CID
          
      - Committee
          - committeeGA - PK
          - metablockHash
          - ? metachainId
          - <s> formation height</s>
          - seed
          - status
          - decision
          - size
          - quorum
      
      - Validator
          - validatorGA - PK
          - coreGA
          - beginHeight
          - endHeight
          - reputation
          - status
          - withdrawal block height
          
      - Link
          - voteMessageHash - PK
          - parent
          - child // @qn (pro): This should be an array, or we can have a isLeaf/isTip instead.
          - targetBlockNumber
          - targetBlockHash
          - targetFinalizationStatus
          - sourceTransitionHash
          - proposedLinkHeight
          - metachainId
          - coreGA
          - selfDynasty // @qn (pro): Is this sourceDynasty
          
      - Metablock 
          - metablockHash - PK
          - metachainId
          - metablockHeight
          - status
          - kernelHash
          - transitionHash
          - sourceBlockHash
          - targetBlockHash
          - sourceBlockNumber
          - targetBlockNumber
          - round
          - roundMetablockNumber
          
      - Message
          - message hash
          - message type
            - Deposit
            - Withdraw
            - Open new kernel
            - new ERC20Gateway
            - activate gateway
          - intent hash
          - source status
            - Undeclared
            - Declared
          - target status
            - Undeclared
            - Declared
          - fee gas price
          - fee gas limit
          - source gateway global address
          - source declaration block height
          
       - SignedVoteMessage
           - signature - PK
           - voteMessageHash
           - CoreSealInclusion
               - Unknown
               - Included
               - Rejected 
           - ProtocoreSealInclusion
               - Unknown
               - Included
               - Rejected
                        
      - Gateway
          - Gateway global address - PK
          - Remote gateway global address
          - Gateway type
              - consensus
              - most
              - erc20
          - Destination global address
              - Consensus
              - Most
              - erc20
              - NFT
           - Remote gateway last proven block number
           - AnchorGA    
           
    - ValidatorDepositIntent       
        
    - DepositIntent
        - Deposit intent hash
        - message hash
        - deposit amount
        - beneficiary
        
    - GenesisDepositIntent
        - Deposit intent hash
        - message hash
        - deposit amount
        - beneficiary
        
    - WithdrawIntent
        - withdraw intent hash
        - message hash
        - amount
        - beneficiary
    
    - OpenKernelIntent 
        - open kernel intent hash - PK
        - message hash
        - kernel height
        - kernel hash
        
    - AnchoredStateRoots
        - StateRootProviderGA - PK
        - latestAnchoredBlockNumber
        
      - Anchor can have multiple gateways
      - Get gateway address for the anchor from Gateway model   
        
    - Member
        - memberGA - PK
        - validatorGA
        - metablockHash (Committee identifier)
        - CalculatedRank
            - position where validator should be there
            - it should not be current position
        - status
            - Missing
            - Entered
            - Committed
            - Revealed
            - Ejected
        - position
      
    - Protocore
        - protocoreGA - PK
        - openMetablockHeight
        - coreGA
        - ? metachainId
      
    - Kernel
          - kernelHash
          - coreGA
          - (metachain Id)
          - Kernel height
          - parent
          - updated validators
              - json type
              - delta validators
              - delta reputation
          - gas target
          - status - it's not in contract
              - opened(core)
              - declared(core)
              - confirmed(protocore)
              - CoOpened(protocore)
              - Committed(core)
          
      - TransitionObject
          - transition hash - PK
          - coreGA
          - blockHash
          - kernel hash
          - latest observation of origin
          - dynasty
          - accumulated gas
          - committeeSolution
          - status
              - broadcasted
              - calculated
              - proposed
      
      - CoConsensusi
          - CoreGA
          - (metachainId)
    
      @qn (pro): Do we miss coconsensus global address in CoConsensusi?
          
      
      - ERC20Gateway?
          - gateway
          - remoteGateway
          - metachainid
          - type / direction
              - origin
              - auxiliary
          - token    
          - isActive
          
       - AuxiliaryChain?
         - Metachainid(aux side)
         - Origin Metachainid(Anchor address is 0)
         - originChainName
         - originAnchor (on auxiliary)
         - auxiliaryAnchor (on origin)
         - lastOriginBlockHeight
         - lastAuxiliaryBlockHeight  
         
      - TrustedEndpoint
          - endpoint - PK
          - metachainId
          - serviceType
              - enode
              - ipfs
              - GraphWs
              - GraphRpc
              - ChainRpc
          - ....    
     
### Notes: 
  
  - OriginProtocore
  - OriginObserverBlock is Anchor
      - It observe state root
      - OriginObserver.observeCheckPoint()
      - anchorSR

## Handlers
1. Implement `ConsensusGateway::Deposit` handler
        - Creates entry in DepositIntent model
        - Save DepositRepository
        - Create entry in Message model
        - Save MessageRepository
     
## Validator services
**PM document:** [Validator PM Document](https://github.com/mosaicdao/mosaic-pm/blob/master/specification-discussions/20200114-validator-models.md#user-stories)

1. Implement Deposit Service 
    - It should call `ConsensusGateway::deposit()` method to stake OST/MOST and WETH tokens for joining the metachain/core.
    
1. Implement JoinBeforeCreation Service 
    - It should call `Core::joinBeforeCreation()` method.
    
1. Implement JoinAfterCreation Service
    - It should call `Core::join()` to join the metachain/core when the core is open or precommitted. 

1. Implement OriginProtocoreProposeLink Service
    - It should call `OriginProtocore::proposeLink()`
    - It should observe the blocks generated on the origin chain so that I can report the origin chain links in the OriginProtocore contract on the auxiliary chain, every epoch length. (source must be justified)

1. Implement SelfProtocoreProposeLink Service
    - It should call `SelfProtocore::proposeLink()`
    - It should observe the blocks generated in the auxiliary chain so that I can report the auxiliary chain links in the SelfProtocore contract on the auxiliary chain, every epoch length. (source must be justified)

1. Implement ProposeMetablock Service
    - It should call `Core::proposeMetablock()`
    - It should be able to propose new metablock in the core contract on the origin chain.

1. Implement RegisterVote Service
    - It should call `Core::registerVote`
    - It should be able to register votes for the new metablock proposals in the core contract 

1. Implement FormCommittee Service
    - It should call `Consensus::formCommittee`
    - It should be able to form a new committee, after the committee formation block height is reached.

1. Implement EnterCommittee Service
    - It should call `Consensus::enterCommittee`
    - Validators should also be able to add other validators also to a committee
    - It will call `AuxiliaryChainManager` library and should be able to start the auxiliary chain node for which the committee was formed, and subscribe to the the mosaic subgraph for this auxiliary chain.
    
    <b>Question:</b> Should AuxiliaryChainManager be a service?

1. Implement AnchorStateRoot Service
    - It anchor the origin state root in the origin anchor (if any are present).

1. Implement ChallengeCommittee Service
    - It should call `Committee::challengeCommittee()` during the committee cool down period

1. Implement ActivateCommittee Service
    - It should call `Committee::activateCommittee()`

1. Implement SubmitSealedCommit Service
    - It should call `Committee::submitSealedCommit()`
    - It should call submit a sealed commit after verifying the auxiliary chain, till the direct (checkpoint) child of the target checkpoint of the votemessage which precommitted the metablock)

1. Implement RevealCommit Service
    - It should call `Committee::revealCommit()`
    - It should be able to reveal position by providing the salt that was used in the sealed commit.

1. Implement Logout Service
    - It should call `Consensus::logout()`


## Validator other tasks(@abhay)

1. Explore Graph database - Gen1 needed?
    - It's needed to maintain relationships between metachain, core, anchor 
    - Below are possible options
        - Neo4j - https://neo4j.com/
        - ArangoDB - https://www.arangodb.com/

1. Define mosaic-subgraph entities
    - https://hackmd.io/CkFgqDFORvac2AiPFTH0Yw#User-stories 
    - Implement events in contracts based on entities

1. Implement mosaic-subgraph 
    - Create new repository
    - Add abis
    - Add schema.graphql
    - Implement subgraph.yml template. Template because contract addresses are dynamic. 
        - Define datasources
        - Define event signatures 
        - Define event handlers
        - Define entities 
    - Deploy subgraph
    - Implement event handlers
    - Call handlers?
    - Block handlers?  

<b>Question:</b> It's more than 3 pointer.

1. Implement Validator config (it's not manifest)
    - Origin chain geth endpoint
    - Other data points? 

1. Populate seed data
    - metachains
    - cores
    - genesis
    - anchor
    - protocores
    - consensusi

1. Implement Subscription libraries
    - It should be able to subscribe to the entities from the graph node.
    - We should create SubscriptionQuery class and define susbcription queries for the different entities
    - Implement GraphClient client library to intract with graph node
    - Implement Subscriber class with below methods
        - Subscribe
        - Unsubscribe

    <b>Question:</b> Move common libraries to new repository
1. Implement starting of validator
    - It does the subscription 
    - Restart subscription after specific interval
     
1. Implement stopping of validator
    - Stops unsubscription
    - Validator executable is gracefully stopped  

1. Implement CalculateMembers method/utility class
    - It should be able to calculate the correct members that should enter the committee. 

1. Implement AuxiliaryChainManager Library
    - Kubernetes for docker process management?
    - When the minimum number of validators join the metachain/core, It should create the genesis file and start the auxiliary chain node. (wait N ethereum blocks)
    - Stop the auxiliary chain after the sealed commit was submitted.
    - Refer [GenesisFileCreation](https://hackmd.io/sz-1wtHdTl6xU7aCwDsAsw)


