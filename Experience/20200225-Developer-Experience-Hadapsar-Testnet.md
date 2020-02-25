# Developer Experience

### Description

![](https://i.imgur.com/xXswcoE.png)

Hadapsar helps developers in scaling the DApp. The DApp contracts can be deployed on Hadapsar chain and can appreciate the provided horizontal scaling. Through the gateways, the developers can maintain the available economy for their respective tokens.

ERC20 tokens can be moved between chains by depositing on origin chain, which after facilitator action will be available as utility token on the metachain. If the token does not exist on the metachain then it will be deployed and the mapping will be created for all the registered tokens. These utility tokens can later be withdrawn from the metachain and will be made available to the desired beneficiary account on the origin chain.

Explorer helps in tracking the origin chain and the metachain. Transaction specific to the DApp can be tracked and monitored using the provided metachain explorer. The faucet can be used to get the gas on the metachain which is required for the deployment of the DApp contracts and also for performing the transactions.

#### Video(DEMO video with all the steps)(optional, if possible)

### Getting Started on Hadapsar testnet

#### Faucet

Gas is required to perform the transaction of depositing ERC20 token on the origin chain. You can fund your origin chain account using the faucet.

- Link for origin chain faucet:

Gas is also required to deploy the DApp contracts, perform transfers, perform withdraw and other chain related tasks on the metachain. You can fund your metachain account using the provided faucet.

- Link for metachain faucet(TODO: metachain explorer link can be added to our faucet):

#### Connect to Hadapsar testnet

You will require the RPC URL to connect to the origin chain to perform the transaction and interact with the gateway contracts

- RPC URL for origin chain: https://rpc.mosaicdao.org/goerli

You will also require the RPC URL to connect to the metachain to deploy the DApp contracts, perform transfers, perform transfers, provide support for blockchain interaction within DApp for the users, and other chain related tasks on the metachain.

- RPC URL for metachain: https://chain.mosaicdao.org/hadapsar

#### Deposit ERC20 tokens on origin chain

```
TODO:
- List of all the registered tokens(addresses)
- Gateway contract address, and other required missing config/envs
```

For depositing ERC20 tokens on the origin chain

- Node script

  - Install npm package dependencies
    ```sh
    npm install --save web3 ethereumjs-tx
    ```
  - Create `deposit.js` script as,
    **Note:** Identify missing parameters and improve script after testing(TODO)

    ```js
    const Web3 = require('web3');
    const Tx = require('ethereumjs-tx');

    const erc20GatewayContractABI = <CONTRACT_ABI(DEPOSIT_FUNCTION)_ON_ORIGIN>;
    const erc20GatewayContractAddress = <CONTRACT_ADDRESS_ON_ORIGIN>;

    const account = <ACCOUNT_ADDRESS>;
    const privateKey = new Buffer('<YOUR_PRIVATE_KEY>', 'hex');
    const valueTokenAddress = <VALUE_TOKEN_ADDRESS>;

    const web3Origin = new Web3('https://rpc.mosaicdao.org/goerli');

    const erc20GatewayContract = new web3Origin.eth.Contract(
      erc20GatewayContractABI,
      erc20GatewayContractAddress
    );

    const deposit = erc20GatewayContract.methods
      .deposit(amount, beneficiary, feeGasPrice, feeGasLimit, valueTokenAddress)
      .encodeABI();

    web3Origin.eth
      .getTransactionCount(account)
      .then(nonce => {
        const rawTxDeposit = {
          from: account,
          nonce: "0x" + nonce.toString(16),
          data: deposit,
          to: erc20GatewayContractAddress,
          gasLimit: 6500000,
          gasPrice: 10000000000
        };

        //sign transaction
        const txDeposit = new Tx(rawTxDeposit);

        txDeposit.sign(privateKey);

        const serializedTxDeposit = txDeposit.serialize();

        //send signed transaction (deposit)
        web3Origin.eth
          .sendSignedTransaction("0x" + serializedTxDeposit.toString("hex"))
          .on("receipt", receipt => {
            console.log(receipt);
          })
          .catch(console.error);
      });
    ```

  - Run using,
    ```sh
    node deposit.js
    ```

- SDK (TODO)

  - Create a config(config.js)

  ```js
  module.exports = {
    ORIGIN_CHAIN_RPC_URL: '',
    HADAPSAR_CHAIN_RPC_URL: '',
    PRIVATE_KEY: '',
    ACCOUNT_ADDRESS: '',
    GATEWAY_CONTRACT_ADDRESS: ''
    ...
  }
  ```

  - Create `deposit.js`

  ```js
  const Mosaic = require('mosaic-sdk');
  const config = require('./config');

  const mosaic = new Mosaic({
    ...
  });

  mosaic.deposit(
    ...
  )
  .then()
  .catch(error => {
    console.log(error);
  });
  ```

**Note:** If the token does not exists on the Hadapsar chain then facilitator service deploys the ERC20 token on Hadapsar chain and provides token address(using token mapping)

#### How to deploy DApp contracts on metachain

- Using Metamask and remix solidity browser
  1. Connect Metamask to the Hadapsar chain using [link](https://hackmd.io/QAY3itsfQzGSibKtN-GGdA?view#Connect-metamask-to-the-G%C3%B6erli-chain0)
  2. Open [remix solidity browser](https://remix.ethereum.org/)
     ![](https://i.imgur.com/DFi97Nm.png)
  3. Activate the modules for remix solidity browser
     ![](https://i.imgur.com/ofmU0zt.png)
  4. Provide permission to metamask to connect it to the Remix ethereum IDE
     ![](https://i.imgur.com/2Ot3ZlJ.png)
  5. Compile the DApp contract
     ![](https://i.imgur.com/jmwWpOc.png)
  6. Add the DApp contract
     ![](https://i.imgur.com/WuqJ0TH.png)
  7. Deploy the contracts
     - Get the deployed contract addresses and add them to your DApp for interaction

* Using truffle package

  1. Install truffle and truffle-hdwallet-provider packages as dependency to your node application

  ```sh
  npm install --save truffle truffle-hdwallet-provider
  ```

  2. Update `truffle-config.js` by adding the `hadapsar` network config and also add your 12 word mnemonic,

  ```js
  const HDWalletProvider = require('truffle-hdwallet-provider');
  const mnemonic = <PASTE YOUR 12 WORDS MNEMONIC HERE>;
  const metachainRpc = 'https://chain.mosaicdao.org/hadapsar';

  module.exports = {
    networks: {
      hapadsar: {
        provider: function() {
          return new HDWalletProvider(
            mnemonic,
            metachainRpc
          )
        },
        network_id: <TODO>,
        gas: <TODO>,
        gasPrice: <TODO>
      }
    }
  };
  ```

  3. Compile the contracts

  ```sh
  ./node_modules/.bin/truffle compile
  ```

  4. Deploy the contracts

  ```sh
  ./node_modules/.bin/truffle migrate --network hapadsar
  ```

  **Note**:

  - Do not forget to add the mnemonic to the truffle config.
  - The address at the first HD path to the mnemonic must have sufficient gas required for deployment.
  - The deployed contract addresses from the logs or build, update the DApp to reference the deployed contract addresses.

#### Withdraw ERC20 tokens from metachain

- Node script

  - Install npm package dependencies
    ```sh
    npm install --save web3 ethereumjs-tx
    ```
  - Create `deposit.js` script by adding your account address, private key, and the utility token address on the metachain,
    **Note:** Identify missing parameters and improve script after testing(TODO)

    ```js
    const Web3 = require('web3');
    const Tx = require('ethereumjs-tx');

    const erc20CogatewayContractABI = <CONTRACT_ABI(WITHDRAW_FUNCTION)_ON_HADAPSAR_TESTNET>;
    const erc20CogatewayContractAddress = <CONTRACT_ADDRESS_ON_HADAPSAR_TESTNET>;

    const account = <ACCOUNT_ADDRESS>;
    const privateKey = new Buffer('<YOUR_PRIVATE_KEY>', 'hex');
    const utilityTokenAddress = <UtilityTokenAddress>;

    const web3Metachain = new Web3('https://chain.mosaicdao.org/hadapsar');

    const erc20CogatewayContract = new web3Metachain.eth.Contract(
      erc20CogatewayContractABI,
      erc20CogatewayContractAddress
    );

    const withdraw = erc20CogatewayContract.methods
      .withdraw(amount, beneficiary, feeGasPrice, feeGasLimit, utilityTokenAddress)
      .encodeABI();

    web3Metachain.eth
      .getTransactionCount(account)
      .then(nonce => {
        const rawTxWithdraw = {
          from: account,
          nonce: "0x" + nonce.toString(16),
          data: withdraw,
          to: erc20CogatewayContractAddress,
          gasLimit: 6500000,
          gasPrice: 10000000000
        };

        //sign transaction
        const txWithdraw = new Tx(rawTxWithdraw);

        txWithdraw.sign(privateKey);

        const serializedTxWithdraw = txWithdraw.serialize();

        //send signed transaction (withdraw)
        web3Metachain.eth
          .sendSignedTransaction("0x" + serializedTxWithdraw.toString("hex"))
          .on("receipt", receipt => {
            console.log(receipt);
          })
          .catch(console.error);
      });
    ```

- SDK (TODO)

  - Create a config(config.js)

  ```js
  module.exports = {
    ORIGIN_CHAIN_RPC_URL: '',
    HADAPSAR_CHAIN_RPC_URL: '',
    PRIVATE_KEY: '',
    ACCOUNT_ADDRESS: '',
    GATEWAY_CONTRACT_ADDRESS: ''
    ...
  }
  ```

  - Create `withdraw.js`

  ```js
  const Mosaic = require('mosaic-sdk');
  const config = require('./config');

  const mosaic = new Mosaic({
    ...
  });

  mosaic.withdraw(
    ...
  )
  .then()
  .catch(error => {
    console.log(error);
  });
  ```

#### Explorer

- Link to Origin chain explorer:
- Link to Metachain explorer:

### Connect metamask to the Origin chain

1. Download metamask browser extension
   ![](https://i.imgur.com/oHP7xjZ.png)

2. Login to your metamask browser extension
   ![](https://i.imgur.com/8Zpvbg2.png)

3. Go to the network tab and add RPC url for Origin chain
   ![](https://i.imgur.com/U1lsBhd.png)

### Connect metamask to the Metachain

1. Download metamask browser extension
   ![](https://i.imgur.com/oHP7xjZ.png)

2. Login to your metamask browser extension
   ![](https://i.imgur.com/8Zpvbg2.png)

3. Go to the network tab and add RPC url for metachain
   ![](https://i.imgur.com/U1lsBhd.png)

### SDK reference

- Deposit ERC20 on Origin chain
- Withdraw ERC20 from Metachain
- Get token balance on Origin chain
- Get token balance on Metachain
- Get token mapping(address) on Metachain
