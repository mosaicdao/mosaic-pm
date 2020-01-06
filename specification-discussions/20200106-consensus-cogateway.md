---
title: 'Consensus Cogateway'
disqus: hackmd
---

Consensus cogateway
===
| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
|  0.14   |  06/01/2020   | Consensus cogateway |

**Meeting date/time:** N/A
**Editor:** Deepesh Kumar Nath
**Team:** N/A

---

## Overview
This is the pair of consensus gateway contract on the auxiliary chain
When a new message is declared on the consensus gateway for the origin chain, it has to be confirmed on the auxiliary chain. 
ConsensusCogateway is used to confirm such declared messages on the auxiliary chain.
This contract uses `MessageBus` for atomically transfer of cryptographically verifiable messages between two chains.

## Goals

The Goal for `ConsensusCogateway` is to achieve the following in the auxiliary chain.

- `ConsensusCogateway` should follow the proxy pattern for deployment
- Setup of contract. (For testing only)
- Use `ConsensusCogatewayGenesis` contract for storage.
- Confirm open kernel hash.
- Confirm deposit of MOST.
- Declare withdraw of MOST.
- Declare a new cogateway.
- Confirm gateway active.

## Assumptions
- N/A
## Out of scope
Currently, the following is not in scope. This will be done later
- Declare withdraw of MOST.
- Declare a new cogateway.
- Confirm gateway active.
## Open questions
- N/A
## Approach
- ### Contract architecture
    ```mermaid
        classDiagram

          ConsensusGatewayBase <|-- ConsensusGateway
          
          ConsensusGatewayBase <|-- ConsensusCogateway
          
          ERC20GatewayBase <|-- ConsensusGateway
          
          ERC20GatewayBase <|-- ConsensusCogateway
          
          MessageBus <|-- ConsensusGateway
          
          MessageBus <|-- ConsensusCogateway
          
          ConsensusCogatewayGenesis <|-- ConsensusCogateway
          
          CoConsensusModule <|-- ConsensusCogateway
          
          ConsensusModule <|-- ConsensusGateway
          
        class ConsensusGateway{
              - OUTBOX_OFFSET
              - INBOX_OFFSET
              - setup()
              - declareOpenKernel()
              - deposit()
          }
          
          class ConsensusCogateway{
              - OUTBOX_OFFSET
              - INBOX_OFFSET
              - setup()
              - proveConsensusGateway()
              - confirmOpenKernel()
              - confirmDeposit()
          }
          
          class MessageBus{
              - mapping(bytes32 => bool) public inbox
              - mapping(bytes32 => bool) public outbox
              - setupMessageOutbox()
              - setupMessageInbox()
              - declareMessage()
              - confirmMessage()
          }
          
          class ERC20GatewayBase{
              - ERC20I public token
              - hashDepositIntent()
              - hashWithdrawIntent()
          }
          
          class ConsensusGatewayBase{
            - ERC20I most;
            - mapping(address => uint256) nonces;
            - uint256 currentMetablockHeight;
            - setup(ERC20I _most,uint256 _currentMetablockHeight)
            - hashOpenKernelIntent()
          }
          
          class ConsensusCogatewayGenesis{
              - TBD
              - TBD()
          }
          
          class CoConsensusModule{
            - coConsensus
            - setupCoConsensus(CoConsensusI _coConsensus)
          }

          class ConsensusModule{
            - ConsensusI consensus
            - setupConsensus(ConsensusI _consensus)
          }
          
    ```
- ### Contracts
    - `ConsensusGatewayBase`
    - `ERC20GatewayBase`
    - `ConsensusModule`
    - `MessageBus`
    - `ConsensusCogatewayGenesis`
    - `ConsensusGateway`
    - `ConsensusCogateway`
    
- ### Contract flow    
    #### OUTBOX_OFFSET
    The contract should have a public constant variable that indicated the storage index of `MessageBus.outbox` mapping. This will help read the index from the contract while generating the proof data.
    
    Example:
    ```solidity
    uint8 constant public OUTBOX_OFFSET = uint8(index);    
    ```
    #### INBOX_OFFSET
    The contract should have a public constant variable that indicated the storage index of `MessageBus.inbox` mapping. This will help read the index from the contract while generating the proof data.
    
    Example:
    ```solidity
    uint8 constant public INBOX_OFFSET = uint8(index);
    ```
    #### Setup
    Consensus Cogateway follows the proxy pattern. The setup function will be called while deploying the proxy contract for `ConsensusCogateway`.
    This can be called only once.
    The following will be initialized in this function
    -  Metachain id
    -  CoConsensus contract address
    -  utMOST token address
    -  ConsensusGateway contract address
    -  Storage index of outbox mapping in ConsensusGateway contract.
    -  The maximum number of storage roots to be stored in a circular buffer.
    -  Initial metablock height.
    
    The proposed function signature looks like below:
    ```solidity
        function setup(
            bytes32 _metachainId,
            address _coConsensus,
            ERC20I _utMOST,
            address _consensusGateway,
            uint8 _outboxStorageIndex,
            uint256 _maxStorageRootItems,
            uint256 _metablockHeight
        )
            external
    ```
    **Please note: This will be needed for testing purposes only. The initial setup will already be done in the genesis block.**
    #### Prove ConsensusGateway contract
    This contract will have a function to prove the existence of ConsensusGateway contract address in the origin anchored state root (Merkel Patricia Proof). Once the proof is done this contract will store the storage root of the contract. This function should be called before confirming the open kernel and confirming of deposit message.
    
    Proposed function signature:
    ```solidity
    function proveConsensusGateway(
        uint256 _blockHeight,
        bytes calldata _rlpAccount,
        bytes calldata _rlpParentNodes
    )
        external
    ```
    #### Confirm Open Kernel
    This contract can confirm the declaration of the open kernel in the origin chain.
    This function should keep track of metablock heights. A message can only be confirmed when the kernel height is `+1` of the metablock height.
    A Merkel proof should be provided to confirm the declaration of an open kernel in the `ConsensusGateway.outbox` mapping.
    This contract should track the confirmed kernel hashes.
    
    Proposed function signature:
    ```solidity
    function confirmOpenKernel(
        uint256 _kernelHeight,
        bytes32 _kernelHash,
        uint256 _feeGasPrice,
        uint256 _feeGasLimit,
        address _sender,
        uint256 _blockHeight,
        bytes calldata _rlpParentNodes
    )
        external
        returns (bytes32 messageHash_)
    ```
    #### Confirm Deposit
    TBD
---
## Meeting notes
NA