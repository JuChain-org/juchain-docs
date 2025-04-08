# Price Oracle

The JuChain Oracle service provides reliable on-chain price data for decentralized applications (dApps). The JU-USDT Price Oracle specifically delivers real-time pricing of the JU token relative to USDT, supporting DeFi applications, trading platforms, and other smart contracts requiring precise price data.

#### Mainnet Deployment Information

* **Network**: JuChain Mainnet
* **Contract Address**: `0x49E5c7f25711abe668F404307b27f4bE4836B0e7`
* **Deployer Address**: `0x7389F1B4717F5152B6Cc107bce4A42a11dC0b76E`
* **Owner**: `0x5021A15FaAFEFEC1daCB1c8b24FFE3F3E3f7277b`
* **Updater Permission Address**: `0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D`
* **Deployment Transaction Hash**: `0x233654f76766fb0f9fd1377a573bed11c60a44c0cdc8e59340ffda333d191140`

#### Testnet Deployment Information

This document also includes information about the JuChain Testnet environment, where developers can test oracle functionality.

{% hint style="info" %}
The current public chain Solidity version compilation supports only <= 0.8.8. Support for subsequent versions will be gradually rolled out based on official announcements.
{% endhint %}

***

#### Contract Information

**Mainnet Contract Information**

* **Network**: JuChain Mainnet
* **Contract Address**: `0x49E5c7f25711abe668F404307b27f4bE4836B0e7`
* **Deployer Address**: `0x7389F1B4717F5152B6Cc107bce4A42a11dC0b76E`
* **Owner**: `0x5021A15FaAFEFEC1daCB1c8b24FFE3F3E3f7277b`
* **Updater Permission Address**: `0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D`
* **Deployment Transaction Hash**: `0x233654f76766fb0f9fd1377a573bed11c60a44c0cdc8e59340ffda333d191140`
* **Update Frequency**: Every 1 minute
* **Price Source**: Aggregated data from JU-USDT trading pairs across multiple centralized and decentralized exchanges

**Testnet Contract Information**

* **Network**: JuChain Testnet
* **Contract Address**: `0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95`
* **Deployer Address**: `0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0`
* **Updater Permission Address**: `0x4878683a8C3007258278824228a92aC4E072F050` (only this address can update prices)
* **Deployment Transaction Hash**: `0x7e42454909c3ea9b52af4af84217149a87aa71aa08f129e11a01d5cea0989659`
* **Update Frequency**: Every 1 minute
* **Price Source**: Aggregated data from JU-USDT trading pairs across multiple centralized and decentralized exchanges

**Permission Details**

* **Deployer (Owner)**:
  * Mainnet: `0x5021A15FaAFEFEC1daCB1c8b24FFE3F3E3f7277b`, responsible for contract management and permission allocation.
  * Testnet: `0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0`, responsible for contract management and permission allocation.
* **Authorized Updater**:
  * Mainnet: `0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D`, the only account authorized to call `updatePrice` to update prices.
  * Testnet: `0x4878683a8C3007258278824228a92aC4E072F050`, the only account authorized to call `updatePrice` to update prices.
* **Design Change**: The new oracle version separates the deployer and price updater roles to enhance security.

***

#### Key Methods

**`getLatestPrice`**

* **Description**: Retrieves the latest price information for the JU-USDT trading pair.
* **Returns**:
  * `string`: Trading pair symbol (e.g., `"JU/USDT"`).
  * `uint256`: Latest price (must be divided by the precision value).
  * `uint256`: Last update timestamp (Unix timestamp).
*   **Call Example**:

    ```solidity
    (string memory symbol, uint256 price, uint256 timestamp) = oracle.getLatestPrice();
    ```

**`latestPrice`**

* **Description**: Retrieves the latest price value (without additional information).
* **Returns**:
  * `uint256`: Latest price value.

**`pricePrecision`**

* **Description**: Retrieves the price precision value used to calculate the actual price.
* **Returns**:
  * `uint256`: Price precision (e.g., `1e8`, actual price = returned price / precision).

**`lastUpdatedAt`**

* **Description**: Retrieves the timestamp of the last price update.
* **Returns**:
  * `uint256`: Unix timestamp.

**`symbol`**

* **Description**: Retrieves the trading pair symbol.
* **Returns**:
  * `string`: Trading pair symbol (e.g., `"JU/USDT"`).

