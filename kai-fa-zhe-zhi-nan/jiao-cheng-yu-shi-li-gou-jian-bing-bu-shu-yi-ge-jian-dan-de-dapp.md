# 教程与示例：构建并部署一个简单的 dApp

本教程将引导您在 JuChain 上创建一个简单的去中心化应用程序（dApp）：一个“问候”智能合约，用户可以设置并读取自定义问候语。我们将使用 Solidity 编写合约，通过 Truffle 部署到 JuChain 测试网，并创建一个前端界面与 MetaMask 交互。本教程旨在展示 JuChain 的开发流程、快速交易确认和低成本优势。

***

### 前提条件

* **工具**：
  * Node.js（v16 或更高版本，安装：`https://nodejs.org/`）
  * Truffle Suite（全局安装：`npm install -g truffle`）
  * MetaMask 浏览器扩展（安装：`https://metamask.io/`）
* **JuChain 测试网**：
  * RPC 端点：`https://testnet-rpc.juchain.org`
  * 网络 ID：202599
  * 测试 JU 代币：通过 JuChain 测试网水龙头获取（假设为 `https://faucet-testnet.juchain.org`）
* **基础知识**：了解 JavaScript、Solidity 和基本的区块链概念。

***

### 步骤 1：设置项目

1.  **创建项目目录并初始化 Truffle**：

    ```bash
    mkdir JuChainGreeting
    cd JuChainGreeting
    truffle init
    ```

    这将生成 `contracts/`、`migrations/` 和 `truffle-config.js` 等文件夹。
2.  **安装依赖**：

    ```bash
    npm install @openzeppelin/contracts web3 dotenv
    ```

    * `@openzeppelin/contracts`：提供安全合约模板。
    * `web3`：用于前端与区块链交互。
    * `dotenv`：管理环境变量。
3.  **创建 .env 文件**：\
    在项目根目录创建 `.env` 文件，用于存储敏感信息：

    ```plaintext
    MNEMONIC="你的 MetaMask 助记词"
    RPC_URL="https://testnet-rpc.juchain.org"
    ```

    > **安全提示**：不要将 `.env` 文件上传到 GitHub。

***

### 步骤 2：编写智能合约

在 `contracts/` 文件夹中创建 `Greeting.sol`：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Greeting {
    string public greeting;

    event GreetingSet(address indexed user, string newGreeting);

    constructor() {
        greeting = "Hello, JuChain!";
    }

    function setGreeting(string memory _greeting) public {
        require(bytes(_greeting).length > 0, "Greeting cannot be empty");
        greeting = _greeting;
        emit GreetingSet(msg.sender, _greeting);
    }

    function getGreeting() public view returns (string memory) {
        return greeting;
    }
}
```

**说明**

* **`greeting`**：存储问候语的公共变量。
* **`setGreeting`**：允许用户设置新问候语，触发事件。
* **`getGreeting`**：读取当前问候语。
* **事件**：记录每次问候语变更，便于前端监听。

***

### 步骤 3：配置 Truffle

编辑 `truffle-config.js`，添加 JuChain 测试网配置：

```javascript
require("dotenv").config();
const HDWalletProvider = require("@truffle/hdwallet-provider");

const { MNEMONIC, RPC_URL } = process.env;

