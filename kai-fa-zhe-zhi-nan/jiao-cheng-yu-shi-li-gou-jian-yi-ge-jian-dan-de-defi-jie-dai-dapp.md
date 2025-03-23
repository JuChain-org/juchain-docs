# 教程与示例：构建一个简单的 DeFi 借贷 dApp

本教程将指导你在 **JuChain** 上开发一个简单的去中心化金融（DeFi）借贷 dApp。用户可以存入 JU（JuChain 的原生代币）作为抵押，借出一定比例的 JU，偿还借款后取回抵押品。我们将使用 Solidity 编写智能合约，利用 JuChain 的 EVM 兼容性进行部署，并通过前端与用户交互。

此示例展示了 JuChain 的 **秒级交易确认** 和 **极低成本** 优势，非常适合 DeFi 应用场景。

***

### 教程：简单 DeFi 借贷 dApp

#### 前提条件

**开发环境**

* **Node.js**：v16 或更高版本
* **Truffle Suite**：通过 `npm install -g truffle` 安装
* **MetaMask**：浏览器扩展，配置为 JuChain 测试网
* **基础知识**：熟悉 Solidity 和 Web3.js

**JuChain 测试网**

* **测试网 RPC**：`https://testnet-rpc.juchain.org`
* **网络 ID**：`202599`（假设）
* **测试 JU**：通过 JuChain 测试网水龙头获取

***

#### 步骤 1：设置项目

1.  **创建项目目录并初始化 Truffle**：

    ```bash
    mkdir JuChainLending
    cd JuChainLending
    truffle init
    ```
2.  **安装依赖**：

    ```bash
    npm install @openzeppelin/contracts web3
    ```

***

#### 步骤 2：编写借贷智能合约