**`updatePrice`**

* **Description**: Updates the JU-USDT price, callable only by the authorized updater.
* **Parameters**:
  * `uint256 _price`: New price value.
* **Permission**: Only `0x4878683a8C3007258278824228a92aC4E072F050` can call this.

**`setAuthorizedUpdater`**

* **Description**: Sets a new authorized updater, callable only by the Owner.
* **Parameters**:
  * `address newAuthorizedUpdater`: New authorized updater address.
* **Permission**: Only `0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0` can call this.

**`owner`**

* **Description**: Retrieves the current contract owner address.
* **Returns**:
  * `address`: Owner address.

***

#### ABI

Below is the complete ABI for the JU-USDT Price Oracle:

```json
[
  {
    "inputs": [
      {"internalType": "address", "name": "initialOwner", "type": "address"},
      {"internalType": "address", "name": "initialAuthorizedUpdater", "type": "address"}
    ],
    "stateMutability": "nonpayable",
    "type": "constructor"
  },
  {"inputs": [{"internalType": "address", "name": "owner", "type": "address"}], "name": "OwnableInvalidOwner", "type": "error"},
  {"inputs": [{"internalType": "address", "name": "account", "type": "address"}], "name": "OwnableUnauthorizedAccount", "type": "error"},
  {
    "anonymous": false,
    "inputs": [
      {"indexed": false, "internalType": "address", "name": "previousUpdater", "type": "address"},
      {"indexed": false, "internalType": "address", "name": "newUpdater", "type": "address"}
    ],
    "name": "AuthorizedUpdaterChanged",
    "type": "event"
  },
  {
    "anonymous": false,
    "inputs": [
      {"indexed": true, "internalType": "address", "name": "previousOwner", "type": "address"},
      {"indexed": true, "internalType": "address", "name": "newOwner", "type": "address"}
    ],
    "name": "OwnershipTransferred",
    "type": "event"
  },
  {
    "anonymous": false,
    "inputs": [
      {"indexed": false, "internalType": "uint256", "name": "price", "type": "uint256"},
      {"indexed": false, "internalType": "uint256", "name": "timestamp", "type": "uint256"},
      {"indexed": false, "internalType": "address", "name": "updater", "type": "address"}
    ],
    "name": "PriceUpdated",
    "type": "event"
  },
  {"inputs": [], "name": "renounceOwnership", "outputs": [], "stateMutability": "nonpayable", "type": "function"},
  {"inputs": [{"internalType": "address", "name": "newAuthorizedUpdater", "type": "address"}], "name": "setAuthorizedUpdater", "outputs": [], "stateMutability": "nonpayable", "type": "function"},
  {"inputs": [{"internalType": "address", "name": "newOwner", "type": "address"}], "name": "transferOwnership", "outputs": [], "stateMutability": "nonpayable", "type": "function"},
  {"inputs": [{"internalType": "uint256", "name": "_price", "type": "uint256"}], "name": "updatePrice", "outputs": [], "stateMutability": "nonpayable", "type": "function"},
  {"inputs": [], "name": "authorizedUpdater", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "stateMutability": "view", "type": "function"},
  {"inputs": [], "name": "getLatestPrice", "outputs": [{"internalType": "string", "name": "", "type": "string"}, {"internalType": "uint256", "name": "", "type": "uint256"}, {"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
  {"inputs": [], "name": "lastUpdatedAt", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
  {"inputs": [], "name": "latestPrice", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
  {"inputs": [], "name": "owner", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "stateMutability": "view", "type": "function"},
  {"inputs LuaTeX error (string "[]"):1: unexpected symbol near ']'.
": [], "name": "pricePrecision", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
  {"inputs": [], "name": "symbol", "outputs": [{"internalType": "string", "name": "", "type": "string"}], "stateMutability": "view", "type": "function"}
]
```

***

#### Usage Examples

**Web3.js Example**

The following code demonstrates how to interact with the oracle using Web3.js:

