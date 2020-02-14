---
title: "Task breakdown 2"
disqus: https://hackmd.io/nQqBmXPBQauKDrEfo-6eLQ
---

# Task Breakdown 2

| Version | Last Updated | Component        |
| ------- | ------------ | ---------------- |
| 0.14    | 13/02/2020   | Task Breakdown 2 |


## ERC20Gateway and ERC20Cogateway
**HackMD document:** [Star Gateway HackMD Document](https://hackmd.io/vs6eBAk7QAK_4gQZbusqag)

1. ERC20Gateway::depositERC20
1. ERC20CoGateway::withdrawERC20
1. ERC20Cogateway::confirmDepositERC20
1. ERC20Gateway::confirmWithdrawERC20
1. ERC20GatewayBase::proveGateway
1. Implement integration test for deposit flow
1. Implement integration test for withdraw flow
1. Generate interacts and publish npm

## Facilitator

1. Update existing code base for creation of personas

Facilitator manifest:
```yaml=
version: "v0.14",
personas:
    m0_facilitator=false,
    m1_facilitator=true
chain: // section for m0 utility chains
metachain: // section metachains for m1
    auxiliary:
        avatar_account: "",
        node_endpoint: "",
        graph_ws_endpoint: "",
        graph_rpc_endpoint: ""
    origin:
        avatar_account: "",
        node_endpoint: "",
        graph_ws_endpoint: "",
        graph_rpc_endpoint: ""
accounts:
    "0xac7E36b3cdDb14Bf1c67dC21fFB24C73d03d8FF7":
        keystore_path: "",
        keystore_password_path: ""
origin_contract_addresses:
    erc20_gateway: ""
facilitate_tokens:
    - 0xA
    - 0xB
```

Notes:

1. Restructure repository to support different personas (don't define modes, it s an unneeded abstraction, be explicit about which personas to run in manifest)

2. Define facilitator manifest

3. Implement facilitator manifest schema

4. Facilitator init:
    1. default mode will be star gateway facilitator; optionally `--persona m0_facilitator=true` option can be used to toggle personas on or off;
    2. password will be taken from file
    3. keystore will be generated.
    4. command should provide instruction to backup, fund keys and file path
    5. Populate db path
    6. Populate seed data

```bash=
facilitator init \
    --origin-endpoint origin_endpoint \
    --aux-endpoint auxiliary_endpoint \
    --origin-graph-rpc-endpoint origin_graph_rpc_endpoint \
    --origin-graph-ws-endpoint origin_graph_ws_endpoint \
    --aux-graph-rpc-endpoint aux_graph_rpc_endpoint \
    --aux-graph-ws-endpoint aux_graph_ws_endpoint \
    --gateway-address gateway_address \
    --origin-password-file origin_password_file \
    --aux-password-file aux_password_file \
    --output-path path_of_manifest
    --facilitate-tokens 0xA,0xB
```

5. Facilitator start: Default will be star gateway facilitator.

```
facilitator start manifest.yaml
```

6. Gracefully stop facilitator

### Models & Repositories

- Implement DepositIntent model and DepositIntent repository.
- Implement WithdrawIntent model and WithdrawIntent repository.
- Implement AnchoredStateRoots model and AnchoredStateRoots repository.
- Implement Gateway model and Gateway repository.
- Implement Message model and Message repository.
- Implement ContractEntity model and ContractEntity repository.

### Services

- Implement ConfirmDeposit service
- Implement ConfirmWithdraw service
- Implement ProveGateway service


### Entity Handlers

- Implement DeclaredDepositIntent handler
- Implement ConfirmedDepositIntent handler
- Implement DeclaredWithdrawIntent handler
- Implement ConfirmedWithdrawIntent handler
- Implement StateRootAvailable handler
- Implement GatewayProven handler

### Integration Tests

- Implement integration test for deposit flow for ERC20token
- Implement integration test for withdraw flow for ERC20token

### Facilitator Other Tasks

- Implement ERC20GatewayAvatar
- Readme update

### Implement System Test
Reference: https://github.com/mosaicdao/facilitator/issues/206

### Demo scripts

- Implement script for creating depositer and withdrawer
- Implement script for deposit
- Implement script for withdraw
- Readme update

## Subgraph entites
**HackMD document:** [Subgraph entites HackMD Document](https://hackmd.io/J5nwK6zySlm4204yIKnc1w)

1.  Define all the entities in schema.graphql

2. Define handlers
    - in mapping for ERC20Gateway i.e DeclaredDepositIntent, ConfirmedWithdrawIntent(https://github.com/mosaicdao/mosaic-chains/blob/develop/graph/origin/src/EIP20GatewayMapping.ts#L28)
    - in mapping for ERC20Cogateway i.e. CreatedUtilityToken, ProvenGateway, ConfirmedDepositIntent, DeclaredWithdrawIntent. Also, for anchors.
    - Add custom fields for entities in ERC20GatewaySchema and ERC20CogatewaySchema, AnchorSchema.

3. Add ABIS of ERC20Gateway, ERC20Cogateway contract. Add DepositIntentDeclared, WithdrawIntentConfirmed, GatewayProven, UtilityTokenCreated events in subgraph.yaml/ subgraph.mustache.yaml.
Deploy the contracts as well.

4. Add command to deploy subgraph for each chain.

<b>Question:</b>
Placeholder for mosaic subgraph?

## Faucet


## Blockscout explorer

- Support explorer to provide a view for all the transactions for hadapsar Gen1 testnet.

## Eth stats

- Eth stats support with new hadapsar Gen1 testnet.

## Devops

- Deploy ERC20Gateway and ERC20Cogateway contracts
- Setup star gateway facilitator



## Follow-up tickets

- Add the following events:
    - `ConsensusGateway::OpenKernelDeclared`.
    - `ConsensusCogateway::OpenKernelConfirmed`.
    - `Protocore::LinkTargetFinalizationStatusUpdated`.
    - `Core::KernelOpened`.
    - `Coconsensus::KernelCoopened`.
    - `Coconsensus::ProtocoreSetup`.
    - `Consensus(Co)Gateway::RemoteGatewayProven`.
    - `ConsensusGateway::Deposited`.
    - `ConsensusCogateway::DepositConfirmed`.
    - `ConsensusCogateway::Withdrawn`.
    - `ConsensusGateway::WithdrawalConfirmed`.

- Implement `ConsensusGateway:: genesisDeposit`
    - This function will be called by the validator to deposit OST before the core is opened (metachain is created)
    - Emit `GenesisDeposited` event.

- Update test cases for SelfProtocore::upsertValidator
  - Add test cases to check more edge cases covering combinations of
    - validator,
    - different values of height,
    - and, different values of reputation
    - Ref: https://github.com/mosaicdao/mosaic-1/pull/230#discussion_r373202738

- Update validator mapping in ValidatorSet contract
  - Make validators mapping in ValidatorSet private.
  - An external view function iterates over validator to return array of validators.
  - Ref: https://github.com/mosaicdao/mosaic-1/pull/230#discussion_r372434058

- Implement GenesisConsensusCogateway contract
  - Implement `setup` function
  - `ConsensusCogateway` contract should inherit `GenesisConsensusCogateway` contract
  - Update the setup function of `ConsensusCogateway` contact.

- Update Coconsensus::setup
  - `Coconsensus::setup` will call `Utmost::setup` and `ConsensusCogateway::setup`

- Rename blockHeight, sourceBlockHeight and targetBlockHeight
  - Rename `blockHeight` to `blockNumber`
  - Rename `sourceBlockHeight` to `sourceBlockNumber`
  - Rename `targetBlockHeight` to `targetBlockNumber`
  - Ref: https://github.com/mosaicdao/mosaic-1/pull/221#discussion_r370245486

- Update test cases for Protocore::registerVoteInternal
  - Cleanup test cases

- Update `Consensus::NewMetachain`
  - initialize the metablock tip to zero
  - store an initial committed metablock with `Height=0` and `round=Committed`
  - create the assigned Core with `creationHeight=Height+1

- Update travis.yml
  - Add `npm run lint:sol:solium`
  - Update `.soliumignore` to handle required dependencies

- Update the Block struct in Coconsensus contract
  - The `Block` struct should have `previouslyFinalizedBlockNumber`

- Update the variables in GenesisSelfProtocore
  - like `genesisAuxiliarySourceBlockHash` to `genesisSelfSourceBlockHash`
  - and others

- Have a general setup check implemented in CoconsensuModule and Coconsensus for every external function
  - This will ensure that the functions are not being called until the setup is done.

- Update Utmost implementation:
    - Remove UtilityToken dependencies form Utmost
        - The `mint` function of `Utmost` needs to perform 2 operations
            - Call internal `_mint` function of ERC20 token.
            - Unwrap the token to get the base token.
        - In the current implementation, the `mint` external function is defined in the `UtilityToken`.
        - In order to unwrap the token, the `mint` function needs to be overridden, resulting in to bypass the calls to Utility token.
        - `ERC20Token` already has `_mint`, `_burn` and `_burnFrom` internal functions. Currently the `Utmost is UtilityToken`, remove the dependencies of UtilityToken and directly use the functions from ERC20 token.
        - The mint function in the utmost can only be called by `ConsensusCogateway` address.
        - The `mint external` function should not emit `Transfer` event as its already handled in the `_mint internal` function of ERC20Token..
        - The `mint`, `burn` and `burnFrom` functions must not return `boolean` value.

- Fix the unit test cases for `Committee::proposalAccepted`
    - Fix the skipped unit tests in `test/committee/proposal_accepted.js`.
    - Implement the missing unit tests.
- Fix the unit test cases for `ConsensusGatewayBase::hashKernelIntent`
    - Fix the skipped unit tests in `test/consensus-gateway/hash_kernel_intent.js`.
    - Implement the missing unit tests.
- Fix the unit test cases for `Consensus::commitMetablock`
    - Fix the skipped unit tests in `test/consensus/commit_metablock.js`.
    - Implement the missing unit tests.

- Fix the unit test cases for `Consensus::enterCommittee`
    - Fix the skipped unit tests in `test/consensus/enter_committee.js`.
    - Implement the missing unit tests.

- Fix the unit test cases for `Consensus::formCommittee`
    - Fix the skipped unit tests in `test/consensus/form_committee.js`.
    - Implement the missing unit tests.

- Fix the unit test cases for `Consensus::setup`
    - Fix the skipped unit tests in `test/consensus/setup.js`.
    - Implement the missing unit tests.

- Fix the unit test cases for `Consensus::precommitMetablock`
    - Fix the skipped unit tests in `test/consensus/precommit_metablock.js`.
    - Implement the missing unit tests.

- Fix the unit test cases for `Consensus::upsertValidator`
    - Fix the skipped unit tests in `test/self_protocore/upsert_validator.js`.
    - Implement the missing unit tests.

- Renaming the internal setup funtions
  - External setup functions will be named as `setup`
  - Internal setup functions can be named with the corresponding contract postfixed to setup
    - eg. in `Core` contract and other places; `setupConsensus(_consensus)` will be renamed to `setupConsensusModule` and `setupCoconsensusModule`

- Update `StateRootI` and `AnchorI` contracts


## Validator

### Validator models

**PM document:** [Validator models PM Document](https://github.com/mosaicdao/mosaic-pm/blob/master/specification-discussions/20200114-validator-models.md#modelsrepositories)

1. Implement Metachain model and MetachainRepository
    - The `Metachain` model will have following attributes:
      - metachainId - PK
      - origin chain id (it can be constant)
      - anchorGA
      - mosaic version
      - consensus global address

1. Implement Core model and CoreRepository
    - The `Core` model will have following attributes:
      - metachain id
      - core status
      - min validators
      - max validator
      - origin observation block height

1. Implement UntrustedEndpoint model and UntrustedEndpointRepository
    - The `UntrustedEndpoint` model will have following attributes:
      - endpoint - PK
      - validatorGA
      - serviceType
      - metachainId

1. Implement Genesis model and GenesisRepository
    - The `Genesis` model will have following attributes:
      - coreGA - PK
      - CID

1. Implement Committee model and CommitteeRepository
    - The `Committee` model will have following attributes:
      - committeeGA - PK
      - metablockHash
      - metachainId
      - seed
      - status
      - decision
      - size
      - quorum

1. Implement Validator model and ValidatorRepository
    - The `Validator` model will have following attributes:
      - metachain id
      - reputation
      - begin height
      - end height
      - status
      - withdrawalAddress
      - lockedEarnings
      - cashableEarnings
      - withdrawalBlockHeight

1. Implement Link model and LinkRepository
    - The `Link` model will have following attributes:
      - voteMessageHash - PK
      - parent
      - isLeaf
      - targetBlockNumber
      - targetBlockHash
      - targetFinalizationStatus
      - sourceTransitionHash
      - proposedMetablockHeight
      - metachainId
      - coreGA
      - relativeSelfDynasty

1. Implement Metablock model and MetablockRepository
    - The `Metablock` model will have following attributes:
      - metablockHash - PK
      - metachainId
      - metablockHeight
      - ~~status (need to figure this out, how is this different from round)~~
      - kernelHash
      - transitionHash
      - sourceBlockHash
      - targetBlockHash
      - sourceBlockNumber
      - targetBlockNumber
      - round
      - roundMetablockNumber

1. Implement Message model and MessageRepository
    - The `Message` model will have following attributes:
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

1. Implement SignedVoteMessage model and SignedVoteMessageRepository
    - The `SignedVoteMessage` model will have following attributes:
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

1. Implement Gateway model and GatewayRepository
    - The `Gateway` model will have following attributes:
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
      - Remote gateway last proven block number
      - AnchorGA

1. Implement DepositIntent model and DepositRepository
    - The `DepositIntent` model will have following attributes:
      - Deposit intent hash
      - message hash
      - deposit amount
      - beneficiary

1. Implement GenesisDepositIntent model and GenesisDepositIntent Repository
    - The `GenesisDepositIntent` model will have following attributes:
        - Deposit intent hash
        - message hash
        - deposit amount
        - beneficiary

1. Implement WithdrawIntent model and WithdrawIntentRepository
    - The `WithdrawIntent` model will have following attributes:
      - withdraw intent hash
      - message hash
      - amount
      - beneficiary

1. Implement OpenKernelIntent model and OpenKernelIntentRepository
    - The `OpenKernelIntent` model will have following attributes:
      - open kernel intent hash - PK
      - message hash
      - kernel height
      - kernel hash

1. Implement Anchor model and AnchorRepository
    - The `Anchor` model will have following attributes:
      - metachainid
      - state root
      - block height

1. Implement AnchoredStateRoot model and AnchoredStateRootRepository
    - The `AnchoredStateRoot` model will have following attributes:
      - StateRootProviderGA - PK
      - latestAnchoredBlockNumber

1. Implement Member model and MemberRepository
    - The `Member` model will have following attributes:
      - memberGA - PK
      - validatorGA
      - metablockHash
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

1. Implement Protocore model and ProtocoreRepository
    - The `Protocore` model will have following attributes:
      - protocoreGA - PK
      - openMetablockHeight
      - coreGA
      - metachainId

1. Implement Kernel model and KernelRepository
    - The `Kernel` model will have following attributes:
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

1. Implement TransitionObject model and TransitionObjectRepository
    - The `TransitionObject` model will have following attributes:
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

1. Implement Coconsensi model and CoconsensiRepository
    - The `Coconsensi` model will have following attributes:
      - CoreGA
      - metachainId

1. Implement TrustedEndpoint model and TrustedEndpointRepository
    - The `TrustedEndpoint` model will have following attributes:
      - endpoint - PK
      - metachainId
      - serviceType
        - enode
        - ipfs
        - GraphWs
        - GraphRpc
        - ChainRpc


### Validator: transaction handlers

1. Implement `Axiom::CreatedMetachain`.
    - Updates `Metachain` and `Gateway` models.
1. Implement `Consensus::CreatedCore`.
    - Updates `Core` model.
1. Implement `Consensus::UpdatedCoreLifetime`.
    - Updates `Core` model.
1. Implement `Core::UpdatedCoreStatus`.
    - Updates `Core` model.
1. Implement `Consensus::PublishedEndpoint`.
    - Updates `UntrustedEndpoint` model.


#### Self Link cycle

1. Implement `SelfProtocore::ProposedLink`.
    - Update event `LinkProposed` to `ProposedSelfLink`
    - Updates `Link` and `TransitionObject` models.
    - `relativeSelfDynasty` will be set to `-1` for now.

#### Origin Link cycle

1. Implement `OriginProtocore::ProposedLink`.
    - Update event `LinkProposed` to `ProposedOriginLink`
    - Updates `Link` model.
    - `relativeSelfDynasty` will be set to `-1` for now.

#### Link cycle

1. Implement `Protocore::VoteRegistered`.
    - Updates `SignedVoteMessage` model.
1. Implement `Protocore::JustifiedLink`.
    - Updates `Link` model.
1. Implement `Protocore::FinalizedLink`.
    - Updates `Link` model.

/*
 * Model "Block": (blockhash-PK, metachainId, blocknumber, < blockheader >, relativeSelfDynasty )
 * Coconsensus emit `Coconsensus::CheckpointFinalized(metachainId, blockHash, relativeSelfDynasty)`
 */


#### Kernel cycle

1. Implement `Core::OpenedKernel`.
    - Updates `Kernel` model.
1. Implement `ConsensusGateway::DeclaredOpenKernel`.
    - Updates `Kernel`, `Message` and `OpenKernelIntent` models.
1. Implement `ConsensusCogateway::OpenKernelConfirmed`.
    - Updates `Kernel` and `Message` models.
1. Implement `Coconsensus::KernelCoopened`.
    - Updates `Kernel` and `Protocore` models.

#### Metablock cycle

1. Implement `Core::MetablockProposed`.
    - Updates `Metablock` model.
1. Implement `Core::VoteRegistered`.
    - Updates `SignedVoteMessage` model.
1. Implement `Consensus::MetablockPrecommitted`.
    - Updates `Metablock` model.
1. Implement `Consensus::CommitteeFormed`.
    - Updates `Committee` and `Metablock` models.
1. Implement `Consensus::CommitteeDecided`.
    - Updates `Committee` and `Metablock` models.
1. Implement `Consensus::MetablockCommitted`.
    - Updates `Metablock` and `Kernel` models.

#### Committee lifecycle

1. Implement `Committee::StatusUpdated`.
    - Updates `Committee` model.

#### Committee member lifecycle

1. Implement `Committee::MemberEntered`.
    - Updates `Member` model.
1. Implement `Committee::MemberEjected`.
    - Updates `Member` model.
1. Implement `Committee::MemberCommitted`.
    - Updates `Member` model.
1. Implement `Committee::MemberRevealed`.
    - Updates `Member` model.

#### Validator lifecycle

1. Implement `Consensus::ValidatorJoined`.
    - Updates `Validator` model.
1. Implement `Consensus::ValidatorLoggedOut`.
    - Updates `Validator` model.
1. Implement `Reputation::ValidatorWithdrawn`.
    - Updates `Validator` model.

#### StateRootProvider

1. Implement `StateRootProvider::StateRootAvailable`.
    - Updates `AnchoredStateRoots` model.
    Q(Should this be `AnchoredStateRoot`?)
1. Implement `Coconsensus::ProtocoreSetup`.
    - Updates `Protocore` and `Link` model.

#### Gateway

1. Implement `ConsensusGateway::RemoteGatewayProven`.
    - Updates `Gateway` model.

#### Deposit lifecycle

1. Implement `ConsensusGateway::Deposited`.
    - Updates `DepositIntent` and `Message` models.
1. Implement `ConsensusCogateway::DepositConfirmed`.
    - Updates `DepositIntent` and `Message` models.

#### Withdrawal lifecycle

1. Implement `ConsensusCogateway::Withdrawn`.
    - Updates `WithdrawIntent` and `Message` models.
1. Implement `ConsensusGateway::WithdrawalConfirmed`.
    - Updates `WithdrawIntent` and `Message` models.

#### GenesisDeposit lifecycle

1. Implement `ConsensusGateway::GenesisDeposited`.
    - Updates `Genesis`, `DepositIntent` and `Message` models.
    (note: SourceStatus and TargetStatus are both instantaneously`Declared`)

## Validator services

**PM document:** [Validator PM Document](https://github.com/mosaicdao/mosaic-pm/blob/master/specification-discussions/20200114-validator-models.md#user-stories)

1. Implement Deposit Service
1. Implement JoinBeforeCreation Service
1. Implement JoinAfterCreation Service
1. Implement OriginProtocoreProposeLink Service
1. Implement SelfProtocoreProposeLink Service
1. Implement ProposeMetablock Service
1. Implement RegisterVote Service
1. Implement FormCommittee Service
1. Implement EnterCommittee Service
1. Implement AnchorStateRoot Service
1. Implement ChallengeCommittee Service
1. Implement ActivateCommittee Service
1. Implement SubmitSealedCommit Service
1. Implement RevealCommit Service
1. Implement Logout Service

## Genesis file generation

**PM document:** [Genesis file generation PM document](https://github.com/mosaicdao/mosaic-pm/blob/master/specification-discussions/20200108-genesis-file-generation.md)

1. Implement a function `getBytecode()` to get the source bytecode and ABI for all the contracts, from the IPFS.
1. Implement function `retrieveGenesisTemplate()` to retrieve template of genesis file (const in config)
1. Implement function `generateGenesis()` to generate the genesis file.
1. Implement write storage key-value pairs for GenesisSelfProtocore
1. Implement write storage key-value pairs for GenesisOriginProtocore
1. Implement write storage key-value pairs for GenesisUtmost (totalSupply)
1. Implement write storage key-value pairs for GenesisOriginObserver (GenesisOriginObservation)
1. Implement write storage key-value pairs for GenesisCoconsensus
1. Save the generated genesis file to the IPFS and store the CID to the `GenesisFiles` model.



## Validator other tasks(@abhay)

1. Explore Graph database - Gen1 needed?
1. Define mosaic-subgraph entities
1. Implement mosaic-subgraph
   <b>Question:</b> It's more than 3 pointer?
1. Implement Validator config (it's not manifest)
1. Populate seed data
1. Implement Subscription libraries
   <b>Question:</b> Move common libraries to shared space?
1. Implement starting of validator
1. Implement stopping of validator
1. Implement CalculateMembers method/utility class
1. Implement Library to interact with Kubernetes
   <b>Question:</b> Should this be a service?
