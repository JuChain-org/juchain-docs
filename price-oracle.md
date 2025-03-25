# Price Oracle

The JuChain Oracle service delivers reliable on-chain price data for decentralized applications (dApps). The JU-USDT Price Oracle specifically provides real-time pricing of JU tokens against USDT, empowering DeFi apps, trading platforms, and smart contracts needing accurate price feeds. This guide is tailored for the JuChain Testnet, where developers can test oracle functionality.

{% hint style="info" %}
The current public chain supports Solidity compilation up to version 0.8.8 only. Support for later versions will roll out gradually per official updates.
{% endhint %}

***

### Contract Details

* **Network**: JuChain Testnet
* **Contract Address**: `0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95`
* **Deployer Address**: `0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0`
* **Update Authority Address**: `0x4878683a8C3007258278824228a92aC4E072F050` (only this address can update prices)
* **Deployment Tx Hash**: `0x7e42454909c3ea9b52af4af84217149a87aa71aa08f129e11a01d5cea0989659`
* **Update Frequency**: Every 1 minute
* **Price Source**: Aggregates JU-USDT pair data from multiple centralized and decentralized exchanges

#### Permissions

* **Deployer (Owner)**: `0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0`—manages the contract and assigns permissions.
* **Authorized Updater**: `0x4878683a8C3007258278824228a92aC4E072F050`—the sole account allowed to call `updatePrice`.
* **Design Update**: Future versions will separate deployer and updater roles for enhanced security.

***

### Key Methods

#### `getLatestPrice`

* **Description**: Fetches the latest JU-USDT price info.
* **Returns**:
  * `string`: Trading pair symbol (e.g., `"JU/USDT"`)
  * `uint256`: Latest price (divide by precision for actual value)
  * `uint256`: Last update timestamp (Unix timestamp)
*   **Example Call**:

    ```solidity
    (string memory symbol, uint256 price, uint256 timestamp) = oracle.getLatestPrice();
    ```

#### `latestPrice`

* **Description**: Retrieves the latest price value only.
* **Returns**:
  * `uint256`: Latest price

#### `pricePrecision`

* **Description**: Gets the price precision for calculating the real price.
* **Returns**:
  * `uint256`: Precision (e.g., `1e8`; actual price = returned price / precision)

#### `lastUpdatedAt`

* **Description**: Returns the timestamp of the last price update.
* **Returns**:
  * `uint256`: Unix timestamp

#### `symbol`

* **Description**: Fetches the trading pair symbol.
* **Returns**:
  * `string`: Pair symbol (e.g., `"JU/USDT"`)

#### `updatePrice`

* **Description**: Updates the JU-USDT price (restricted to the authorized updater).
* **Parameters**:
  * `uint256 _price`: New price value
* **Permission**: Only callable by `0x4878683a8C3007258278824228a92aC4E072F050`

#### `setAuthorizedUpdater`

* **Description**: Assigns a new authorized updater (restricted to the owner).
* **Parameters**:
  * `address newAuthorizedUpdater`: New updater address
* **Permission**: Only callable by `0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0`

#### `owner`

* **Description**: Returns the current contract owner’s address.
* **Returns**:
  * `address`: Owner address

***

### ABI

Here’s the full ABI for the JU-USDT Price Oracle:

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
  {"inputs": [], "name": "pricePrecision", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
  {"inputs": [], "name": "symbol", "outputs": [{"internalType": "string", "name": "", "type": "string"}], "stateMutability": "view", "type": "function"}
]
```

***

### Usage Examples

#### Web3.js Example

Here’s how to interact with the oracle using Web3.js:

```javascript
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

const contractAddress = '0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95';
const contract = new web3.eth.Contract(abi, contractAddress);

// Fetch full price info
async function getCompletePrice() {
  try {
    const [symbol, price, timestamp] = await contract.methods.getLatestPrice().call();
    const precision = await contract.methods.pricePrecision().call();
    console.log('Full Price Info:');
    console.log(`Pair: ${symbol}`);
    console.log(`Price: ${price / precision} JU/USDT`);
    console.log(`Timestamp: ${timestamp} (${new Date(Number(timestamp) * 1000).toLocaleString()})`);
    return { symbol, price, timestamp, precision };
  } catch (error) {
    console.error('Failed to fetch price info:', error);
  }
}

// Fetch latest price only
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

// Fetch authorized updater
async function getAuthorizedUpdater() {
  try {
    const updater = await contract.methods.authorizedUpdater().call();
    console.log(`Authorized Updater: ${updater}`);
    return updater;
  } catch (error) {
    console.error('Failed to fetch authorized updater:', error);
  }
}

// Update price (authorized updater only)
async function updatePrice(newPrice, privateKey) {
  try {
    const account = web3.eth.accounts.privateKeyToAccount(privateKey);
    web3.eth.accounts.wallet.add(account);
    const tx = contract.methods.updatePrice(newPrice);
    const gas = await tx.estimateGas({ from: account.address });
    const receipt = await tx.send({ from: account.address, gas });
    console.log(`Price updated successfully, Tx Hash: ${receipt.transactionHash}`);
    return receipt;
  } catch (error) {
    console.error('Failed to update price:', error);
  }
}

// Run all queries
async function getAllInfo() {
  await getCompletePrice();
  await getLatestPrice();
  await getAuthorizedUpdater();
}

getAllInfo();
```

#### Solidity Example

Here’s a sample smart contract consuming the oracle:

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
        return price; // Frontend must divide by precision
    }

    function getFormattedPrice() public view returns (string memory symbol, uint256 price, uint256 precision, uint256 timestamp) {
        (symbol, price, timestamp) = oracle.getLatestPrice();
        precision = oracle.pricePrecision();
        return (symbol, price, precision, timestamp);
    }
}
```

***

### Key Notes

* **Price Precision**: The returned `price` must be divided by `pricePrecision()` (e.g., `1e8` for 8-decimal precision) to get the actual value.
* **Update Permissions**: Only `0x4878683a8C3007258278824228a92aC4E072F050` can call `updatePrice`. The owner can change this via `setAuthorizedUpdater`.
* **Update Delay**: Prices refresh every 1 minute—check `lastUpdatedAt` to ensure data freshness.
* **Testnet Only**: This contract is for the JuChain Testnet, not production.
* **Price Volatility**: Crypto prices can swing wildly—add risk management to your dApp.
* **Contract Upgrades**: The oracle may be updated—stay tuned to official announcements for the latest address and features.
