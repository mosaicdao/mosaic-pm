---
id: withdraw
sidebar_label: Withdraw ERC20 tokens from metachain
---

# Withdraw ERC20 tokens from metachain

## Node script

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

## SDK (TODO)

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
