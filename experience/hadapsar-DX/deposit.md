---
id: deposit
title: Deposit ERC20 tokens on origin chain
sidebar_label: Deposit ERC20 tokens on origin chain
---

```
TODO:
- List of all the registered tokens(addresses)
- Gateway contract address, and other required missing config/envs
```

For depositing ERC20 tokens on the origin chain

## Node script

- Install npm package dependencies
  ```sh
  npm install --save web3 ethereumjs-tx
  ```
- Create `deposit.js` script as,
  **Note:** Identify missing parameters and improve script after testing(TODO)

  ```js
  const Web3 = require("web3");
  const Tx = require("ethereumjs-tx");

  const erc20GatewayContractABI = DEPOSIT_FUNCTION_CONTRACT_ABI_ON_ORIGIN;
  const erc20GatewayContractAddress = CONTRACT_ADDRESS_ON_ORIGIN;

  const account = ACCOUNT_ADDRESS;
  const privateKey = new Buffer(PRIVATE_KEY, "hex");
  const valueTokenAddress = VALUE_TOKEN_ADDRESS;

  const web3Origin = new Web3("https://rpc.mosaicdao.org/goerli");

  const erc20GatewayContract = new web3Origin.eth.Contract(
    erc20GatewayContractABI,
    erc20GatewayContractAddress
  );

  const deposit = erc20GatewayContract.methods
    .deposit(amount, beneficiary, feeGasPrice, feeGasLimit, valueTokenAddress)
    .encodeABI();

  web3Origin.eth.getTransactionCount(account).then(nonce => {
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
