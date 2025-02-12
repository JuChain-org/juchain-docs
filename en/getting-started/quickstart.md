# Quick Start

This guide will help you quickly get started with JuChain development.

## Prerequisites

Before you begin, make sure you have:

1. A Web3 wallet (MetaMask or OKX Wallet)
2. Basic understanding of blockchain development
3. Development environment setup (Node.js, Git)

## Development Steps

### 1. Connect to JuChain Network

First, add JuChain network to your wallet:

* Network Name: JuChain Testnet
* RPC URL: https://testnet-rpc.juchain.org
* Chain ID: 66633666
* Currency Symbol: JU
* Block Explorer: http://explorer-testnet.juchain.org/

### 2. Get Test Tokens

1. Visit the faucet: http://faucet-testnet.juchain.org/
2. Enter your wallet address
3. Receive test tokens

### 3. Start Development

#### Using Web3.js

```javascript
const Web3 = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// Check connection
web3.eth.net.isListening()
   .then(() => console.log('Connected!'))
   .catch(err => console.error('Connection failed'));
```

#### Using Ethers.js

```javascript
const { ethers } = require('ethers');
const provider = new ethers.providers.JsonRpcProvider('https://testnet-rpc.juchain.org');

// Check connection
provider.getNetwork()
   .then(network => console.log('Connected to network:', network.name))
   .catch(err => console.error('Connection failed'));
```

### 4. Deploy Smart Contracts

1. Install development framework (e.g., Hardhat):
```bash
npm install --save-dev hardhat
```

2. Configure network in hardhat.config.js:
```javascript
module.exports = {
  networks: {
    juchain: {
      url: "https://testnet-rpc.juchain.org",
      chainId: 66633666
    }
  }
};
```

3. Deploy your contract:
```bash
npx hardhat run scripts/deploy.js --network juchain
```

## Next Steps

- Explore our [Developer Documentation](../basics/smart-contracts.md)
- Join our [Developer Community](https://discord.gg/juchain)
- Check out [Sample Projects](https://github.com/juchain/examples)

## Support

If you need help:
1. Visit our [Documentation](https://docs.juchain.org)
2. Join our [Discord Community](https://discord.gg/juchain)
3. Submit issues on [GitHub](https://github.com/juchain/juchain) 