在 `contracts/` 文件夹中创建 `LendingPool.sol`：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract LendingPool is ReentrancyGuard {
    mapping(address => uint256) public collateral; // 用户抵押的 JU 数量
    mapping(address => uint256) public borrowed;   // 用户借出的 JU 数量
    uint256 public constant LTV = 50;              // 贷款价值比（Loan-to-Value），50%
    uint256 public constant INTEREST_RATE = 5;     // 年化利率 5%（简化为每次还款收取）

    event Deposited(address indexed user, uint256 amount);
    event Borrowed(address indexed user, uint256 amount);
    event Repaid(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    constructor() {
        // 无需参数，JU 是原生代币
    }

    // 存入 JU 作为抵押
    function depositCollateral() external payable nonReentrant {
        require(msg.value > 0, "Amount must be greater than 0");
        collateral[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    // 借出 JU，基于抵押的 50% LTV
    function borrow(uint256 amount) external nonReentrant {
        require(amount > 0, "Amount must be greater than 0");
        uint256 maxBorrow = (collateral[msg.sender] * LTV) / 100;
        require(borrowed[msg.sender] + amount <= maxBorrow, "Exceeds max borrowable amount");
        borrowed[msg.sender] += amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Borrowed(msg.sender, amount);
    }

    // 偿还借款（含利息）
    function repay(uint256 amount) external payable nonReentrant {
        require(amount > 0, "Amount must be greater than 0");
        require(borrowed[msg.sender] >= amount, "Repay exceeds borrowed amount");
        uint256 interest = (amount * INTEREST_RATE) / 100;
        uint256 totalRepay = amount + interest;
        require(msg.value >= totalRepay, "Insufficient payment");
        borrowed[msg.sender] -= amount;
        emit Repaid(msg.sender, amount);

        // 退回多余的 JU
        if (msg.value > totalRepay) {
            (bool success, ) = msg.sender.call{value: msg.value - totalRepay}("");
            require(success, "Refund failed");
        }
    }

    // 提取抵押（需先还清借款）
    function withdrawCollateral(uint256 amount) external nonReentrant {
        require(amount > 0, "Amount must be greater than 0");
        require(collateral[msg.sender] >= amount, "Insufficient collateral");
        require(borrowed[msg.sender] == 0, "Must repay all loans first");
        collateral[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }

    // 查看用户状态
    function getUserStatus(address user) external view returns (uint256, uint256, uint256) {
        uint256 maxBorrow = (collateral[user] * LTV) / 100;
        return (collateral[user], borrowed[user], maxBorrow);
    }
}
```

**功能说明**

* **`depositCollateral()`**：用户发送 JU（原生代币）作为抵押。
* **`borrow(uint256 amount)`**：根据 50% LTV 从合约借出 JU。
* **`repay(uint256 amount)`**：偿还借款并支付 5% 利息，多余的 JU 会退回。
* **`withdrawCollateral(uint256 amount)`**：还清借款后提取抵押的 JU。
* **`getUserStatus(address user)`**：查看用户的抵押、借款和最大可借额度。
* **安全性**：使用 `ReentrancyGuard` 防止重入攻击。

**注意**

JU 是 JuChain 的原生代币，不依赖 ERC20 标准，直接通过 `msg.value` 和 `call` 处理。

***

#### 步骤 3：配置 Truffle

编辑 `truffle-config.js`，连接 JuChain 测试网：

```javascript
const HDWalletProvider = require("@truffle/hdwallet-provider");
const mnemonic = "你的 MetaMask 助记词"; // 替换为你的助记词

module.exports = {
  networks: {
    juchain_testnet: {
      provider: () => new HDWalletProvider(mnemonic, "https://testnet-rpc.juchain.org"),
      network_id: 202599,
      gas: 5500000,
      gasPrice: 1000000000, // 1 gwei
    },
  },
  compilers: {
    solc: {
      version: "0.8.0",
    },
  },
};
```

***

#### 步骤 4：部署合约

1.  **创建部署脚本**：\
    在 `migrations/` 文件夹中添加 `2_deploy_lending.js`：

    ```javascript
    const LendingPool = artifacts.require("LendingPool");

    module.exports = function (deployer) {
      deployer.deploy(LendingPool);
    };
    ```
2.  **部署到 JuChain 测试网**：

    ```bash
    truffle migrate --network juchain_testnet
    ```

    记录部署后的 `LendingPool` 合约地址。

***

#### 步骤 5：测试合约

使用 Truffle 控制台验证功能：

1.  **进入控制台**：

    ```bash
    truffle console --network juchain_testnet
    ```
2.  **测试代码**：

    ```javascript
    let lending = await LendingPool.deployed();
    let accounts = await web3.eth.getAccounts();

    // 存入 100 JU 作为抵押
    await lending.depositCollateral({ from: accounts[0], value: web3.utils.toWei("100") });
    console.log((await lending.collateral(accounts[0])).toString()); // 输出 "100000000000000000000"

    // 借出 50 JU
    await lending.borrow(web3.utils.toWei("50"), { from: accounts[0] });
    console.log((await lending.borrowed(accounts[0])).toString()); // 输出 "50000000000000000000"

    // 查看用户状态
    let status = await lending.getUserStatus(accounts[0]);
    console.log(status.map(x => web3.utils.fromWei(x))); // 输出 [100, 50, 50]

    // 偿还 50 JU（含 5% 利息，共 52.5 JU）
    await lending.repay(web3.utils.toWei("50"), { from: accounts[0], value: web3.utils.toWei("52.5") });
    console.log((await lending.borrowed(accounts[0])).toString()); // 输出 "0"

    // 提取 100 JU
    await lending.withdrawCollateral(web3.utils.toWei("100"), { from: accounts[0] });
    console.log((await lending.collateral(accounts[0])).toString()); // 输出 "0"
    ```

***

#### 步骤 6：创建前端界面

在项目根目录创建 `index.html`：

```html
<!DOCTYPE html>
<html>
<head>
  <title>JuChain Lending dApp</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
    button { padding: 10px 20px; margin: 5px; }
    input { padding: 5px; margin: 5px; }
  </style>
</head>
<body>
  <h1>JuChain Lending Pool</h1>
  <p>Collateral: <span id="collateral">0</span> JU</p>
  <p>Borrowed: <span id="borrowed">0</span> JU</p>
  <p>Max Borrowable: <span id="maxBorrow">0</span> JU</p>
  <input id="amount" type="number" placeholder="Amount in JU" />
  <br />
  <button onclick="deposit()">Deposit Collateral</button>
  <button onclick="borrow()">Borrow JU</button>
  <button onclick="repay()">Repay Loan</button>
  <button onclick="withdraw()">Withdraw Collateral</button>

  <script src="https://cdn.jsdelivr.net/npm/web3@1.5.3/dist/web3.min.js"></script>
  <script>
    const contractAddress = "你的 LendingPool 合约地址"; // 替换为实际地址
    const abi = [ /* 从 build/contracts/LendingPool.json 获取 ABI */ ]; // 替换为完整 ABI
    let web3, lending;

    async function init() {
      if (window.ethereum) {
        web3 = new Web3(window.ethereum);
        await window.ethereum.enable();
        lending = new web3.eth.Contract(abi, contractAddress);
        updateStatus();
      } else {
        alert("Please install MetaMask!");
      }
    }

    async function updateStatus() {
      const accounts = await web3.eth.getAccounts();
      const status = await lending.methods.getUserStatus(accounts[0]).call();
      document.getElementById("collateral").innerText = web3.utils.fromWei(status[0]);
      document.getElementById("borrowed").innerText = web3.utils.fromWei(status[1]);
      document.getElementById("maxBorrow").innerText = web3.utils.fromWei(status[2]);
    }

    async function deposit() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      await lending.methods.depositCollateral().send({ from: accounts[0], value: amount });
      updateStatus();
    }

    async function borrow() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      await lending.methods.borrow(amount).send({ from: accounts[0] });
      updateStatus();
    }

    async function repay() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      const interest = (BigInt(amount) * 5n) / 100n;
      const total = (BigInt(amount) + interest).toString();
      await lending.methods.repay(amount).send({ from: accounts[0], value: total });
      updateStatus();
    }

    async function withdraw() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      await lending.methods.withdrawCollateral(amount).send({ from: accounts[0] });
      updateStatus();
    }

    init();
  </script>
