---
title: "M1 Facilitator Task Breakdown (sprint 2)"
disqus: https://hackmd.io/v28-vOHPRCOkScE9MUgRBA?both
---

# M1 Facilitator Task Breakdown  (sprint 2)

| Version | Last Updated | Component        |
| ------- | ------------ | ---------------- |
| 0.14    | 13/02/2020   | Task Breakdown  (sprint 2)|


## UtBase
1. Update Utmost->UtBase implementation (3)
    - Rename `Utmost` to `UtBase`
    - Remove `UtilityToken` dependencies from `UtBase`
        - The `mint` function of `UtBase` needs to perform 2 operations
            - Call internal `_mint` function of ERC20 token.
            - Unwrap the token to get the base token.
        - In the current implementation, the `mint` external function is defined in the `UtilityToken`.
        - In order to unwrap the token, the `mint` function needs to be overridden, resulting in to bypass the calls to Utility token.
        - `ERC20Token` already has `_mint`, `_burn` and `_burnFrom` internal functions. Currently the `UtBase is UtilityToken`, remove the dependencies of UtilityToken and directly use the functions from ERC20 token.
        - The mint function in the UtBase can only be called by `ConsensusCogateway` address.
        - The `mint external` function should not emit `Transfer` event as its already handled in the `_mint internal` function of ERC20Token..
        - The `mint`, `burn` and `burnFrom` functions must not return `boolean` value.

## Utility token

1. Update Utility token contract (2)
    - Make it master copy non upgradable 
    - update setup function
    ```
    function setup(
        string calldata _symbol,
        string calldata _name,
        uint8 _decimals,
        uint256 _totalTokenSupply,
        address _erc20Cogateway,
        address _valueToken
    )
        external
    ```
### Update ERC20GatewayBase, ConsensusGateway and ConsensusCogateway. (2)

- ERC20GatewayBase:
    - Update `DEPOSIT_INTENT_TYPEHASH` and `hashDepositIntent()` to include `address valueToken`.
    - Update `WITHDRAW_INTENT_TYPEHASH` and `hashWithdrawIntent()` to include `address valueToken,address utilityToken`.
- ConsensusGateway:
    - Update `deposit` function to pass most token in `hashDepositIntent()`.
    - Update `confirmWithdraw` function to pass most and utbase token address in `hashWithdrawIntent()`.
- ConsensusCogateway:
    - Update `withdraw` function to pass most and utbase token address in `hashWithdrawIntent()`.
    - Update `confirmDeposit` function to pass most address in `hashDepositIntent()`.

