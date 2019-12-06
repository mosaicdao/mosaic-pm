# Etherscan contract verification tool

Aim of the tool is to automate and verify contracts on Etherscan (mainnet, Ropsten and Goerli).

## Origin contracts to verify per auxiliary chain
- Anchor
- AnchorOrganization
- OSTGatewayOrganization
- OSTEIP20Gateway
- MerklePatriciaProof
- MessageBus
- GatewayLib

## Origin contracts to verify per token
- erc20GatewayOrganization
- erc20Gateway
- ValueBrandedToken

## Tool to automate etherscan verification

We should create tool/script which should automate verification of contracts on etherscan.

- Proposed command
    ```bash
    npm run verify-contract <apiKey> <network> <githubRepoUrl> <byteCode> <contractName> <contractAddress> <contractCreationTxHash>
    ```
    - <b>apiKey</b>: Etherscan API key.
    - <b>network</b>: Network where contract was deployed. e.g. "mainnet", "ropsten", "rinkeby", "kovan", "goerli".
    - <b>GithubRepoUrl</b>: Github exact tag/release contract repository which was used to deploy the contracts.
    - <b>byteCode</b>: Bytecode which was used to deploy the contracts. 
    - <b>contractName</b>: Contract which is to be verified on etherscan.
    - <b>contractAddress</b>: Contract address to verify.
    - <b>contractCreationTxHash</b>: Contract's creation transaction hash.

- Pre-requisite
    - npm `verify-on-etherscan` should be installed. Checkout [more details] (https://github.com/gnosis/verify-on-etherscan#as-a-cli-utility) about the npm.

- Description
    - The tool will clone the given github release/tag repository.
    - run `npm ci`
    - `npm run compile` needs to run. This will populate compiled contract build files in contract/build directory.
        
        BUGs
        - BrandedTokenContracts:: package.json and MosaicContracts:: package.json is using publisher global truffle version for compilation
        - Truffle "beta" version is taking latest beta version for compilation
    - Update contract/build/<contractName>.json file network section:
    ```json
    "networks": {
        "3": {
          "address": "<contractAddress>",
          "transactionHash": "<contractCreationTxHash>",
          "links": {}
        }
    ```
    For linked library contract address, library contract addresses needs to be specified in the links section. Library contracts should be verified before verifying linked library contract.
    ```json
    "networks": {
        "3": {
          "address": "<gatewayContractAddress>",
          "transactionHash": "<gatewayContractCreationTxHash>",
          "links": {
              "MerklePatriciaProof": "<contractAddress>",
              "MessageBus": "<contractAddress>"
          }
        }
     ```

- `verify-on-etherscan.verify` method will be called from inside the script

    ```javascript
        const verify = require('verify-on-etherscan');
        const result = await verify({
          cwd,
          artifacts,
          apiKey,
          web3,
          network,
          useFetch,
          optimizer,
          output,
          delay,
          logger,
          verbose
        })
    ```
    
## Notes
- Verification of contracts should be done as soon as they are deployed
- Rotate etherscan API key

## Chain contracts verification

- Anchor organization - Done
https://etherscan.io/address/0xc450a2fa865972fb7a07acad6cf6c3654e37c617#code
solc version: v0.5.2+commit.1df8f40c

- OSTGateway organization - Done
https://etherscan.io/address/0x813bcae43e1aff7466d9d34a8163a2bf5216dc1f
solc version: v0.5.2+commit.1df8f40c

- OSTGateway - Done
https://etherscan.io/address/0xe0a3929c6947cdc091e6e6725826a09615062131#code
compiler version: v0.5.3+commit.10d17f24

- Anchor - Done
https://etherscan.io/address/0x981884d4369f7b7892d8c279d2125daf7422d9e0
compiler version: v0.5.3+commit.10d17f24
    
## Pepo contracts verification

- BrandedToken - DONE
https://etherscan.io/address/0x8d3d262fb1139d5d55d9ccbe7fff5fc45f242184#code  
solc version: v0.5.0+commit.1d4f565a

- Organization - Done
 https://etherscan.io/address/0x377cecea5900b9483320dff639ca49a7e2df15d1#code
 solc version: v0.5.2+commit.1df8f40c
 
- MerklePatriciaProof - Done 
  https://etherscan.io/address/0x30db9dd1875dbea601ba061e81dd88325d68fd18
  solc version: v0.5.2+commit.1df8f40c
  
- MessageBus - Done
    https://etherscan.io/address/0x0b71bb83d2814633f7dfeb9cb5c4c2e63be215ed
  solc version: v0.5.3+commit.10d17f24

- GatewayLib - Done
    https://etherscan.io/address/0x1c9e18c8ab156753f7eca95d8c042f7842aaf09e
  compiler version: v0.5.3+commit.10d17f24  

- EIP20Gateway - Done
    https://etherscan.io/address/0x73ae859256da871afd09bf4d45fd60936b15d8ea
  compiler version: v0.5.3+commit.10d17f24    
    


