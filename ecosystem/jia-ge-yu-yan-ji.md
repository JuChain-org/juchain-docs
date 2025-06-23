---
hidden: true
---

# 价格预言机

## 价格预言机

### JU-USDT 价格预言机

JuChain 预言机服务为去中心化应用（dApps）提供可靠的链上价格数据。JU-USDT 价格预言机专门提供 JU 代币相对于 USDT 的实时价格，支持 DeFi 应用、交易平台和其他需要精准价格数据的智能合约。

{% hint style="info" %}
当前公链 Solidity 版本编译仅支持 <= 0.8.8，后续版本支持将根据官方通知逐步开放。
{% endhint %}

#### 合约信息

**主网合约信息**

* **网络**：JuChain 主网
* **合约地址**：`0x49E5c7f25711abe668F404307b27f4bE4836B0e7`
* **部署地址**：`0x7389F1B4717F5152B6Cc107bce4A42a11dC0b76E`
* **Owner**：`0x5021A15FaAFEFEC1daCB1c8b24FFE3F3E3f7277b`
* **更新权限地址**：`0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D`
* **部署交易哈希**：`0x233654f76766fb0f9fd1377a573bed11c60a44c0cdc8e59340ffda333d191140`
* **更新频率**：每 1 分钟
* **价格来源**：聚合多个中心化和去中心化交易所的 JU-USDT 交易对数据

**测试网合约信息**

* **网络**：JuChain 测试网
* **合约地址**：`0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95`
* **部署地址**：`0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0`
* **更新权限地址**：`0x4878683a8C3007258278824228a92aC4E072F050`（仅此地址有权更新价格）
* **部署交易哈希**：`0x7e42454909c3ea9b52af4af84217149a87aa71aa08f129e11a01d5cea0989659`
* **更新频率**：每 1 分钟
* **价格来源**：聚合多个中心化和去中心化交易所的 JU-USDT 交易对数据

**权限说明**

* **部署者（Owner）**：
  * 主网：`0x5021A15FaAFEFEC1daCB1c8b24FFE3F3E3f7277b`，负责合约管理和权限分配。
  * 测试网：`0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0`，负责合约管理和权限分配。
* **授权更新者（Authorized Updater）**：
  * 主网：`0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D`，唯一有权调用 `updatePrice` 更新价格的账户。
  * 测试网：`0x4878683a8C3007258278824228a92aC4E072F050`，唯一有权调用 `updatePrice` 更新价格的账户。
* **设计变更**：新版预言机将部署者和价格更新者分离，提升安全性。

#### 主要方法

**`getLatestPrice`**

* **描述**：获取 JU-USDT 交易对的最新价格信息。
* **返回值**：
  * `string`：交易对符号（如 `"JU/USDT"`）。
  * `uint256`：最新价格（需除以精度值）。
  * `uint256`：最后更新时间戳（Unix 时间戳）。
*   **调用示例**：

    ```solidity
    (string memory symbol, uint256 price, uint256 timestamp) = oracle.getLatestPrice();
    ```

**`latestPrice`**

* **描述**：获取最新价格值（不含额外信息）。
* **返回值**：
  * `uint256`：最新价格值。

**`pricePrecision`**

* **描述**：获取价格精度值，用于计算实际价格。
* **返回值**：
  * `uint256`：价格精度（如 `1e8`，实际价格 = 返回价格 / 精度）。

**`lastUpdatedAt`**

* **描述**：获取价格最后更新的时间戳。
* **返回值**：
  * `uint256`：Unix 时间戳。

**`symbol`**

* **描述**：获取交易对符号。
* **返回值**：
  * `string`：交易对符号（如 `"JU/USDT"`）。

**`updatePrice`**

* **描述**：更新 JU-USDT 价格，仅限授权更新者调用。
* **参数**：
  * `uint256 _price`：新价格值。
* **权限**：仅 `0x4878683a8C3007258278824228a92aC4E072F050` 可调用。

**`setAuthorizedUpdater`**

* **描述**：设置新的授权更新者，仅限 Owner 调用。
* **参数**：
  * `address newAuthorizedUpdater`：新授权更新者地址。
* **权限**：仅 `0xa01d5Be3fDea4Fd8f1C35Ced0919353036De15d0` 可调用。

**`owner`**

* **描述**：获取当前合约拥有者地址。
* **返回值**：
  * `address`：Owner 地址。

#### ABI

以下是 JU-USDT 价格预言机的完整 ABI：

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