</body>
</html>
```

**配置说明**

* **ABI**：从 `build/contracts/LendingPool.json` 中复制完整的 ABI 替换到 `abi` 变量。
* **合约地址**：将 `contractAddress` 替换为步骤 4 中部署的地址。
* **运行**：使用本地服务器（如 `npx http-server`）打开 `index.html`，通过 MetaMask 连接 JuChain 测试网。

***

#### 步骤 7：运行和测试

1. 在 MetaMask 中切换到 JuChain 测试网，并确保有足够的测试 JU。
2. 打开浏览器中的 `index.html`，依次测试：
   * **存入抵押**：输入 100 JU，点击 "Deposit Collateral"。
   * **借出 JU**：输入 50 JU，点击 "Borrow JU"。
   * **偿还借款**：输入 50 JU，点击 "Repay Loan"（自动包含 5% 利息，共 52.5 JU）。
   * **提取抵押**：输入 100 JU，点击 "Withdraw Collateral"。
3. JuChain 的秒级确认将确保交易快速完成，交易成本极低（通常低于 0.001 美元）。

***

#### 成果

你已成功构建并部署了一个简单的 DeFi 借贷 dApp：

* 用户可以存入 JU 作为抵押。
* 根据 50% LTV 借出 JU。
* 偿还借款并支付 5% 利息后取回抵押品。
* 前端界面直观显示用户的抵押、借款和可借额度。

此 dApp 展示了 JuChain 的高性能金融基础设施，适合扩展到更复杂的 DeFi 应用（如多资产支持或动态利率）。

***

#### 注意事项

1. **安全性**：
   * 实际生产中需添加清算机制（防止抵押价值不足）并进行代码审计。
   * 当前合约未处理极端情况（如 JU 余额不足）。
2. **测试网**：
   * 确保账户中有足够的测试 JU 用于交互。
3. **扩展性**：
   * 可添加时间相关的利息计算或支持其他资产。

