# Price Oracle

## JU-USDT Price Oracle Documentation

### Overview

JuChain Oracle Service provides reliable on-chain price data for decentralized applications. The JU-USDT price oracle specifically offers real-time price data for the JU token relative to USDT, supporting DeFi applications, trading platforms, and other smart contracts requiring accurate price information.

### Contract Information

* **Contract Address**: `0xC00E9C9a2ec48e6032255DF4975EfC63c6995881`
* **Network**: JuChain Testnet
* **Update Frequency**: Every minutes
* **Price Source**: Aggregated data from multiple centralized and decentralized exchanges for the JU-USDT trading pair

### Main Methods

#### getLatestPrice

Retrieves the latest price information for the JU-USDT trading pair.

**Returns**:

* `string`: Trading pair symbol (e.g., "JU/USDT")
* `uint256`: Latest price (needs to be divided by precision value for actual price)
* `uint256`: Last update timestamp (Unix timestamp format)

**Example Call**:

```solidity
(string memory symbol, uint256 price, uint256 timestamp) = oracle.getLatestPrice();
```

#### latestPrice

Gets the latest price value (without additional information).

**Returns**:

* `uint256`: Latest price value

#### pricePrecision

Gets the price precision value used to calculate the actual price.

**Returns**:

* `uint256`: Price precision (e.g., if precision is 1e8, then actual price = returned price / 1e8)

#### lastUpdatedAt

Gets the timestamp when the price was last updated.

**Returns**:

* `uint256`: Unix timestamp of the last update

#### symbol

Gets the trading pair symbol tracked by the oracle.

**Returns**:

* `string`: Trading pair symbol (e.g., "JU/USDT")

### Usage Examples

#### Web3.js Example

```javascript
// Import method suitable for Web3.js v4.x
const { Web3 } = require('web3');

// Initialize web3 instance, connecting to the Ethereum node
const web3 = new Web3('https://testnet-rpc.juchain.org');

// Contract address and ABI
const contractAddress = '0xC00E9C9a2ec48e6032255DF4975EfC63c6995881';
const contractABI = [
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "initialOwner",
				"type": "address"
			}
		],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "owner",
				"type": "address"
			}
		],
		"name": "OwnableInvalidOwner",
		"type": "error"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "OwnableUnauthorizedAccount",
		"type": "error"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "previousOwner",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "OwnershipTransferred",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "price",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"name": "PriceUpdated",
		"type": "event"
	},
	{
		"inputs": [],
		"name": "renounceOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "transferOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "_price",
				"type": "uint256"
			}
		],
		"name": "updatePrice",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getLatestPrice",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "lastUpdatedAt",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "latestPrice",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "owner",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "pricePrecision",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "symbol",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	}
];

// Create contract instance
const contract = new web3.eth.Contract(contractABI, contractAddress);

// Get complete price information
async function getCompletePrice() {
  try {
    const result = await contract.methods.getLatestPrice().call();
    console.log('Complete price information:');
    console.log(`Token symbol: ${result[0]}`);
    console.log(`Price: ${result[1]}`);
    console.log(`Timestamp: ${result[2]}`);
    console.log(`Time: ${new Date(Number(result[2]) * 1000).toLocaleString()}`);
    return result;
  } catch (error) {
    console.error('Error getting complete price information:', error);
  }
}

// Get latest price
async function getLatestPrice() {
  try {
    const price = await contract.methods.latestPrice().call();
    console.log(`Latest price: ${price}`);
    return price;
  } catch (error) {
    console.error('Error getting latest price:', error);
  }
}

// Get last updated time
async function getLastUpdatedAt() {
  try {
    const timestamp = await contract.methods.lastUpdatedAt().call();
    const date = new Date(Number(timestamp) * 1000);
    console.log(`Last updated time: ${timestamp} (${date.toLocaleString()})`);
    return timestamp;
  } catch (error) {
    console.error('Error getting last updated time:', error);
  }
}

// Get token symbol
async function getSymbol() {
  try {
    const symbol = await contract.methods.symbol().call();
    console.log(`Token symbol: ${symbol}`);
    return symbol;
  } catch (error) {
    console.error('Error getting token symbol:', error);
  }
}

// Get price precision
async function getPricePrecision() {
  try {
    const precision = await contract.methods.pricePrecision().call();
    console.log(`Price precision: ${precision}`);
    return precision;
  } catch (error) {
    console.error('Error getting price precision:', error);
  }
}

// Get contract owner
async function getOwner() {
  try {
    const owner = await contract.methods.owner().call();
    console.log(`Contract owner: ${owner}`);
    return owner;
  } catch (error) {
    console.error('Error getting contract owner:', error);
  }
}

// Update price (requires owner permission)
async function updatePrice(newPrice, privateKey) {
  try {
    // Get account
    const account = web3.eth.accounts.privateKeyToAccount(privateKey);
    web3.eth.accounts.wallet.add(account);
    
    // Create transaction
    const tx = contract.methods.updatePrice(newPrice);
    const gas = await tx.estimateGas({ from: account.address });
    const gasPrice = await web3.eth.getGasPrice();
    
    // Send transaction
    const receipt = await tx.send({
      from: account.address,
      gas,
      gasPrice
    });
    
    console.log(`Price updated successfully, transaction hash: ${receipt.transactionHash}`);
    return receipt;
  } catch (error) {
    console.error('Error updating price:', error);
  }
}

// Get all information
async function getAllInfo() {
  console.log('===== Getting all contract information =====');
  await getCompletePrice();
  await getLatestPrice();
  await getLastUpdatedAt();
  await getSymbol();
  await getPricePrecision();
  await getOwner();
  console.log('===== Information retrieval complete =====');
}

// Execute function
getAllInfo();
```

#### Solidity Smart Contract Example

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
        
        // Note: Solidity doesn't support floating point, so this returns the raw price
        // In frontend applications, divide by precision value
        return price;
    }
    
    function getFormattedPrice() public view returns (string memory symbol, uint256 rawPrice, uint256 precision, uint256 timestamp) {
        (symbol, rawPrice, timestamp) = oracle.getLatestPrice();
        precision = oracle.pricePrecision();
        return (symbol, rawPrice, precision, timestamp);
    }
}
```

### Important Notes

1. **Price Precision**: The price returned by the contract needs to be divided by the value returned by the `pricePrecision()` method to get the actual price.
2. **Update Delay**: Price data is updated every 5 minutes. Check `lastUpdatedAt` to ensure you're using the latest data before making critical decisions.
3. **Testnet Usage**: The current contract is deployed on JuChain Testnet and is intended for development and testing purposes only.
4. **Price Volatility**: Cryptocurrency prices can experience significant volatility. Implement appropriate risk management mechanisms.
5. **Contract Upgrades**: The oracle contract may be updated. Follow official announcements for the latest contract addresses and feature updates.
