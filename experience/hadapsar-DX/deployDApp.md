---
id: deploy
sidebar_label: Deploy DApp contracts on metachain
---

# Deploy DApp contracts on metachain

## Using Metamask and remix solidity browser

1. Connect Metamask to the Hadapsar chain using [link](metamask.md)
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

## Using truffle package

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
