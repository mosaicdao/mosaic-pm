# ConsensusGateway + CoConsensus Tasks:

1. Create a contract to calculate the KernelHash, as it will be used in multiple contracts (CoConsensus, Core).

## MessageInbox and MessageOutbox:
1. Update chainId to metachainId.
2. In message, rename `gasPrice` and `gasLimit` to `feeGasPrice` and `feeGasLimit` respectively

## Anchor:
~~1. A Function to return `maxItems`.
    - ConsensusGateway will use this value to number of storage roots in circular buffer.~~
## Define MOST interface.
## Define utMOST interface.
## Consensus: 
1. Add internal getters for `uint256 _feeGasPrice` and `uint256 _feeGasLimit`. Return constant values for now, technical governance can update the values(Later, not in the scope for now.). This values will be used while calling `ConsensusGateway::declareOpenKernel`.

## Core:

1. Add `Core::getOpenKernel()` function.
    - This will return `openKernelHeight` and `openKernelHash`.
    - This function will be called from `ConsensusGateway::declareOpenKernel`. `openKernelHeight` and `openKernelHash` is used in ConsensusGateway, and to avoid 2 separate calls on Core, we can get the values in one call.

## ConsensusGatewayBase:
1. This is a base contract for `ConsensusGateway` and `ConsensusCoGateway`.
    - ConsensusGatewayBase is ConsensusModule
    - It should have the following variables to store 
        - ERC20Token MOST/utMOST
            - `ERC20I public most;`
        - Mapping to store nonce of senders. This will track the nonce that is used in message declaration and confirmation.
            - `mapping(address => uint256) nonces;`
        - Store current metablock height. This will track the latest height of the latest open metablock.
            - `uint256 currentMetablockHeight;`
            
2. A function to hash kernel intent.
    - This hashing function will be used by both `ConsensusGateway` and `ConsensusCoGateway`.
    
    ```
        bytes32 public constant KERNEL_INTENT_TYPEHASH = keccak256(
            abi.encode(
                "KernelIntent(uint256 height,bytes32 kernelHash)"
            )
        )
    ```
    
    ```
        function hashKernelIntent(
            uint256 _height,
            bytes32 _kernelHash
        ) 
            public 
            returns (bytes32 kernelIntentHash_) 
    ```
## ERC20GatewayBase
1. A function to hash deposit intent.
    - This hashing function will be used by both `ConsensusGateway` and `ConsensusCoGateway`.

    ```
        bytes32 constant public DEPOSIT_INTENT_TYPEHASH = keccak256(
            abi.encode(
                "DepositIntent(uint256 amount,address beneficiary)"             )  
        );
    ```
    
    ```
    function hashDepositIntent(uint256 _amount, address _beneficiary)
    ```    
    
## ConsensusGateway:

1. ConsensusGatway contract (dependent on ConsensusGatewayBase.1)
    - This should follow proxy pattern.
        - `contract ConsensusGateway is MasterCopyNonUpgradable, MessageBus, ConsensusGatewayBase, ERC20GatewayBase`
    /* TODO: Make MessageBus */

    - Possible variables for this contracts.
        - Define a constant for storage index of message box outbox mapping.
            - `uint8 constant public OUTBOX_OFFSET = <Identify this>;`
            - `uint8 constant public INBOX_OFFSET = <Identify this>;`
    eg.
    - `MasterCopyNonUpgradable` should be at location 0.
    - `MessageOutbox` should be always at location 1.
    
    - **Setup function**.
        ```
        function setup(
            bytes32 _metachainId,
            address _consensus,
            address _consensusCogateway
        ) 
            external
        ```
        - This function can be called only once.
        - Call setup function of `ConsensusGatewayBase`
            ```
              ConsensusGatewayBase.setupConsensusGatewayBase(_consensus);
            ```
        - Call setup function of `MessageOutbox`
        
            ```
            MessageOutbox.setupMessageOutbox(
                _metachainId,
                _consensusCogateway,
                address(this)
            );
            ```
3. A function to declare open kernel. (Dependend on ConsensusGateway.1 and Core.1)

    
    ```
    function declareOpenKernel(CoreI _core, uint256 _feeGasPrice, uint256 _feeGasLimit)
        onlyConsensus
        external
        returns (bytes32 messageHash_)
    ```
    - Can only be called by `Consensus` contract.
    - Pass core contract address as param.
    - Assert that `_core` address is not zero.
    - Get `openKernelHeight`, `openKernelHash` from Core.
        - Here we get all the required values from core contract in one call instead of 4.
        ```
         (height, kernelHash) = core.getOpenKernel()
        ```
    - The `height` can be equal to `currentMetablockHeight` or `currentMetablockHeight + 1`. We should allow equal height, incase of chain halting when new core is created, then the new kernel can be opened at the same height. As this function can be called only by consensus, we can trust that same height will not be passed in normal conditions.
    - Update `currentMetablockHeight = height;`
    - Create `kernelIntentHash`. It is sha3(height, kernelHash)
    - Increament `nonce` for the caller(from nonces mapping).
    - Call `MessageOutbox.declareMessage` with the following params.
        - kernelIntentHash
        - nonce
        - _feeGasPrice
        - _feeGasLimit
        - msg.sender for sender

4. A function to deposit MOST token on the origin chain  and mint utMOST token on the auxiliary chain.
    ```
    function deposit(
        uint256 _amount,
        address _beneficiary,
        uint256 _feeGasPrice,
        uint256 _feeGasLimit,
    )
    ``` 
    - Anyone can all this function to deposit MOST token.
    - Validate input params.
    - Check if reward is possible with given input params amount, gasprice and gaslimit. If the reward is not possible then revert.
    - Create `depositIntentHash`. Call `ConsensusGatewayBase::hashStateIntent` to get the `depositIntentHash`.
    - Increament `nonce` for the caller(from nonces mapping).
    - Call `MessageOutbox.declareMessage` with the following params.
        - depositIntentHash
        - nonce
        - gasPrice
        - gasLimit
        - address(msg.sender i.e depositor)
    - Transfer `_amount` number of the MOST token from `address(msg.sender)` to ConsensusGateway `address(this)`
    - TODO: Discuss if any events is needed here.


    