module.exports = {
  networks: {
    juchain_testnet: {
      provider: () => new HDWalletProvider(MNEMONIC, RPC_URL),
      network_id: 202599, // JuChain 测试网 ID
      gas: 5500000,     // Gas 上限
      gasPrice: 1000000000, // 1 gwei
      confirmations: 2, // 等待 2 个区块确认
      timeoutBlocks: 200, // 超时区块数
    },
  },
  compilers: {
    solc: {
      version: "0.8.0", // Solidity 版本
      settings: {
        optimizer: {
          enabled: true,
          runs: 200,
        },
      },
    },
  },
};
```

**检查点**

* 确保 `.env` 文件中的 `MNEMONIC` 和 `RPC_URL` 已正确配置。
*   测试网络连接：

    ```bash
    truffle console --network juchain_testnet
    > web3.eth.getBlockNumber()
    ```

    如果返回区块高度，说明连接成功。

***

### 步骤 4：部署合约

1.  **创建部署脚本**：\
    在 `migrations/` 文件夹中创建 `2_deploy_greeting.js`：

    ```javascript
    const Greeting = artifacts.require("Greeting");

    module.exports = function (deployer) {
      deployer.deploy(Greeting);
    };
    ```
2.  **部署到 JuChain 测试网**：

    ```bash
    truffle migrate --network juchain_testnet
    ```

    *   输出示例：

        ```
        2_deploy_greeting.js
        ====================
        Deploying 'Greeting'
        --------------------
        > transaction hash: 0x...
        > contract address: 0x1234567890abcdef1234567890abcdef12345678
        ```
    * 记录合约地址（如 `0x1234...5678`），用于后续步骤。
3.  **验证部署**：\
    在 Truffle 控制台中：

    ```bash
    truffle console --network juchain_testnet
    > let greeting = await Greeting.at("0x1234567890abcdef1234567890abcdef12345678")
    > await greeting.getGreeting()
    ```

    输出应为 `"Hello, JuChain!"`。

***

### 步骤 5：创建前端界面

1.  **初始化前端文件夹**：

    ```bash
    mkdir public && cd public
    touch index.html
    ```
2.  **编写 index.html**：\
    在 `public/index.html` 中添加以下内容：

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>JuChain Greeting dApp</title>
      <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
        input { padding: 5px; margin: 10px; width: 200px; }
        button { padding: 10px 20px; margin: 5px; }
        #status { color: green; }
      </style>
    </head>
    <body>
      <h1>JuChain Greeting dApp</h1>
      <p>Current Greeting: <span id="greeting">Loading...</span></p>
      <input id="newGreeting" type="text" placeholder="Enter new greeting" />
      <br />
      <button onclick="setGreeting()">Set Greeting</button>
      <p id="status"></p>

      <script src="https://cdn.jsdelivr.net/npm/web3@1.5.3/dist/web3.min.js"></script>
      <script>
        const contractAddress = "0x1234567890abcdef1234567890abcdef12345678"; // 替换为你的合约地址
        const abi = [
          {
            "inputs": [],
            "stateMutability": "nonpayable",
            "type": "constructor"
          },
          {
            "anonymous": false,
            "inputs": [
              {"indexed": true, "internalType": "address", "name": "user", "type": "address"},
              {"indexed": false, "internalType": "string", "name": "newGreeting", "type": "string"}
            ],
            "name": "GreetingSet",
            "type": "event"
          },
          {
            "inputs": [],
            "name": "getGreeting",
            "outputs": [{"internalType": "string", "name": "", "type": "string"}],
            "stateMutability": "view",
            "type": "function"
          },
          {
            "inputs": [{"internalType": "string", "name": "_greeting", "type": "string"}],
            "name": "setGreeting",
            "outputs": [],
            "stateMutability": "nonpayable",
            "type": "function"
          },
          {
            "inputs": [],
            "name": "greeting",
            "outputs": [{"internalType": "string", "name": "", "type": "string"}],
            "stateMutability": "view",
            "type": "function"
          }
        ];
        let web3, contract;

        async function init() {
          if (window.ethereum) {
            web3 = new Web3(window.ethereum);
            try {
              await window.ethereum.request({ method: "eth_requestAccounts" });
              contract = new web3.eth.Contract(abi, contractAddress);
              updateGreeting();
            } catch (error) {
              console.error("User denied account access", error);
              document.getElementById("status").innerText = "Please connect MetaMask";
            }
          } else {
            alert("Please install MetaMask!");
          }
        }

        async function updateGreeting() {
          const greeting = await contract.methods.getGreeting().call();
          document.getElementById("greeting").innerText = greeting;
        }

        async function setGreeting() {
          const accounts = await web3.eth.getAccounts();
          const newGreeting = document.getElementById("newGreeting").value;
          if (!newGreeting) {
            document.getElementById("status").innerText = "Greeting cannot be empty!";
            return;
          }
          document.getElementById("status").innerText = "Setting greeting...";
          try {
            await contract.methods.setGreeting(newGreeting).send({ from: accounts[0] });
            document.getElementById("status").innerText = "Greeting updated successfully!";
            updateGreeting();
          } catch (error) {
            document.getElementById("status").innerText = "Error: " + error.message;
          }
        }

        window.onload = init;
      </script>
    </body>
    </html>
    ```
3. **获取 ABI**：
   * 从 `build/contracts/Greeting.json` 中复制 `"abi"` 字段，替换 `index.html` 中的 `abi` 变量内容。
   * 将 `contractAddress` 替换为步骤 4 中记录的实际合约地址。

***

### 步骤 6：运行和测试

1.  **启动本地服务器**：

    ```bash
    npx http-server ./public
    ```

    * 默认访问 `http://localhost:8080`。
2. **连接 MetaMask**：
   * 在 MetaMask 中添加 JuChain 测试网：
     * 网络名称：JuChain Testnet
     * RPC URL：`https://testnet-rpc.juchain.org`
     * 链 ID：202599
     * 货币符号：JU
   * 确保账户中有测试 JU 代币（从水龙头获取）。
3. **测试 dApp**：
   * 打开浏览器，访问 `http://localhost:8080`。
   * 检查初始问候语是否显示为 `"Hello, JuChain!"`。
   * 输入新问候语（如 `"Welcome to JuChain!"`），点击“Set Greeting”。
   * 确认状态更新为“Greeting updated successfully!”，页面显示新问候语。
   * 检查 MetaMask，交易应在 3-6 秒内确认，费用极低（约 0.001 JU）。

***

### 成果

您已成功构建并部署了一个简单的“问候”dApp：

* **智能合约**：部署在 JuChain 测试网，支持设置和读取问候语。
* **前端界面**：通过 MetaMask 与合约交互，直观显示结果。
* **JuChain 优势**：
  * **秒级确认**：交易快速完成，提升用户体验。
  * **低成本**：每次设置问候语的费用极低，适合频繁交互。
  * **EVM 兼容**：使用熟悉的以太坊工具（Truffle、Web3.js）无缝开发。

***

### 注意事项

* **网络状态**：确保 JuChain 测试网 RPC 可用，若不可用，联系 JuChain 支持。
* **Gas 费用**：测试网费用低，但在主网部署前需估算实际成本。

###