```javascript
const { Web3 } = require('web3');

// Mainnet RPC
const mainnetRpc = 'https://rpc.juchain.org';
// Testnet RPC
const testnetRpc = 'https://testnet-rpc.juchain.org';

// Select network (Mainnet/Testnet)
const web3 = new Web3(mainnetRpc); // or testnetRpc

// Mainnet contract address
const mainnetContractAddress = '0x49E5c7f25711abe668F404307b27f4bE4836B0e7';
// Testnet contract address
const testnetContractAddress = '0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95';

// Select contract address
const contractAddress = mainnetContractAddress; // or testnetContractAddress
const contract = new web3.eth.Contract(abi, contractAddress);

// Get complete price information
async function getCompletePrice() {
  try {
    const [symbol, price, timestamp] = await contract.methods.getLatestPrice().call();
    const precision = await contract.methods.pricePrecision().call();
    console.log('Complete Price Information:');
    console.log(`Trading Pair: ${symbol}`);
    console.log(`Price: ${price / precision} JU/USDT`);
    console.log(`Timestamp: ${timestamp} (${new Date(Number(timestamp) * 1000).toLocaleString()})`);
    return { symbol, price, timestamp, precision };
  } catch (error) {
    console.error('Failed to fetch price information:', error);
  }
}

// Get latest price
async function getLatestPrice() {
  try {
    const price = await contract.methods.latestPrice().call();
    const precision = await contract.methods.pricePrecision().call();
    console.log(`Latest Price: ${price / precision} JU/USDT`);
    return price;
  } catch (error) {
    console.error('Failed to fetch latest price:', error);
  }
}

// Get authorized updater
async function getAuthorizedUpdater() {
  try {
    const updater = await contract.methods.authorizedUpdater().call();
    console.log(`Authorized Updater: ${updater}`);
    return updater;
  } catch (error) {
    console.error('Failed to fetch authorized updater:', error);
  }
}

// Update price (only for authorized updater)
async function updatePrice(newPrice, privateKey) {
  try {
    const account = web3.eth.accounts.privateKeyToAccount(privateKey);
    web3.eth.accounts.wallet.add(account);
    const tx = contract.methods.updatePrice(newPrice);
    const gas = await tx.estimateGas({ from: account.address });
    const receipt = await tx.send({ from: account.address, gas });
    console.log(`Price updated successfully, Transaction Hash: ${receipt.transactionHash}`);
    return receipt;
  } catch (error) {
    console.error('Failed to update price:', error);
  }
}

// Execute all queries
async function getAllInfo() {
  await getCompletePrice();
  await getLatestPrice();
  await getAuthorizedUpdater();
}

getAllInfo();
```

**Solidity Example**

Below is an example of a smart contract utilizing the oracle:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IJUOracle {
    function getLatestPrice() external view returns (string memory, uint256, uint256);
    function pricePrecision() external view returns (uint256);
}

contract JUPriceConsumer {
    IJUOracle public oracle;

    constructor(address _oracleAddress) {
        oracle = IJUOracle(_oracleAddress);
    }

    function getJUPrice() public view returns (uint256) {
        (, uint256 price, ) = oracle.getLatestPrice();
        uint256 precision = oracle.pricePrecision();
        return price; // Frontend needs to divide by precision
    }

    function getFormattedPrice() public view returns (string memory symbol, uint256 price, uint256 precision, uint256 timestamp) {
        (symbol, price, timestamp) = oracle.getLatestPrice();
        precision = oracle.pricePrecision();
        return (symbol, price, precision, timestamp);
    }
}
```

***

#### Important Notes

* **Network Selection**:
  * Mainnet: For production environments, contract address is `0x49E5c7f25711abe668F404307b27f4bE4836B0e7`.
  * Testnet: For development and testing, contract address is `0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95`.
* **Price Precision**: The `price` returned by the contract must be divided by the value from `pricePrecision()` (e.g., `1e8` indicates 8 decimal precision).
* **Update Permissions**:
  * Mainnet: Only `0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D` can call `updatePrice`.
  * Testnet: Only `0x4878683a8C3007258278824228a92aC4E072F050` can call `updatePrice`.
  * The Owner can change the updater via `setAuthorizedUpdater`.
* **Update Delay**: Prices are updated every 1 minute. Check `lastUpdatedAt` to ensure data freshness before use.
* **Price Volatility**: Cryptocurrency prices may fluctuate significantly; consider implementing risk management mechanisms in your dApp.
* **Contract Upgrades**: The oracle contract may be updated. Stay tuned to official announcements for the latest addresses and features.
* **Mainnet Considerations**:
  * Operations on the mainnet incur real costs; exercise caution.
  * Thoroughly test on the testnet before deploying to the mainnet.