## Star Gateway
**HackMD document:** [Star Gateway HackMD Document](https://hackmd.io/vs6eBAk7QAK_4gQZbusqag)

1. ERC20Gateway::setup (2)
    - Implement the `setup` function
        ```
        function setup(
            bytes32 _metachainId,
            address _erc20Cogateway,
            address _stateRootProvider,
            uint256 _maxStorageRootItems,
            uint8 _outboxStorageIndex
        )
        ```
        - call `setupMessageOutbox`
        - call `setupMessageInbox`
1. Implement GenesisERC20Cogateway (1)
    - This will include the following storage variables
        - genesisMetachainId
        - genesisERC20Gateway
        - genesisStateRootProvider
        - genesisMaxStorageRootItems
        - genesisOutboxStoreageIndex
1. ERC20Cogateway::setup (2)
    - Implement the `setup` function
        ```
        function setup()
        ```
        - call `setupMessageOutbox`, using genesis storage variables
        - call `setupMessageInbox`, using genesis storage variables
        - Add a modifier isActive.
1. ERC20Gateway::deposit (2)
1. ERC20Cogateway::withdraw (2)
1. ERC20Cogateway::confirmDeposit (2)
1. ERC20Gateway::confirmWithdraw (2)
1. ERC20GatewayBase::proveGateway (2)
1. Implement integration test for deposit flow (3)
1. Implement integration test for withdraw flow (3)
1. Generate interacts and prepare publish npm (2)
1. Implement M0ERC20Cogateway (2)
	- `M0ERC20Cogateway is ERC20Cogateway`
	-  Override setup function. Do nothing.
	-  Implement activate function
	    ```
        function activate(
            bytes32 _metachainId,
            address _erc20Gateway
            address _stateRootProvider,
            uint256 _maxStorageRootItems,
            uint8 _outboxStorageIndex,        
        )
        ```
	- Store the parameters in the genesis storage variables.
	- Call ERC20Cogateway.setup();

## Subgraph M1 Facilitator
**HackMD document:** [Subgraph Gen1 Facilitator HackMD Document](https://hackmd.io/J5nwK6zySlm4204yIKnc1w)

1. Setup schema.graphql, mapping(handler), schema, subgraph.yml file for Anchor (2)
1. Upsert schema.graphql, mapping(handler), schema, subgraph.yml file for ERC20Gateway (2)
1. Upsert schema.graphql, mapping(handler), schema, subgraph.yml file for ERC20CoGateway (2)
4. Add command to deploy subgraph for each chain. 

**Note**: New Repo? + '4th point'

## Facilitator
**HackMD document:** [Facilitator specs](https://hackmd.io/L7imUcH7RMmOXB6-alyXgg)
### Folder restructure
1. Restructure repository to support different personas (don't define modes, it s an unneeded abstraction, be explicit about which personas to run in manifest) (2)
    - Move everything for m0-facilitator
      - For `src`, `test` and `test_integration`
      - using `git mv`

### Models & Repositories
1. Implement `DepositIntent` model and `DepositIntent repository`. (2)
1. Implement `WithdrawIntent` model and `WithdrawIntent repository`. (2)
1. Implement `Achor` model and `Anchor repository`. (2)
1. ~~Implement AnchoredStateRoots model and AnchoredStateRoots repository. (2)~~
1. Implement `Gateway` model and `Gateway repository`. (2)
1. Implement `Message` model and `Message repository`. (2)
1. Implement `ContractEntity` model and `ContractEntity repository`. (2)
1. ~~Implement `Metachain` model and `Metachain repository` (to be estimated).~~
1. ~~Implement `TrustedEndpoint` and `TrustedEndpointRepository` (to be estimated).~~
1. Implement `ERC20GatewayTokenPair` and `ERC20GatewayTokenPairRepositiry` (2) [Implementeer can suggest a better name]
    ```
    - ERc20Gateway address
    - ValueToken address
    - UtilityToken address    
    ```

### Manifest file
1. Define facilitator manifest and manifest schema(FOR M1); validate the manifest file. (2)
    - Proposed facilitator manifest:
    ```yaml=
    version: "v0.14"
    architecture_layout: M1
    personas:        
        facilitator
        validator        
    chain: // section for m0 utility chains
    metachain: // section metachains for m1
        auxiliary:
            avatar_account: ""
            node_endpoint: ""
            graph_ws_endpoint: ""
            graph_rpc_endpoint: ""
        origin:
            avatar_account: ""
            node_endpoint: ""
            graph_ws_endpoint: ""
            graph_rpc_endpoint: ""
    accounts:
        "0xac7E36b3cdDb14Bf1c67dC21fFB24C73d03d8FF7":
            keystore_path: ""
            keystore_password_path: ""
    origin_contract_addresses:
        erc20_gateway: ""
    facilitate_tokens:
        - 0xA
        - 0xB
    ```
**Note**: Testcases for validation of schema
1. Implement a function to check if the DB is initialized with the proper seed data. (1)
	- Check if the tables in the DB has the the following data as per the manifest file
		- ~~Metachain~~
		- Gateway
		- ~~Anchor~~
		- ~~ContractEntity~~
1. Implement facilitator init, support M1 faciltiator. (3)
    - Facilitator init. (if -f option is used then clear the DB)
    - Check if the facilitator is already initialized, if yes display error.
    - Create the DB (use gateway address in file path)
    - Connect to origin rpc endpoint
    - Get the `ERC20Cogateway` address from the provided `ERC20Gateway` address.
    - Get the `Anchor` contract address from the provided `ERC20Gateway` address.
    - Get the last proven block number form the origin achor contract.
    - Connect to auxiliary rpc endpoint
    - Get the `Anchor` contract address from the provided `ERC20Cogateway` address.
    - Get the last proven block number form the auxiliary achor contract.
    - Update the following seed data:
        - ~~Metachain~~
            
            ~~metachainId - < auxiliary metachain id >~~
            ~~originMetachainId - < origin metachain id >~~
            ~~anchorGA - < auxiliary_anchor_address >~~
            ~~mosaic version - < get from gateway contracts >~~
            ~~consensus address - null~~
            

        - Gateway
            ```
            GatewayGA - erc20gateway address
            RemoteGatewayGA - erc20Cogateway address
            Gateway type - erc20
            Destination global address - null.		
            RemoteGatewayLastProvenBlockNumber - block number form the origin achor contract
            AnchorGA - Origin anchor address.
            ```
            ```
            GatewayGA - erc20Cogateway address
            RemoteGatewayGA - erc20gateway address
            Gateway type - erc20
            Destination global address - null.		
            RemoteGatewayLastProvenBlockNumber - block number form the auxiliary achor contract
            AnchorGA - Auxiliary anchor address.
            ```	
        - Anchor
            ```
            AnchorGA - origin anchor address.
            blockHeight - anchored block height from origin anchor.

            ```	
            ```
            AnchorGA - auxiliary anchor address.
            blockHeight - anchored block height from auxiliary anchor.

            ```	
        - ContractEntity (already exists in the repo)
        

1. Implement facilitator start, support M1 faciltiator. (3)
    - Check if the manifest.yaml is valid.
    - Check if the seed data is populated as per the provided manifest file.
	- Subscribe to origin graph
	- Subscribe to auxlilairy graph

1. Gracefully stop facilitator (M0 and M1) (2)
    - Unsubscribe to the origin graph node, aux graph node
    - Clear set interval for resubscription


### Entity Handlers
1. Implement `DeclaredDepositIntent` handler (2)
1. Implement `ConfirmedDepositIntent` handler (2)
1. Implement `DeclaredWithdrawIntent` handler (2)
1. Implement `ConfirmedWithdrawIntent` handler (2)
1. Implement `StateRootAvailable` handler (2)
1. Implement `GatewayProven` handler (2)
1. Implement `CreatedUtilityToken` handler (2) 

### Services
1. Implement ConfirmDeposit service (3)
1. Implement ConfirmWithdraw service (3)
1. Implement ProveGateway service (2)


### M1 Facilitator Integration Tests
1. Chain setup for integration testing (3)
    - Take inspiration from contract integration tests
1. Implement integration test for deposit flow for ERC20token (5)
1. Implement integration test for withdraw flow for ERC20token (3)

### Facilitator Other Tasks
1. Implement Transaction executor. (3)
    - Revisit the nonce logic in testcases
1. ~~Implement ERC20GatewayAvatar~~
1. Readme update

### Implement System Test ()
Reference: https://github.com/mosaicdao/facilitator/issues/206

### Demo scripts

1. Implement script for creating depositer and withdrawer 
1. Implement script for deposit
1. Implement script for withdraw
1. Readme update

## Faucet

- Configure faucet 

## Blockscout explorer

- Support explorer to provide a view for all the transactions for hadapsar Gen1 testnet.

## Eth stats

- Eth stats support with new hadapsar Gen1 testnet.

## Devops

- Deploy ERC20Gateway and ERC20Cogateway contracts
- Setup star gateway facilitator



Total points 102
Total devs 
    - Abhay + 1  => 12      10     11
    - Sarvesh + 1 => 12     10
    - Gulshan + 1 => 12     10 
    - Pro () => 10           8
    - Deepesh (0.5) => 6     5
    
    - Jayesh (0)
    - Ben (0)
    
    Total => 52 dev days     43 
    
Velocity => 2 (~1.96)        2.37
    