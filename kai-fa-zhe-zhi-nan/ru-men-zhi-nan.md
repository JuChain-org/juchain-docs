# 入门指南

## JuChain 开发者文档 - 入门指南



{% hint style="warning" %}
JuChain 当前仅支持 Solidity 版本 <= 0.8.8 进行编译，后续版本支持将根据官方通知逐步开放。
{% endhint %}

欢迎体验 JuChain 开发者之旅！本指南将帮助您快速上手 JuChain 平台，从设置开发环境到部署您的首个智能合约，再到连接 JuChain 测试网。本章节面向初次接触 JuChain 的开发者，旨在提供清晰的步骤和实用示例，确保您能迅速开始构建。

### 前提条件

在开始之前，请确保您具备以下基础知识和工具：

* **基础区块链知识**：熟悉智能合约、区块链交易和以太坊虚拟机（EVM）的基本概念。
* **开发工具**：
  * **Node.js**（建议 v16 或更高版本）：用于运行 JavaScript 和安装依赖。
  * **npm** 或 **yarn**：包管理工具，用于安装项目依赖。
  * **代码编辑器**：推荐 Visual Studio Code 或类似工具。
* **钱包**：安装 MetaMask 或其他兼容 EVM 的钱包，用于管理账户和测试网代币。
* **终端技能**：基本的命令行操作能力。

如果您已经熟悉以太坊开发，JuChain 的完全 EVM 兼容性将让您的迁移过程非常顺畅。

### 第一步：设置开发环境

#### 安装必备工具

