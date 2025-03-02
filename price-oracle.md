---
description: Price
---

# Price Oracle

## JU-USDT Price Oracle Documentation

### Overview

JuChain Oracle Service provides reliable on-chain price data for decentralized applications. The JU-USDT price oracle specifically offers real-time price data for the JU token relative to USDT, supporting DeFi applications, trading platforms, and other smart contracts requiring accurate price information.

### Contract Information

* **Contract Address**: `0xC00E9C9a2ec48e6032255DF4975EfC63c6995881`
* **Network**: JuChain Testnet
* **Update Frequency**: Every 5 minutes
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
const Web3 = require('web3');
const web3 = new Web3('https://testnet.juchain.org');

const oracleAddress = '0xC00E9C9a2ec48e6032255DF4975EfC63c6995881';
const oracleABI = [...]; // Use the complete ABI provided above

async function getJUPrice() {
  const oracleContract = new web3.eth.Contract(oracleABI, oracleAddress);
  
  try {
    const result = await oracleContract.methods.getLatestPrice().call();
    const symbol = result[0];
    const rawPrice = result[1];
    const timestamp = result[2];
    
    // Get precision
    const precision = await oracleContract.methods.pricePrecision().call();
    
    // Calculate actual price
    const actualPrice = rawPrice / precision;
    
    // Format time
    const updateTime = new Date(timestamp * 1000).toLocaleString();
    
    console.log(`${symbol} Price: $${actualPrice}`);
    console.log(`Last Updated: ${updateTime}`);
    
    return actualPrice;
  } catch (error) {
    console.error('Failed to get price:', error);
    return null;
  }
}
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

### Technical Support

For any questions about the JU-USDT oracle or technical support, please contact us through:

* Developer Forum: [forum.juchain.org](https://forum.juchain.org)
* Technical Support Email: dev@juchain.org
