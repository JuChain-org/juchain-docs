# Oracle

## JU-USDT 价格

### 概述

JuChain 预言机服务提供链上可靠的价格数据，为去中心化应用提供关键的市场信息。JU-USDT 价格预言机专门提供 JU 代币相对于 USDT 的实时价格数据，支持 DeFi 应用、交易平台和其他需要准确价格信息的智能合约。

### 合约信息

* **合约地址**: `0xC00E9C9a2ec48e6032255DF4975EfC63c6995881`
* **网络**: JuChain 测试网
* **更新频率**: 每 1 分钟更新一次
* **价格来源**: 聚合多个中心化和去中心化交易所的 JU-USDT 交易对数据

### 主要方法

#### getLatestPrice

获取 JU-USDT 交易对的最新价格信息。

**返回值**:

* `string`: 交易对符号 (例如 "JU/USDT")
* `uint256`: 最新价格 (需要除以精度值来获取实际价格)
* `uint256`: 最后更新时间戳 (Unix 时间戳格式)

**示例调用**:

```solidity
(string memory symbol, uint256 price, uint256 timestamp) = oracle.getLatestPrice();
```

#### latestPrice

获取最新价格值（不包含其他信息）。

**返回值**:

* `uint256`: 最新价格值

#### pricePrecision

获取价格精度值，用于计算实际价格。

**返回值**:

* `uint256`: 价格精度 (例如，如果精度为 1e8，则实际价格 = 返回价格 / 1e8)

#### lastUpdatedAt

获取价格最后更新的时间戳。

**返回值**:

* `uint256`: 最后更新时间的 Unix 时间戳

#### symbol

获取预言机跟踪的交易对符号。

**返回值**:

* `string`: 交易对符号 (例如 "JU/USDT")

### 使用示例

#### Web3.js 示例

```javascript
const Web3 = require('web3');
const web3 = new Web3('https://testnet.juchain.org');

const oracleAddress = '0xC00E9C9a2ec48e6032255DF4975EfC63c6995881';
const oracleABI = [...]; // 使用上面提供的完整 ABI

async function getJUPrice() {
  const oracleContract = new web3.eth.Contract(oracleABI, oracleAddress);
  
  try {
    const result = await oracleContract.methods.getLatestPrice().call();
    const symbol = result[0];
    const rawPrice = result[1];
    const timestamp = result[2];
    
    // 获取精度
    const precision = await oracleContract.methods.pricePrecision().call();
    
    // 计算实际价格
    const actualPrice = rawPrice / precision;
    
    // 格式化时间
    const updateTime = new Date(timestamp * 1000).toLocaleString();
    
    console.log(`${symbol} 价格: $${actualPrice}`);
    console.log(`最后更新时间: ${updateTime}`);
    
    return actualPrice;
  } catch (error) {
    console.error('获取价格失败:', error);
    return null;
  }
}
```

#### Solidity 智能合约示例

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
        
        // 注意：Solidity 不支持浮点数，所以这里返回的是原始价格
        // 在前端应用中需要除以精度值
        return price;
    }
    
    function getFormattedPrice() public view returns (string memory symbol, uint256 rawPrice, uint256 precision, uint256 timestamp) {
        (symbol, rawPrice, timestamp) = oracle.getLatestPrice();
        precision = oracle.pricePrecision();
        return (symbol, rawPrice, precision, timestamp);
    }
}
```

### 注意事项

1. **价格精度**: 合约返回的价格需要除以 `pricePrecision()` 方法返回的值才能获得实际价格。
2. **更新延迟**: 价格数据每 5 分钟更新一次，在做出关键决策前请检查 `lastUpdatedAt` 确保使用的是最新数据。
3. **测试网使用**: 当前合约部署在 JuChain 测试网上，仅用于开发和测试目的。
4. **价格波动**: 加密货币价格可能出现剧烈波动，建议实现适当的风险管理机制。
5. **合约升级**: 预言机合约可能会更新，请关注官方公告获取最新合约地址和功能更新。

### 技术支持

如有任何关于 JU-USDT 预言机的问题或需要技术支持，请通过官网给出的渠道联系我们。