1. **安装 Node.js 和 npm**
   * 下载并安装最新版本的 Node.js（包含 npm）：[Node.js 官网](https://nodejs.org/)。
   *   验证安装：

       ```bash
       node -v
       npm -v
       ```
   * 如果您更喜欢 yarn，可使用 `npm install -g yarn` 安装。
2. **安装 Truffle 或 Hardhat**\
   JuChain 支持以太坊主流开发框架。这里以 Hardhat 为例：
   *   全局安装 Hardhat：

       ```bash
       npm install -g hardhat
       ```
   *   创建一个新 Hardhat 项目：

       ```bash
       npx hardhat
       ```

       选择“Create a basic sample project”并按提示完成初始化。
3. **安装 MetaMask**
   * 下载并安装 MetaMask 浏览器扩展：[MetaMask 官网](https://metamask.io/)。
   * 创建或导入一个钱包账户，用于后续测试网交互。

#### 配置 JuChain 网络

JuChain 提供主网和测试网供开发者使用。以下是测试网的配置信息：

* **网络名称**：JuChain Testnet
* **RPC URL**：`https://testnet-rpc.juchain.org`
* **链 ID**：
* **货币符号**：JU
* **区块浏览器 URL**：`https://explorer-testnet.juchain.org`

在 MetaMask 中添加 JuChain 测试网：

1. 打开 MetaMask，点击“网络”下拉菜单。
2. 选择“添加网络” > “手动添加网络”。
3. 输入上述配置信息并保存。

### 第二步：获取测试网 JU 代币

在 JuChain 测试网上部署和测试智能合约需要 JU 代币。您可以通过官方水龙头获取测试网 JU。

1. 访问 JuChain 测试网水龙头：[https://faucet-testnet.juchain.org](https://faucet-testnet.juchain.org)。
2. 输入您的 MetaMask 钱包地址。
3. 点击“领取”按钮，每日可领取一定数量的测试 JU（例如 10 JU）。
4. 在 MetaMask 中确认您的测试网余额已更新。

### 第三步：编写您的首个智能合约

以下是一个简单的智能合约示例，用于在 JuChain 上存储和读取数据。

#### 创建智能合约

1. 在 Hardhat 项目目录下，进入 `contracts` 文件夹。
2.  创建文件 `SimpleStorage.sol`，并输入以下代码：

    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract SimpleStorage {
        uint256 private storedValue;

        // 存储一个值
        function set(uint256 _value) public {
            storedValue = _value;
        }

        // 获取存储的值
        function get() public view returns (uint256) {
            return storedValue;
        }
    }
    ```

#### 配置 Hardhat

1.  打开 `hardhat.config.js`，添加 JuChain 测试网配置：

    ```javascript
    require("@nomicfoundation/hardhat-toolbox");

    module.exports = {
      solidity: "0.8.0",
      networks: {
        juchain_testnet: {
          url: "https://testnet-rpc.juchain.org",
          chainId: 
          accounts: ["YOUR_PRIVATE_KEY"] // 替换为您的私钥（注意安全）
        }
      }
    };
    ```

    * 将 `YOUR_PRIVATE_KEY` 替换为您的 MetaMask 私钥（从“账户详情”中导出，注意不要公开分享）。
2.  确保安装必要的 Hardhat 插件：

    ```bash
    npm install --save-dev @nomicfoundation/hardhat-toolbox
    ```

#### 编译智能合约

在终端运行以下命令编译合约：

```bash
npx hardhat compile
```

成功后，您将在 `artifacts` 文件夹中看到编译结果。

### 第四步：部署到 JuChain 测试网

#### 编写部署脚本

1.  在 `scripts` 文件夹中，创建 `deploy.js`：

    ```javascript
    const hre = require("hardhat");

    async function main() {
      const SimpleStorage = await hre.ethers.getContractFactory("SimpleStorage");
      const simpleStorage = await SimpleStorage.deploy();

      await simpleStorage.deployed();
      console.log("SimpleStorage deployed to:", simpleStorage.address);
    }

    main()
      .then(() => process.exit(0))
      .catch((error) => {
        console.error(error);
        process.exit(1);
      });
    ```
2.  部署合约到 JuChain 测试网：

    ```bash
    npx hardhat run scripts/deploy.js --network juchain_testnet
    ```

    *   部署成功后，终端将输出合约地址，例如：

        ```
        SimpleStorage deployed to: 0x1234...abcd
        ```
3. 在 JuChain 测试网区块浏览器（`https://testnet-explorer.juchain.org`）上输入合约地址，验证部署是否成功。

### 第五步：与智能合约交互

#### 测试合约功能

1.  在 Hardhat 控制台中测试：

    ```bash
    npx hardhat console --network juchain_testnet
    ```
2.  输入以下命令与合约交互：

    ```javascript
    const SimpleStorage = await ethers.getContractFactory("SimpleStorage");
    const simpleStorage = await SimpleStorage.attach("0x1234...abcd"); // 替换为您的合约地址
    await simpleStorage.set(42);
    (await simpleStorage.get()).toString();
    ```

    * 输出应为 `"42"`，表示合约正常工作。

#### 使用 ethers.js 前端交互（可选）

如果您想通过网页与合约交互：

1.  初始化一个前端项目：

    ```bash
    npm init -y
    npm install ethers
    ```
2.  创建 `index.html` 和 `app.js`，示例代码如下：

    ```javascript
    import { ethers } from "ethers";

    const provider = new ethers.providers.Web3Provider(window.ethereum);
    await provider.send("eth_requestAccounts", []);
    const signer = provider.getSigner();
    const contractAddress = "0x1234...abcd"; // 替换为您的合约地址
    const abi = [ /* 从 artifacts/SimpleStorage.json 中复制 ABI */ ];
    const contract = new ethers.Contract(contractAddress, abi, signer);

    // 设置值
    await contract.set(100);
    // 获取值
    const value = await contract.get();
    console.log("Stored value:", value.toString());
    ```

### 第六步：连接 JuChain 测试网节点

JuChain 提供免费的公共 RPC 节点，开发者无需自己运行节点即可开发和测试。

#### 使用公共 RPC

* **测试网 RPC**：`https://testnet-rpc.juchain.org`
* 在 Hardhat 或其他工具中直接使用此 RPC URL 连接网络。

#### 运行自己的节点

* 暂未开放。

### 下一步

恭喜您已成功部署首个智能合约到 JuChain 测试网！接下来，您可以：

* 探索 开发者指南 以深入了解 API 和流量金融集成。
* 查看 教程与示例 以构建更复杂的 dApps。
* 加入 JuChain 开发者社区：Discord 或 论坛 获取支持。

如果遇到问题，请参考 FAQ 或联系技术支持团队。

***
