# Etherscan contract verification tool

Aim of the tool is to automate and verify contracts on Etherscan (mainnet, Ropsten and Goerli).

## Tool to automate etherscan verification

We should create tool/script which should automate verification of contracts on etherscan.

- Proposed command
    ```bash
    npm run verify-contract <apiKey> <network> <githubRepoUrl> <contractName>     <contractAddress> <contractCreationTxHash>
    ```
    - <b>apiKey</b>: Etherscan API key.
    - <b>network</b>: Network where contract was deployed. e.g. "mainnet", "ropsten", "rinkeby", "kovan", "goerli".
    - <b>GithubRepoUrl</b>: Github exact tag/release version which was used to deploy the contracts.
    - <b>contractName</b>: Contract which is to be verified on etherscan.
    - <b>contractAddress</b>: Contract address to verify.
    - <b>contractCreationTxHash</b>: Contract's creation transaction hash.

- Pre-requisite
    - npm `verify-on-etherscan` should be installed. Checkout [more details] (https://github.com/gnosis/verify-on-etherscan#as-a-cli-utility) about the npm.

- Description
    - The tool will clone the given github release/tag repository.
    - `truffle compile` needs to run. This will populate compiled contract build files in contract/build directory.
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