## ConsensusCoGateway: ((dependent on ConsensusGatewayBase, Anchor.1))

1. ConsensusCoGatway constract
    - This should follow proxy pattern.
        - `contract ConsensusCoGateway is MasterCopyNonUpgradable, MessageInbox, ConsensusGatewayBase`
    - Possible variables for this contracts.
        - Define a constant for storage index of message box outbox mapping.
            - `uint8 constant public INBOX_OFFSET = 1;`
            - `ProtoCoreI public protocore;`
            - `mapping(uint256/*height*/ => bytes32/*kernelHash*/) kernelHashes;`
        - `MasterCopyNonUpgradable` should be at location 0.
        - `MessageInbox` should be always at location 1.

    - **Setup function.**
        ```
        function setup(
            bytes32 _metachainId,
            address _coConsensus,
            address _consensusGateway,
            uint8 _outboxStorageIndex,
            uint256 _maxStorageRootItems
        )
            external
        ```
        - This function can be called only once.
        - Call setup function of `ConsensusGatewayBase`.
            ```
            ConsensusGatewayBase.setupConsensusGatewayBase(
                _coConsensus
            );
            ```
        - Get `anchor` address from `CoConsensus` contract.        
        - Call setup function of `MessageOutbox`
        
        ```
        MessageInbox.setupMessageInbox(
            _metachainId,
            _consensusGateway,
            _outboxStorageIndex,
            anchor,
            _maxStorageRootItems
        );
        ```
2. Function to prove Consensus contract. (May be this can be merged with some other tickets.)
    
    ```
    function proveConsensusGateway(
        uint256 _blockHeight,
        bytes calldata _rlpAccount,
        bytes calldata _rlpParentNodes
    )
        external
    {
        MessageInbox.proveStorageAccount(
            _blockHeight,
            _rlpAccount,
            _rlpParentNodes
        );
    }
    ```
3. Confirm open kernel declared on auxiliary chain.
    ```
     function confirmOpenKernel(
        uint256 _height,
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
    
    - Check if the `_height` - `currentMetablockHeight` <=1
    - Update `currentMetablockHeight`. `currentMetablockHeight = height;`
    - Create `kernelIntentHash` with params `(_height, _kernelHash);`
    - Increase the nonce count for the `sender` address
    - Call confirm message on MessageInbox.
            ```
                messageHash_ = MessageInbox.confirmMessage(
                    kernelIntentHash,
                    nonce,
                    _feeGasPrice,
                    _feeGasLimit,
                    _sender,
                    _blockHeight,
                    _rlpParentNodes
                );
            ```
    - Update the mapping `kernelHashes`
4. Confirm deposit on auxiliary chain.
    ```
    function confirmDeposit(
        uint256 _amount,
        address payable _beneficiary,
        uint256 _gasPrice,
        uint256 _gasLimit,
        address _depositor,
        uint256 _blockHeight,
        bytes calldata _rlpParentNodes
    )
        external
        returns (bytes32 messageHash_)
    ```
    
    - Get the initial gas amount. `initialGas = gasleft();`. Intention here is to record the gas used in this function call. This gas used will be used for reward calculations.
    - Add validations for input params.
    - Check if reward is possible with given amount, gasprice and gaslimit. If reward is not possible then revert.
    - Increament the nonce for the sender address.
    - Create `depositIntentHash`. Call `ConsensusGatewayBase::hashStateIntent` to get the `depositIntentHash`.
    - Call confirm message in MessageInbox.
        - ```
            messageHash_ = MessageInbox.confirmMessage(
                depositIntentHash,
                nonce,
                _gasPrice,
                _gasLimit,
                _depositor,
                _blockHeight,
                _rlpParentNodes
            );
          ```                
    - Calculate the gas consumed. `initialGas.sub(gasleft());`
    - Calculate the reward.
    - Mint reward to the msg.sender account.
    - Mint token to the beneficiary account.

## `Consensus:newGateway`

Any DApp developer can deploy a custom Gateway for their DApp (or reuse the existing logic for an ERC20 gateway, an NFT gateway, ...)


## CoConsensus.
1. Function to commit meta block.
    ```
    function commitMetablock(
        bytes32 _parentKernelHash,
        uint256 _height,
        address[] _updatedValidators,
        uint256[] _updatedReputation,
        uint256 _gasTarget
    ) external 
    ```
   - Create the kernel hash from the given inputs.
   - Check the generated kernel hash is equal to the `consensusCoGateway::getKernelHash(height);`
   - Check if the `openKernelHash` and `openKernelHeight` is equalt to `_parentKernelHash` and `_height-1`.
       - ```
         (parentKernelHeight, parentKernelHash) = protoCore.openKernel();
         ```
   - Loop through `_updatedValidators` array.
       - ```
           if(reputation::upsert(validator, reputation)) {
               protocore.join(validator)
           }
           
           if(reputation == 0) {
               protocore.logout(validator)
           }
         ```
    - Call `protocore::openMetablock(height, parentKernelHash, openKernelHash, gasTarget)``

        
TODO: make a task.
- If the reputation decreases to zero, then logout from core and reputation.
## MOST:

## UTMOST:

## Protocore:
1. Variables:
    -  
3. (parentKernelHeight, parentKernelHash) = protoCore.openKernel();
    

## CoAnchor:





# Later 