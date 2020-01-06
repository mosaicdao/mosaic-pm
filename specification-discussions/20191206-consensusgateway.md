---
title: 'Consensus gateway'
disqus: https://hackmd.io/TlnWMDt-R962fejKEYyvzQ
---

Consensus gateway
===

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 14/12/2019    | Consensus gateway |

**Meeting date/time:** N/A

**Editor:** Deepesh Kumar Nath

**Team:** N/A

---

## Overview

The Consensus gateway establishes a channel for protocol messages to be communicated from the origin chain to a metachain/core, and back.

One specific message occurs when a new metablock is committed. Then a new kernel is opened in the `Core`. The opening of the new kernel has to be declared on the origin chain and it has to be confirmed in the auxiliary side.
To accomplish this,  `ConsensusGateway` and `ConsensusCogateway`.
These contracts use `MessageBus` for atomically transfer of cryptographically verifiable messages between two chains.

## Goals
The Goal for ConsensusGateway is to achieve the following in the origin chain.
- ConsensusGateway should follow the proxy pattern for deployment
- Setup of contract.
- Declare open kernel hash.
- Declare deposit of MOST.
- Confirm withdraw of MOST.
- Confirm a new cogateway.
- Declare gateway active.

## Assumptions
NA
## Out of scope
Currently, the following is not in scope. This will be done later.
- Confirm withdraw of MOST.
- Confirm a new cogateway.
- Declare gateway active.

