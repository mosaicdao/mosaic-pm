# Mosaic Implementation Discussion: Consensus

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 10/12/2019   | Consensus contract |

Editor: Benjamin Bollen

## Contract architecture
![](https://i.imgur.com/pxra0Gn.jpg)

## Overview

`Consensus` on Ethereum mainnet is the pivotal contract that governs the set of metablockchain. It is a Byzantine Fault-Tolerant (BFT) consensus engine which precommits and commits metablocks in two rounds. A metablockchain cannot fork.

`Consensus` has three types of `ConsensusModule`s: `Core`, `Committee` and `Reputation`.

## Consensus rounds, a code highlight

```js
contract Consensus is MasterCopyUpgradable, ... {

    enum MetablockRound {
        Undefined,
        Precommitted,
        CommitteeFormed,
        CommitteeDecided,
        Committed
    }

    struct Metablock {
        bytes32 metablockHash;
        MetablockRound round;
        uint256 roundBlockNumber;
    }

    mapping(bytes32 /* metachain id */ =>
        mapping(uint256 /* metablock height */ => Metablock)) metablockchains;

    mapping(bytes32 /* metachain id */ => uint256 /* tip */) metablockTips;

    mapping(bytes32 /* metachain id */ => AnchorI) anchors;

    mapping(bytes32 /* metachain id */ => CoreI) assignments;

    mapping(bytes32 /* metablock hash */ => CommitteeI) committees;

    mapping(bytes32 /* metablock hash */ => bytes32 /* decision */) decisions;

    mapping(CoreI => CoreLifetime) CoreLifetimes;

    [...]

    function precommitMetablock(
        bytes32 _metachainId,
        bytes32 _metablockHeight,
        bytes32 _metablockHashPrecommit,
        address payable _reimbursementReceiver
    )
        external
        reimburseGas(GAS_PRECOMMIT, _reimbursementReceiver)
        onlyAssignedCore(_metachainId)
    {
        uint256 currentHeight = metablockTips[_metachainId];
        Metablock storage currentMetablock =
            metablockchains[_metachainId][currentHeight];

        require(
            currentHeight.add(1) == _metablockHeight,
            "A precommit must append to the metablockchain."
        );

        require(
            currentMetablock.round == MetablockRound.Committed,
            "Current metablock must be committed."
        );

        Metablock storage nextMetablock =
            metablockchains[_metachainId][_metablockHeight];

        assert(nextMetablock.round == MetablockRound.Undefined);

        nextMetablock.metablockHash = _metablockHashPrecommit;
        nextMetablock.round = MetablockRound.Precomitted;
        nextMetablock.roundBlockNumber = block.number;

        metablockTips[_metachainId] = _metablockHeight;

        // On first precommit by a core, CoreLifetime state will change to active.
        if (coreLifetimes[msg.sender] == CoreLifetime.genesis) {
            coreLifetimes[msg.sender] = CoreLifetime.active;
        }
    }

    function formCommittee(
        bytes32 _metachainId
        address payable _reimbursementReceiver
    )
        external
        reimburseGas(GAS_FORM_COMMIITEE, _reimbursementReceiver)
    {
        uint256 currentHeight = metablockTips[_metachainId];
        Metablock storage currentMetablock =
            metablockchains[_metachainId][currentHeight];

        require(
            currentMetablock.round == MetablockRound.Precommitted,
            "Assigned core must have precommitted a metablock to form a committee."
        );

        uint256 committeeFormationHeight = currentMetablock.roundBlockNumber.add(
                COMMITTEE_FORMATION_DELAY);

        require(
            block.number > committeeFormationHeight,
            "Committee formation height has not yet come to pass."
        );

        bytes32 seed = hashBlockSegments(
            currentMetablock.metablockHash,
            committeeFormationHeight);

        committees[currentMetablock.metablockHash] = newCommittee(
            _metachainId,
            committeeSize,
            seed,
            currentMetablock.metablockHash
        );

        currentMetablock.round = MetablockRound.CommitteeFormed;
        currentMetablock.roundBlockNumber = block.number;
    }

    function enterCommittee(
        bytes32 _metachainId,
        address _core,
        address _validator,
        address _furtherMember,
        address payable _reimbursementReceiver
    )
        external
        reimburseGas(GAS_ENTER_COMMITTEE, _reimbursementReceiver)
    {
        uint256 currentHeight = metablockTips[_metachainId];
        Metablock storage currentMetablock =
            metablockchains[_metachainId][currentHeight];

        require(
            currentMetablock.round == MetablockRound.CommitteeFormed,
            "Committee must have been formed to enter a validator."
        );

        // TODO: check validator at roundBlockNumber
    }
}
```

** **

# **ConsensusGateway (Kernel Gateway)**

If we can write all the state to the genesis file, then
- validators should get balance they have staked
- staking messages should be stored as declared

## `Consensus:CoreLifetime` and `Core:Status`

In Consensus the Core lifetime is tracked
```
    CoreLifetime:undefined
    CoreLifetime:halted
    CoreLifetime:corrupted
    CoreLifetime:creation
    CoreLifetime:genesis
    CoreLifetime:active
```
In Core the Core status is tracked
```
    CoreStatus:undefined
    CoreStatus:created
    CoreStatus:opened
    CoreStatus:precommitted
    CoreStatus:terminated
```

1. TechGov calls `axiom:newMetachain()`, no params needed. `metachainId = [12bytes type, 20 bytes address of anchor proxy]`

```
CoreLifetime:Undefined for metachainId(<Anchor>)
```
Axiom deploys proxy for Core, and calls `setupCore(parentHash, ...)`, where `bytes32 parentHash = [12bytes type, 20bytes address of core proxy]` -- orchestrated by Consensus L628:
```
        address core = newCore(
            metachainId,
            EPOCH_LENGTH,
            uint256(0), // metablock height
            parentHash, // parent hash
            gasTargetDelta, // gas target
            uint256(0), // dynasty
            uint256(0), // accumulated gas
            _rootBlockHeight
        );
```
Note that the `_rootBlockHash` can be removed after `Core:setupCore(...)` no longer has `bytes32 _source`, and in Core.sol `committedSource` can be removed.
```
Core's status is set from Status:Undefined to Status:Created in `setupCore`
```
```
Consensus' CoreLifetime is set to `CoreLifetime:Creation`
```

Note: proxy for ConsensusGateway is deployed during `newMetaChain` by Consensus

2. Validators can call `consensus:joinDuringCreation(core)`, core must be in `CoreLifetime:Creation`. When `minValidators` have joined, the Core status moves to
```
    CoreStatus:Opened
```
and the `Kernel` can be declared as open in the `ConsensusGateway` by calling from Consensus
```
    ConsensusGateway.declareKernelOpened(core)
        [msg.sender = Consensus]
        (height, kernelHash) = core.openKernel()
        DeclareMessage(OpenKernel(height, kernelHash))

    note: metachainId should be included in the domainSeparator
```
In consensus CoreLifetime ~~remains `Creation`~~ is set to `Genesis` on declaring the Kernel in ConsensusGateway. [ping @sarvesh]

When the Core precommits a first metablock, the `Genesis` lifetime is moved to `Active`

All validators joining during Creation must stake a set amount of `mOST` in ConsensusGateway.

Validators should stake the `mOST` in ConsensusGateway before calling the `joinDuringCreation`.

When `minValidator` joins, Consensus declares the Kernel and Core stores the current Ethereum block height, for `rootOriginObservationBlockHeight`.

All validators must generate the genesis file (offchain), the timestamp is taken to be the timestamp of the origin block at `rootOriginObservationBlockHeight`.

The Genesis file stores the following:
    <To be updated by Deepesh>

Once the validators generate the genesis file, they can start the node and publish the enode enpoint.

Validators can call `Consensus::publishEndpoint(string _service, string _endpoint)`. This value is emitted in an Event `PublishEndpoint(validator, core, service, endpoint)`.

Validators can read the logs to get the peers and start syncing.

mOST and UTmOST

Validators should be written in GenesisFile as Clique signers, in the first implementation before we have a L2 contract which sets the Layer-1 Mosaic blockproposers (pending Casper Fork CHoice rule). Validators can use the normal clique voting in blockheader to align validator-set with Kernel opening (in good faith).

____

## Tasks:
1. Axiom contract: Make `maxStateRoots` as constant like `EPOCH_LENGTH` in Consensus.
2. `Axiom::newMetachain()` no params in the function.
3. In `Axiom::newMetachain()` create parent hash. Create it as `[typebytes, core address]`.
(Example of Typebytes as per EIP191. `0x19 <1 byte version> <version specific data> <data to sign>.` 0x19 0x4d(M))
4. Compare [CID](https://github.com/multiformats/cid) and [EIP191](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-191.md) to use it for `typebytes`
5. Remove `committedSource` from storage for `Core.sol`.
// 6. Remove `assertPrecommit` from `Core.sol`.
7. Remove `assertPrecommit` call from `Core::openMetablock` in `Core.sol`. (Pro)
8. Deploy `ConsensusGateway` in `Consesus::newMetachain`
9. Validators Stake amount while joining the Core.
10. Reputation.isActive() issue
11. Reputation.publishEndpoint()
12. ConsensusGateway.declaredKernelOpened()
13. ConsensusGateway cleanup
14. mOST implemention.
15. UTmOST implementation.
16. Consensus.CoreLifetime
17. Consensus.RegisterCommitteeDecision. (Pro)
18. Committee.SealedCommittee. (Sarvesh)
19. Core.genesisRootObservaationHeight.
20. Generate Genesis file.(Deepesh)
21. init geth with geneisis, know your bootnodes from reputation endpoints.
21. Anchor proxy pattern.
22. Fork Facilitator and hollow out.

## Tickets:

1. Remove the constant `EPOCH_LENGTH` from `Axiom` contract. Add it in `Consensus` contract. Add a constant `MAX_STATE_ROOTS` in `Consensus` contract.
2. Change chainId to metachainId.
    - logic for metachainId creation is
    `metachainId = hash(0x19 0x4d mosaicDomainSeparator metachainIdTypehash)`
    - 0x4d = "M" for Mosaic
    - mosaicDomainSeparator = "MosaicDomain(string name,string version,uint256 originChainId,address consensus)"<name><version><originChainId><consensus>
    - typeHash "MetachainId(address anchor)"<anchor>
3. Convert Anchor contract to proxy pattern. (Sarvesh)
4. Update the `Axiom` constructor to include Anchor master copy.
    - Create a new function `deployMetachainProxies` in Axiom that can do the following.
        - Deploy anchor proxy contract.
        - Deploy core proxy contract.
        - Deploy consensus gateway proxy contract.
5. Refactor Axiom::newMetachain.
    - Remove all params from `axiom:newMetachain()`
    - Remove all the logic related to `maxStateRoot` and `rootRlpBlockHeader`.
    - Call `consensus.newMetaChain`.
    - Get the `metachainId` as the return value of `consensus.newMetaChain`
    - Emit an event `MetachainCreated`
7. Introduce core lifetime
    - Add `CoreLifetime` mapping in `Consensus` contract
    - Define `CoreLifetime` enum with the following states
        - CoreLifetime:Undefined
        - CoreLifetime:Halted
        - CoreLifetime:Corrupted
        - CoreLifetime:Creation
        - CoreLifetime:Genesis
        - CoreLifetime:Active
9. Refactor Consensus::newMetachain
    - Add a new mapping `consensusGateways` (metachainId bytes32 -> consensusGateway address)
    - Call `Axiom::deployMetachainProxies`
    - Generate `metachainId`.
    - Update the `CoreLifetime` status to `Creation`
    - Update `assignments` mapping.
    - Update `consensusGateways` mapping
    - return `metachainId`.
10. Refactor the `CoreStatus` enum.
    - The new statuses are
        - CoreStatus:Undefined
        - CoreStatus:Terminated
        - CoreStatus:Created
        - CoreStatus:Opened
        - CoreStatus:Precommitted
    - Consensus contract should not inherit `CoreStatus` anymore.
12. Refactor Consensus::newCore
    - Update the function `newCore` in consensus contract.
    - Remove the `uint256 _source` param.
    - Update `CORE_SETUP_CALLPREFIX` to exclude `_source`.
    - In `Core::setup` remove `bytes32 _source` param.
    - Remove `committedSource` variable from Core contract.
    - Change the `CoreStatus` to `Created`.

13. Refactor Consensus::JoinDuringCreation.
    - Update the params of `Consensus::joinDuringCreation`.
        - `joinDuringCreation(bytes32 _metachainId, address _withdrawalAddress)`.
    - Add a require/modifier that the value of `CoreLifetime` should be `Creation`.
    - ~~Add a require to verify that the validator has declared a stake message in `ConsensusGateway`~~.
    - Remove `validateJoinParams` function.
    - Remove `isCore` require.
    - Call `reputation.register`~~join~~.
    - Call `core.joinDuringCreation(msg.sender)`.
    - if the `core.joinDuringCreation(msg.sender)` returns true, then call `ConsensusGateway.declareKernelOpened(core)`.

14. Refactor Core::joinDuringCreation
    - Add `rootOriginObservationBlockHeight` variable.
    - Add a boolean return type, the value is true when the coreStatus is opened.
    - Initialise `openKernelHeight` in `Core::setup`
    - Remove the `openKernelHeight = creationKernelHeight;` from `Core::joinDuringCreation`
    - When the CoreStatus is Opened, then update `rootOriginObservationBlockHeight` value with current block number.

16. Validators should publish the end points in consensus contract

    - Add a function `publishEndpoint(string _service, string _endpoint)`.
    - Validator should be actice and **! hasbeenSlashed**. (not slashed.)
    - This can be called only by validators.
    - Emit an event with validator, core, service string, endpoint string.
16. List down all the contracts and its bytecode from auxiliary chain.
15. Genesis creation (EPIC).
    A tool should read the core contract and generate a genesis file. This tool can use a genesis file template and update the following values.
     - metachainId
     - timestamp
     - extraData
     - initial balance for validators
     - precompiled code.
   The address of the precompiled contracts can be determined
   `0x0000000000000000000000000000000000000000000000000000000000004D<00-ff>`
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
      "code": "0xcodeofUTmOST",
      "nonce": "1",
      "storage": {
        "0x0000000000000000000000000000000000000000000000000000000000000000":" < some value>",
        "0x0000000000000000000000000000000000000000000000000000000000000001":" < some value >"
      }
    },
    "< validator addresss 1 >": {
      "balance": "< Staked amount address >"
    },
    "< validator addresss 2 >": {
      "balance": "< Staked amount address >"
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

17. mOST token/interface (burn/mint).
    - task for D+B
18. UTmOST token.
19. ConsensusGateway.
    1. declareKernelOpened(core)
    <More to be added>
20. ConsensusCoGateway.
20. MessageInbox.
21. MessageOutbox.
22. Readability improvement for consensus. Split the contract in to few logical parts and include it in Consensus contract
23. MetahashChain. (Pro)
    https://github.com/mosaicdao/mosaic-1/issues/92

    In consensus contract,
    - Add a new mapping metahashChain(uint256 metablockNumber -> MetaBlockHeader)
    ```
        struct MetablockConsensus {
            bytes32 metablockHash;
            uint256 roundBlockNumber;
            MetablockRound round;
        }

        enum MetablockRound {
            Undefined,
            Precommitted,
            CommitteeFormed,
            CommitteeDecided,
            Committed,
        }
    ```
    - Update the mapping when `precommitMetablock`
    `registerCommitteeDecision` and `commitMetablock`.
    - Keep only latest `n` number of recent blocks, prune the older data.


## Iteration 3:

Here the proposal is for the following.
- Run the transaction(s) in zero gas.
- All validators should get the balance after proving the stake message.
- All validators should run the node
- Publish the enode so that other validators can join.

### Core states:
Following are the core state. A new state `KernelCreated` is proposed here.
1. Undefined.
2. **Creation.**
    - Validators can join when core is at this state.
    - While joining, validators stake amount on the origin chain contract so that they can get the base token in the aux chain. (Message Declaration).
    - Core can move to next state only after the `minValidators` is reached. Wait for `'n'` blocks before moving to next state. This is because we can include the origin block header. So that declared messages can be proved in the aux chain.
    - After `'n'` blocks from the quorum is reached.
        - Anyone can call the `Core::createKernel`.
        - Provide origin block header. And the proof data of message declaration of validators stake. It should be in latest `256` blocks so that the hash can be verified in the contract.
        - This block header and proof data is stored in the Genesis contract.
        - Provide signed transaction for calling `CoAxiom::initialize`. This is because, if all the validators starts the auxiliary node seperatly and calls initialize, all the nodes should create same state root. Otherwise its all the forks and all validators will do seperate observations.
        - Core status is changed to `KernelCreated`.
4. **KernelCreated.**
    - Start the auxiliary chain with zero gas price.
    - Execute the `CoAxiom::initialize` transaction, (and any other transaction(s))
    - Stop the chain, start the chain with gas price.
    - Wait for `n` blocks. `n` is predetermined and fixed.
    - Provide the stateroot of `n`th block along with `enode://` in core contract.
    - When the 2/3rd quorum is reached for the stateroot. Open the kernel. `Core::Open`
    - Commit this stateRoot and block height in anchor.
    - Core status is changed to `Opened`.
6. **Opened.**
    - Now the validators can start the nodes by connecting to the boot nodes.
8. Precommitted.
9. Halted.

---
# Iteration 2:

### Synopsis of the proposal:

Move the following things from the origin chain to auxiliary chain
- Kernel
- ConsensusGateway address
- ERC20Gateway address
- Origin chain block height and block hash. Validators will starts Casper FFG from this block height.

Fund the validators (Stake the equivalent amount in the origin chain)

Once the above information is available in the auxiliary chain
- Observe the origin chain (Casper FFG)
- Observe the auxiliary chain (Casper FFG)
- Propose a new meta-block on the origin chain.

### POC result:

1. Following data from the origin chain was successfully passed to the auxiliary chain through the genesis file.

```
    bytes20 _chainId,
    address _consensusGateway,
    address _techGov,
    address[] calldata _updatedValidators,
    uint256[] calldata _updatedReputation,
    uint256 _gasTarget,
    bytes calldata _rootOriginBlockHash
```
2. Contract was able to read all the above data.
3. Contract code was added at the predetermined address in genesis file.
4. Only one transaction call was made to a contract that was available at the predefined address.
5. It was able to read the origin chain data from the genesis file and deployed new contracts in axiliary chain.
6. ConsensusGateway address was successfully set in the ConsensesCoGateway contract (predefined at a location in the genesis file).


### Creating a new Metachain:
New meta-chain can be created by calling `Axiom::newMetachain`.
Params:
- God facilitator address
- Sealer address*
- The initial funding amount for validators
- The initial funding amount for the God facilitator.
*is sealer address still mandatory if auxiliary runs hybrid PoW + FFG

Following things happen in this call:
- A new Anchor (proxy) contract is deployed.
- A new Core (proxy) is deployed.
- A new ConsensusGateway (proxy) is deployed.
- A new ERC20Gateway (proxy) is deployed.
- A new Genesis contract (proxy) is deployed. All the initial information like god facilitator address, sealer address, validator fund amount, god facilitator amount, ERC20Gateway address,  ConsensusGateway address, etc are set in the Genesis contract.
- All the necessary assignments related to core, anchor, genesis, etc and bookkeeping are done in the consensus contract.

**Please note**: The Genesis contract is not yet for genesis file creation, there will be some modifier that will now allow any calls for genesis file creation yet.

Validators can join the core (during creation).
Once the Quorum is reached following will happen:

- Open a new kernel. This will include the initial validators and the reputations.
- Update/Finalize the Genesis contract with the following
     - Kernel data: validators[], reputation[], gasTarget. Parent hash will be 0x0 so not needed explicitly.
    - Current block height.
    - Current block hash.
- This will data that by ABI.encode of the necessary params required for the aux chain.
- Once the genesis is finalized, now genesis file can be created for the auxiliary chain.

**Questions:**
- Should the OST be staked while calling `newMetaChain` so that only that much amount can be funded to the validators and the god facilitator?
- ERC20 token address as input params for newMetachain? or OST will be the default base token for ERC20Gateway.
- Total max supply of the base token in the aux chain.

### Genesis file creation:
Genesis file will be created by reading the data from the Genesis contract. All the information needed for genesis file creation should be available in the contract so that the same file is created anytime.

Things required for the creation of Genesis file:
- Chain id (number).
- Sealer address (extraData).
- Timestamp (this can be generated from the code).
- Validator addresses.
- Facilitator address
- An amount that the validators need to be funded.
- Data from the origin-chain. (Open kernel, block height, block hash for the given height,  ConsensusGateway address, ERC20Gateway address )
- CoAxiom contract code.
- ConsensusCoGateway code (master copy).
- ConsensusCoGateway proxy code.
- ERC20CoGateway code (master copy).
- ERC20CoGateway proxy code.
- OSTPrime contract code.
- Other contracts master copy.

**Questions:**
- Should `chainId` be the same as `metaChainId`?
- What should be the initial balance for the validators?
- What should be the initial balance for the sealer address?

### Starting auxiliary chain
After the creation of the Genesis file, the auxiliary chain can be started.
The validators, the sealer, god facilitator, etc now have the gas for doing the transactions.

God facilitator can call `Axiom::initialize()` to initialize the contracts.

This will do the following:

- Deploy a new Genesis proxy contract, so that it can read the origin chain data, and provide the needed information to the other contracts.
- Initialize the ConsensusCoGateway with ConsensesGateway address.
- Initialize the ERC20CoGateway with the ERC20Gateway address.
- Setup the CoConsensus, this will include the deployment of the proxy contracts (the same way we do on the origin chain).
- Initialize the ProtoCore with initial Kernel data.

Now the validators can start the Casper FFG.
Validators will start observing the origin chain from the block number and block hash that was provided in the genesis.


##
Below is the Previous discussion

# Iteration 1

This document explains an option on how to create a new metachain.

**Assumption:**
The auxiliary chains will run on the latest Istanbul fork. So that we can take advantage of `chainid` opcode (explained later in the section).

**Mosaic contracts:**
The following are the mosaic contracts and external addresses in the mosaic.

**Origin chain:**
1. Axiom (contract)
2. Consensus (contract)
3. Reputation (contract)
4. Core (contract)
5. Anchor (contract)
6. ConsensusGateway/KernelGateway (contract)
7. TechGov (address)

**Auxiliary chain:**
In auxiliary chain the contracts are created in `Geneis block`.
1. CoConsensusGateway/CoKernelGateway (MasterCopy contract) at `0x0000000000000000000000000000000000001111)`
2. CoConsensus (MasterCopy contract) at `0x0000000000000000000000000000000000001112`
3. Anchor (MasterCopy contract) at `0x0000000000000000000000000000000000001113`
4. Protocore(MasterCopy contract) at `0x0000000000000000000000000000000000001114`
5. MetaChainData (Metachain genesis data) at `0x0000000000000000000000000000000000001115`
   **Please note:** this is not an actual contract, but it holds the bytes, which will be used by other contract as an input (explained in later section).
7. CoAxiom Contract at `0x0000000000000000000000000000000000001116`
8. CoConsensusGateway/KernelGateway contract at `0x0000000000000000000000000000000000001117`
9. Base token address `0x0000000000000000000000000000000000000000`


**Creating a new metachain:**
A new metachain can be created by `TechGov` by calling `Axiom::newMetaChain`.
Provide the following params `_maxStateRoots`, `_rootRlpBlockHeader` and `_godFacilitator`.
- `_maxStateRoots` and `_rootRlpBlockHeader` are used for deploying a new anchor contract and core contract.
- `_godFacilitator` this address will be funded in the auxiliary chain so that it can make transactions and do the setup and initializations.

The following things happen in this newMetachain call.
- Deploys Anchor contract (0xanchor).
- Deploys the New Core contract (0xcore).
- Deploys ConsensusGateway/Kernel Gateway (0xconsensusGateway).
        Note that the `coConsensusGateway` address needed so that messagebus can be configured. This address is already known and is at `0x0000000000000000000000000000000000001117`

There will be few things but not including it in this document.

- ABI encode all the following params
        - anchor address/chainid
        - core address
        - consensus address
        - reputation address
        - ConsensusGateway (Kernel Gateway) address
        - TechGov address
        - God facilitator address.
        - _rootRlpBlockHeader
        ...
        ...
        ...
    This will look something like this
    `0x00000000000000000000000000000000000000ffbe160df334bb721e432790696ca8c8748c2522e300000000000000000000000000000000000000ee......`

- Store this bytes in Consensus contract mapping.
    `mapping(bytes32 /*chain_id*/ => bytes ) public genesisMetaChainInfo`
- Declare a new message in consensusGateway with `newMetaChainIntent`. The hash of this `Consensus::genesisMetaChainInfo[chain_id]` is used, so that all the data in the address can be proved.

**Please note:** Till now the auxiliary chain does not exist. We will create a new auxiliary chain by creating the genesis file.

Now wait for few confirmation blocks and will create a genesis file.
This can be done from a tool in mosaic-chains, and provide the consensus and chainId as params

- Take `chainid` / `anchor address(0xanchor)` from the consensus contract, this will be the chain id in genesis file.
- Read the bytes from  `Consensus::genesisMetaChainInfo[chain_id]` and place the bytes as a code in following address `0x0000000000000000000000000000000000001116`
we call this as MetaChainData (Metachain genesis data).
Contracts in the auxiliary chain will treat these bytes as input while setup and initializations.

The genesis file will have the code for the predetermined address and the alloc section will appear like below.
```
"alloc": {
    "0x0000000000000000000000000000000000000000": {
      "balance": "0x200000000000000000000000000000000000000000000000000000000000000"
    },
    "0x0000000000000000000000000000000000001111": {
      "balance": "0x0",
      "code": "0x608MasterCopyOf_CoConsensusGateway"
    },
    "0x0000000000000000000000000000000000001112": {
      "balance": "0x0",
      "code": "0x608MasterCopyOf_CoConsensusMaster"
    },
    "0x0000000000000000000000000000000000001113": {
      "balance": "0x0",
      "code": "0x608MasterCopyOf_Anchor[TBD]"
    },
   "0x0000000000000000000000000000000000001114": {
      "balance": "0x0",
      "code": "0x608MasterCopyOf_Protocore"
    },
     "0x0000000000000000000000000000000000001115": {
      "balance": "0x0",
      "code": "0xMetaChainData<Copied from the consensusContract>"
    },
    "0x0000000000000000000000000000000000001116": {
      "balance": "0x0",
      "code": "0xCoConsensus"
    },
    "0x0000000000000000000000000000000000001117": {
      "balance": "0x0",
      "code": "0xCoAxiom"
    },
    "0xGOD_FACILITATOR": {
      "balance": "0x1", // Actual value to be determined.
    },
  },
```

Now with the new geneisis file, the auxiliary chain can be started.

The contracts will be deployed in the genesis block.

**Setup auxiliary chain:**

GodFacilitator address now has the balance from genesis block it can be used for setup and initialization

Here we need to do the following:
1. Initialize by calling `CoAxiom.initMetaChain(consensensGatewayAccountProof)

    - `CoConsensusGateway/CoKernelGateway` can be initialized by providing the `ConsensusGateway/KernelGateway` address. The required address can be read from the address `0x0000000000000000000000000000000000001115`

    - Now the `ConsensusGateway/KernelGateway` address can be proved by using `consensensGatewayAccountProof` data. The stateRoot needed can be retrived form `0x0000000000000000000000000000000000001115`.
    - Store the storage proof.
    - **get the chainid using the opcode and check if its equal to the anchor address.**

2. Activate the auxiliary chain by providing the storage proof for the declared message in the `ConsensusGateway/KernelGateway` address.

3. The EIPTokenGateway/MessageBus can also be set up in these steps by providing required data in the `MetaChainData`. As all the data in the `0x0000000000000000000000000000000000001115` is proved in the auxiliary chain, it will be simple to set up the EIP20Gateway for the basetoken.

The GodFacilitator can be used to provide the basetoken to facilitators.

Once the auxiliary chain is activated and facilitators are up the validators not can mint the token in order to participate.




All the following flow can be done in Mosaic-chains.