#### 使用示例

**Web3.js 示例**

以下代码展示如何通过 Web3.js 与预言机交互：

```javascript
const { Web3 } = require('web3');

// 主网 RPC
const mainnetRpc = 'https://rpc.juchain.org';
// 测试网 RPC
const testnetRpc = 'https://testnet-rpc.juchain.org';

// 选择网络（主网/测试网）
const web3 = new Web3(mainnetRpc); // 或 testnetRpc

// 主网合约地址
const mainnetContractAddress = '0x49E5c7f25711abe668F404307b27f4bE4836B0e7';
// 测试网合约地址
const testnetContractAddress = '0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95';

// 选择合约地址
const contractAddress = mainnetContractAddress; // 或 testnetContractAddress
const contract = new web3.eth.Contract(abi, contractAddress);

// 获取完整价格信息
async function getCompletePrice() {
  try {
    const [symbol, price, timestamp] = await contract.methods.getLatestPrice().call();
    const precision = await contract.methods.pricePrecision().call();
    console.log('完整价格信息:');
    console.log(`交易对: ${symbol}`);
    console.log(`价格: ${price / precision} JU/USDT`);
    console.log(`时间戳: ${timestamp} (${new Date(Number(timestamp) * 1000).toLocaleString()})`);
    return { symbol, price, timestamp, precision };
  } catch (error) {
    console.error('获取价格信息失败:', error);
  }
}

// 获取最新价格
async function getLatestPrice() {
  try {
    const price = await contract.methods.latestPrice().call();
    const precision = await contract.methods.pricePrecision().call();
    console.log(`最新价格: ${price / precision} JU/USDT`);
    return price;
  } catch (error) {
    console.error('获取最新价格失败:', error);
  }
}

// 获取授权更新者
async function getAuthorizedUpdater() {
  try {
    const updater = await contract.methods.authorizedUpdater().call();
    console.log(`授权更新者: ${updater}`);
    return updater;
  } catch (error) {
    console.error('获取授权更新者失败:', error);
  }
}

// 更新价格（仅限授权更新者）
async function updatePrice(newPrice, privateKey) {
  try {
    const account = web3.eth.accounts.privateKeyToAccount(privateKey);
    web3.eth.accounts.wallet.add(account);
    const tx = contract.methods.updatePrice(newPrice);
    const gas = await tx.estimateGas({ from: account.address });
    const receipt = await tx.send({ from: account.address, gas });
    console.log(`价格更新成功，交易哈希: ${receipt.transactionHash}`);
    return receipt;
  } catch (error) {
    console.error('更新价格失败:', error);
  }
}

// 执行所有查询
async function getAllInfo() {
  await getCompletePrice();
  await getLatestPrice();
  await getAuthorizedUpdater();
}

getAllInfo();
```

**Solidity 示例**

以下是使用预言机的智能合约示例：

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
        return price; // 前端需除以 precision
    }

    function getFormattedPrice() public view returns (string memory symbol, uint256 price, uint256 precision, uint256 timestamp) {
        (symbol, price, timestamp) = oracle.getLatestPrice();
        precision = oracle.pricePrecision();
        return (symbol, price, precision, timestamp);
    }
}
```

#### 重要注意事项

* **网络选择**：
  * 主网：用于生产环境，合约地址为 `0x49E5c7f25711abe668F404307b27f4bE4836B0e7`
  * 测试网：用于开发和测试，合约地址为 `0x70D3Fc0bcf1ffD64111FC0C708DA407d9732Ab95`
* **价格精度**：合约返回的 `price` 需除以 `pricePrecision()` 返回的值以获得实际价格（如 `1e8` 表示 8 位精度）。
* **更新权限**：
  * 主网：仅 `0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D` 可调用 `updatePrice`
  * 测试网：仅 `0x4878683a8C3007258278824228a92aC4E072F050` 可调用 `updatePrice`
  * Owner 可通过 `setAuthorizedUpdater` 更换更新者
* **更新延迟**：价格每 1 分钟更新一次，使用前请检查 `lastUpdatedAt` 确保数据新鲜度。
* **价格波动**：加密货币价格可能剧烈波动，建议在 dApp 中加入风险管理机制。
* **合约升级**：预言机合约可能更新，请关注官方公告以获取最新地址和功能。
* **主网注意事项**：
  * 主网环境下的操作将产生实际费用，请谨慎操作
  * 建议先在测试网充分测试后再在主网部署
