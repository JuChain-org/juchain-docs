# 快速开始

本指南将帮助您快速开始 JuChain 开发。

## 前置条件

在开始之前，请确保您具备：

1. Web3 钱包（MetaMask 或 OKX 钱包）
2. 基本的区块链开发知识
3. 开发环境配置（Node.js, Git）

## 开发步骤

### 1. 连接到 JuChain 网络

首先，将 JuChain 网络添加到您的钱包：

* 网络名称：JuChain Testnet
* RPC 地址：https://testnet-rpc.juchain.org
* 链 ID：66633666
* 货币符号：JU
* 区块浏览器：http://explorer-testnet.juchain.org/

### 2. 获取测试代币

1. 访问水龙头网站：http://faucet-testnet.juchain.org/
2. 输入您的钱包地址
3. 领取测试代币

### 3. 开始开发

#### 使用 Web3.js

```javascript
const Web3 = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// 检查连接
web3.eth.net.isListening()
   .then(() => console.log('已连接！'))
   .catch(err => console.error('连接失败'));
```

#### 使用 Ethers.js

```javascript
const { ethers } = require('ethers');
const provider = new ethers.providers.JsonRpcProvider('https://testnet-rpc.juchain.org');

// 检查连接
provider.getNetwork()
   .then(network => console.log('已连接到网络：', network.name))
   .catch(err => console.error('连接失败'));
```

### 4. 部署智能合约

1. 安装开发框架（例如 Hardhat）：
```bash
npm install --save-dev hardhat
```

2. 在 hardhat.config.js 中配置网络：
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

3. 部署您的合约：
```bash
npx hardhat run scripts/deploy.js --network juchain
```

## 下一步

- 探索我们的[开发者文档](../basics/smart-contracts.md)
- 加入我们的[开发者社区](https://discord.gg/juchain)
- 查看[示例项目](https://github.com/juchain/examples)

## 支持

如果您需要帮助：
1. 访问我们的[文档](https://docs.juchain.org)
2. 加入我们的 [Discord 社区](https://discord.gg/juchain)
3. 在 [GitHub](https://github.com/juchain/juchain) 上提交问题 