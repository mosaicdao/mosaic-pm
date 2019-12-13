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
- ERC20GatewayOrganization
- ERC20Gateway
- ValueBrandedToken

## Tool to automate contracts verification on etherscan

- Proposed command
    ```bash
    npm run verify-contract <apiKey> <network> <githubRepoUrl> <contractName> <contractAddress> <contractCreationTxHash> <byteCode> <libraryContractConfig>
    ```
    - <b>apiKey</b>: Etherscan API key
    - <b>network</b>: Network where contract is deployed. e.g. "mainnet", "ropsten", "rinkeby", "kovan", "goerli"
    - <b>GithubRepoUrl</b>: Github exact tag/release contract repository which was used to deploy the contracts. Supported repositories:
        - mosaic-contracts
        - brandedtoken-contracts
        - openst-contracts
    - <b>contractName</b>: Contract which is to be verified on etherscan
    - <b>contractAddress</b>: Contract address to verify
    - <b>contractCreationTxHash</b>: Contract's creation transaction hash
    - <b>byteCode</b>: Bytecode which was used to deploy the contracts. This parameter is optional after `contract compilation` bugs are fixed
    - <b>libraryContractConfig</b>: It's an optional field. File path which contains verified library contracts details in json format. If library contracts are not verified, they need to be verified first. e.g.
        ```
            {
              "MerklePatriciaProof": "<contractAddress>",
              "MessageBus": "<contractAddress>"
          }
        ```

- Implementation Details
    - The tool will clone the given github release/tag repository. Supported repositories:
        - mosaic-contracts
        - brandedtoken-contracts
        - openst-contracts
    - Run `npm ci`
    - `npm run compile` needs to run. This will populate compiled contract build files in `contract/build` directory. 
        - Note: Make sure contracts are compiled with exact solc version which was deployed.
    - Update contract/build/<contractName>.json file network section:
    ```json
    "networks": {
        "1": {
          "address": "<contractAddress>",
          "transactionHash": "<contractCreationTxHash>",
          "links": {}
        }
    ```
    - To verify contracts which have links to library contracts, library contract addresses needs to be specified in the links section. 
      Library contracts should be verified before verifying linked library contract. e.g.
    ```json
    "networks": {
        "1": {
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

- Rotate etherscan API key
- BUG: brandedtoken-contracts and MosaicContracts is using different solc version(not 0.5.0) for contracts compilation.
   - mosaic-contracts PR: https://github.com/mosaicdao/mosaic-contracts/pull/802
   - brandedtoken-contracts PR: https://github.com/OpenST/brandedtoken-contracts/pull/181
- Truffle "beta" version is taking latest beta version for compilation
- Verification of contracts should be done as soon as they are deployed

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
    