## Open questions
NA
## Approach
- Describe solution here.

    ### Contract architecture



    ```mermaid
        classDiagram

          ConsensusGatewayBase <|-- ConsensusGateway
          ConsensusGatewayBase <|-- ConsensusCogateway
          ERC20GatewayBase <|-- ConsensusGateway
          ERC20GatewayBase <|-- ConsensusCogateway
          MessageBus <|-- ConsensusGateway
          MessageBus <|-- ConsensusCogateway
          ConsensusCogatewayGenesis <|-- ConsensusCogateway

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
              - uint256 currentMetablockHeight;
              - mapping(address => uint256) nonces
              - nonces
              - hashOpenKernelIntent()
          }
          class ConsensusCogatewayGenesis{
              - TBD
              - TBD()
          }
    ```
    ### Contracts
    1. #### ConsensusGatewayBase
        - This is the base contract that will be included by `ConsensusGateway` and `ConsensusCoGateway`
        - This contract contains the common variables and functions that will be used in `ConsensusGateway` and `ConsensusCogateway` contracts.
    2. #### ConsensusCogatewayGenesis
        - The detailed [Spec](https://) are here. [Update the link]
    4. #### ConsensusGateway
        This is a gateway to transfer messages from the origin chain to the auxiliary chain. Messages are transferred by declaring them in the `ConsensusGateway` in the origin chain and then confirming the declared message on `ConsensusCogateway` by providing the Merkle proof.
       This uses the `MessageBus` for message transfer. The type of message intent that can be passed is `open kernel intent` and `deposit intent`.

       Following are the functionalities that `ConsensusGateway` performs:
       1. Open kernel.
       2. Deposit MOST.
       3. Withdrawal of MOST.
       4. Confirm a new cogateway.
       5. Declare Gateway Active.
    4. #### ConsensusCogateway
        - The detailed [Spec](https://) are here. [Update the link]
    5. #### ERC20GatewayBase
        - The detailed [Spec](https://) are here.[Update the link]
    6. #### MessageBus
        - The detailed [Spec](https://) are here.[Update the link]

    ### Proxy pattern
    `ConsensusGateway` should follow the proxy pattern for the deployment. Whenever a new metachain is created, the proxy contract for `ConsensusGatway` will be deployed. This will save the amount of gas required for deployment.
    ### Setup
    Setup can be called only once. Setup will also set up the following contracts that are included by ConsensusGateway.
    - ConsensusGatewayBase
    - MessageOutbox
    - MessageInbox
    ```mermaid
    sequenceDiagram
        Axiom->>Axiom: createProxy(ConsensusGatewayMasterCopy, setupData)

        Axiom->>ConsensusGatewayProxy: new proxy(ConsensusGatewayMasterCopy)
        ConsensusGatewayProxy-->ConsensusGateway: masterCopy

        Note right of ConsensusGateway: This is the master<br/>copy of consensus<br/>gateway contract.

        ConsensusGatewayProxy-->>ConsensusGateway: setup(metachainId, consensus, consensusCogateway, outboxStorageIndex, maxStorageRootItems)

        ConsensusGateway->>ConsensusGateway:ConsensusGatewayBase.setupConsensusGatewayBase(_consensus)

        ConsensusGateway->>ConsensusGateway: MessageOutbox.setupMessageOutbox(metachainId, consensusCogateway, address(this))

        ConsensusGateway->>ConsensusGateway:MessageInbox.setupMessageInbox(metachainId, consensusCogateway, outboxStorageIndex, maxStorageRootItems,address(this))

        ConsensusGatewayProxy-->>Axiom: consensusGatewayProxy address
    ```

    Following is the proposal for setup function:
    ```
    function setup(
        bytes32 _metachainId,
        address _consensus,
        address _consensusCogateway,
        uint8 _outboxStorageIndex,
        uint256 _maxStorageRootItems
    )
        external
    ```
    - Params
        - `metachainId`
          MetachainId will be used by the `MessageBus`. Check [MessageBus](https://) for details [TODO: update the link]
        - `consensus`
          The `Consensus` contract address is required to set the consensus address in `ConsensusModule`. Few functions like `declareOpenKernel` can only be called by consensus contract.
        - `consensusCogateway`
          This address is needed by the `MessageBus`
        - `outboxStorageIndex`
          This will be needed by `MessageBus` while confirming the messages
        - `maxStorageRootItems`
          This is the number of storage roots to be stored. The message bus implements a circular buffer for storing the storage roots.
    ### Declare open kernel
    ConsensusGateway is used to declare the opening of the kernel. This can be called only by the consensus contract. It should create an openKernelIntentHash, and this intentHash can be used to declare a message in MessageBus.

    Things to consider:
    - Keep track of open kernel height while declaring the open kernel message.
    - Allow declaring new open kernels for a height that is one more than the height of the latest declared open kernel.
    - Declaration of an open kernel with the same height should also be allowed. This is required when a chain halts, then the new core is created and a new open kernel is declared at the same height.
    ```mermaid
    sequenceDiagram

    Consensus->>ConsensusGateway: declareOpenKernel(core, feeGasPrice, feeGasLimit)

    ConsensusGateway->>Core: getKernelHash()

    Note right of Core: Read the latest<br/> openKernelHash &<br/> openKernelHeight

    Core-->>ConsensusGateway: (kernelHeight, kernelHash)

    Note right of ConsensusGateway: 1. Check if <br/> currentKernelHeight<br/>- openKernelHeight<br/><=1

    Note right of ConsensusGateway: 2.Generate<br/>openKernelHash.

    Note right of ConsensusGateway: 3. Get current<br/>nonce for depositor.<br/>Increment the<br/>nonce.

    ConsensusGateway->>ConsensusGateway:MessageOutbox.declareMessage(openKernelHash, nonce, feeGasPrice, feeGasLimit, msg.sender)

    Note right of ConsensusGateway: 4.return<br/>messageHash

    ConsensusGateway-->>Consensus: messageHash
    ```
    The proposed `declareOpenKernel` function is given below.
    ```solidity
    function declareOpenKernel(CoreI _core, uint256 _feeGasPrice, uint256 _feeGasLimit)
        onlyConsensus
        external
        returns (bytes32 messageHash_)
    ```
    - params:
        - `core`
          Core contract address. The latest `openKernelHeight` and `openKernelHash` will be retrieved form this contract address.
        - `feeGasPrice` and `feeGasLimit`
          This is used for declaring messages in `MessageBus`. The value can be changed by the `techGov`, so it is being passed as a parameter here.

    ### Deposit MOST
    To perform any transaction on the auxiliary chain, the validator will require `utMOST` on the auxiliary chain.`utMOST` is the base token and used as a gas in auxiliary chain. Validators can get the `utMOST'` on the auxiliary chain by depositing the equivalent amount of `MOST` on the origin chain.

    A validator has to approve the deposit amount of tokens before calling `deposit`. A validator needs to provide the `beneficiary` address, `feeGasPrice` and `feeGasLimit` while calling the deposit function. The `utMOST` tokens will be minted to the `beneficiary` address on the auxiliary chain.

    The amount of `utMOST` minted in the `beneficiary` address will be determined with provided values of `feeGasPrice` and `feeGasLimit`.
    The address that confirms the message on the auxiliary chain will be rewarded. The reward calculation is as follows

    `reward = MIN(gasUsed*feeGasPrice, feeGasLimit*feeGasPrice)`
    `gasUsed` is the total gas used while confirming the message on the auxiliary chain.

    The amount of `utMOST` minted to `beneficiary` address is `(amount - reward)`.

    There should be an assertion to check if the reward value is possible with given `amount`, `feeGasPrice` and `feeGasLimit`.

    ```mermaid
    sequenceDiagram

    Validator->>MOST: approve(ConsensusGateway, amount)

    Validator->>ConsensusGateway: deposit(amount,beneficiary,feeGasPrice,feeGasLimit)

    Note right of ConsensusGateway: 1. Validate input<br/> params

    Note right of ConsensusGateway: 2. Check if<br/>reward is possible<br/>with given inputs

    Note right of ConsensusGateway: 3. Generate deposit<br/>intent hash

    Note right of ConsensusGateway: 4. Get current<br/>nonce for depositor.<br/>Increament the<br/>nonce.

    ConsensusGateway->>ConsensusGateway:MessageOutbox.declareMessage(depositIntentHash, nonce, feeGasPrice, feeGasLimit, msg.sender)

    ConsensusGateway->>MOST: transferFrom(msg.sender, address(this), amount)

    Note right of ConsensusGateway: 5. return<br/>messageHash


    ConsensusGateway-->>Validator: messageHash
    ```
    A proposed `deposit` function is given below

    ```
    function deposit(
        uint256 _amount,
        address _beneficiary,
        uint256 _feeGasPrice,
        uint256 _feeGasLimit,
    )
        external
        returns (bytes32 messageHash_)
    ```
    - Params
        - `amount`
          Amount of MOST that will be deposited on the origin chain and minted on the auxiliary chain.
        - `beneficiary`
          The `utMOST` token will be minted in this address
        - `feeGasPrice` and `feeGasLimit`
          This is used for reward calculation. This will determine how much a user is willing to pay the reward to the facilitator that confirms the message on the auxiliary chain.
    ### Confirm withdraw of MOST.
    TBD.
    ### Confirm new cogateway.
    TBD.
    ### Declare gateway active.
    TBD.
