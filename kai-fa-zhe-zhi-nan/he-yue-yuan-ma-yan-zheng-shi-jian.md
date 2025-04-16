# 合约源码验证实践

对已部署的智能合约进行源码验证，是提升项目透明度与安全性的关键环节。验证后的合约允许任何人在区块链浏览器（如 JuScan）上审查其行为逻辑，并与之安全交互。本指南将引导您使用 Hardhat 框架完成 JuChain (及其他兼容链) 上的合约验证流程。

***

### 1. 环境配置

确保您的 Hardhat 开发环境已就绪，特别是验证相关的配置。

#### 1.1 核心插件：`@nomicfoundation/hardhat-verify`

此插件负责处理与区块浏览器 API 的交互。通过 npm 或 yarn 将其添加到项目开发依赖中：

```bash
npm install --save-dev @nomicfoundation/hardhat-verify
# 或
# yarn add --dev @nomicfoundation/hardhat-verify
```

#### 1.2 `hardhat.config.js` 配置详解

合约验证的配置集中在 `hardhat.config.js` (或 `.ts`) 文件。

**a) 启用插件:**

在配置文件顶部引入该插件：

```javascript
require("@nomicfoundation/hardhat-verify");
```

**b) 网络定义 (`networks`):**

定义目标部署网络，确保包含 `url` (RPC 地址), `chainId` 和用于部署的 `accounts`。

```javascript
// .env 文件示例: PRIVATE_KEY=0x...
// 使用 dotenv 加载环境变量，如私钥
require("dotenv").config();

// ... 其他 require 语句 ...

networks: {
  JuChainTestnet: {
    url: "https://testnet-rpc.juchain.org",
    chainId: 202599,
    // 强烈建议从环境变量加载私钥，避免硬编码
    accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
  },
  JuChain: {
    url: "https://rpc.juchain.org",
    chainId: 210000,
    accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
  },
  // ... 其他网络如 mainnet, sepolia 等
},
```

* **关键:** `accounts` 列表中的私钥必须是执行合约部署操作的那个账户。

**c) 区块浏览器验证配置 (`etherscan`):**

此部分用于对接区块浏览器的验证服务。**即使目标是 Juscan，配置项也需置于 `etherscan` 字段下**。

```javascript
etherscan: {
  // 目前 Juscan 不需要 API Key，保持该值即可。
  // 如果未来 Juscan 更新策略需要 API Key，请查阅其官方文档获取。
  apiKey: {
    JuChainTestnet: "JUSCAN_NO_API_KEY_REQUIRED",
    JuChain: "JUSCAN_NO_API_KEY_REQUIRED",
  },
  // 自定义兼容 Etherscan API 的区块链配置
  customChains: [
    {
      network: "JuChainTestnet",
      chainId: 202599,
      urls: {
        apiURL: "https://testnet.juscan.io/api",
        browserURL: "https://testnet.juscan.io",
      },
    },
    {
      network: "JuChain", // 确保此 network 名称与 networks 中定义的键名一致
      chainId: 210000,
      urls: {
        // 请以 Juscan 官方文档为准确认最新的主网 API URL
        apiURL: "https://juscan.io/api",
        browserURL: "https://juscan.io",
      },
    },
    // ... 其他自定义链
  ],
},
```

* **`customChains`**: 配置非 Etherscan 原生支持的网络。`network` 字段必须严格对应 `networks` 中的键名。`apiURL` 是验证请求的目标地址。

**d) Sourcify (可选):**

按需配置 Sourcify 验证支持。

```javascript
sourcify: {
  enabled: false, // 如需使用，设为 true
},
```

***

### 2. 合约部署（前置条件）

验证操作的前提是合约已成功部署到目标网络。**以下为示例场景，请替换为您实际的合约和参数：**

* **示例 `ContractA` (无依赖):**
  * 部署地址 (示例): `0x60322a8476918646E297E0267F3444872cF5bA09`
  * 构造参数 (示例): `100`, `"0x...00a"` (假设为某个地址或bytes32)
* **示例 `ContractB` (依赖 `ContractA` 地址):**
  * 部署地址 (示例): `0x2C8B08E38fd8BaaED93F91929659b9aF2B65345E`
  * 构造参数 (示例): `0x60322a8476918646E297E0267F3444872cF5bA09` (ContractA 地址), `50`, `"0x...00b"`

您应通过 Hardhat 部署脚本准确记录下实际的部署地址和使用的构造函数参数。

***

### 3. 执行验证命令

使用 `npx hardhat verify` 命令发起验证。

#### 3.1 验证示例 `ContractA`

```bash
# 命令结构: npx hardhat verify --network <网络名> <合约地址> [构造函数参数...]
# 确保使用您实际的部署地址和构造函数参数
npx hardhat verify --network JuChainTestnet 0x60322a8476918646E297E0267F3444872cF5bA09 100 "0x000000000000000000000000000000000000000a"
```

* 参数必须严格按照部署时的顺序和类型提供。

#### 3.2 验证示例 `ContractB`

```bash
# 确保使用您实际的部署地址和构造函数参数
npx hardhat verify --network JuChainTestnet 0x2C8B08E38fd8BaaED93F91929659b9aF2B65345E "0x60322a8476918646E297E0267F3444872cF5bA09" 50 "0x000000000000000000000000000000000000000b"
```

* 地址参数作为字符串传入。

#### 3.3 检查结果

验证成功后，命令行会打印区块浏览器的链接。访问该链接，确认合约代码旁出现验证标记（如绿色勾号），且"Contract" / "Code" 选项卡下显示了匹配的 Solidity 源码。

***

### 4. 常见问题与排查

验证失败通常由以下原因导致，请逐一排查：

#### 4.1 构造函数参数不匹配

* **核对**: 这是最常见的问题。请仔细比对 `verify` 命令提供的参数与部署脚本或交易记录中实际使用的参数，确保类型、顺序、数值完全一致。
* **复杂参数**: 对于复杂的参数（如 struct 或数组），建议使用 `--constructor-args` 选项指定一个导出参数列表的 JS/TS 文件。例如 `arguments.js`: `module.exports = [arg1, arg2, ...];`，然后运行 `npx hardhat verify --constructor-args arguments.js ...`。

#### 4.2 网络配置错误

* 检查 `hardhat.config.js` 中 `--network` 指定的网络名 (`JuChainTestnet`) 是否存在于 `networks` 和 `etherscan.customChains` 中，且 `chainId` 正确。
* 确认 `customChains` 中的 `apiURL` 是区块浏览器官方提供的有效验证接口。

#### 4.3 合约地址或名称错误

* 确认提供的合约地址无误，并且是部署在该 `--network` (`JuChainTestnet`) 上的。
* 如果一个 Solidity 文件包含多个合约，Hardhat 通常能自动推断。若推断失败或需要明确指定，使用 `--contract` 标志：`npx hardhat verify --contract contracts/MyFile.sol:MySpecificContract ...`。

#### 4.4 编译器或优化器设置不一致

* 确认 `hardhat.config.js` 中的 `solidity` 版本和优化器设置 (`optimizer: { enabled: true/false, runs: ... }`) 与部署时使用的设置完全相同。Hardhat 验证时会默认使用配置文件中的设置。

#### 4.5 代理合约

* **分开验证**: 通常需要先验证实现合约 (Logic Contract)，再单独处理代理合约 (Proxy Contract)。
* **链接代理**: 对于透明代理或 UUPS 代理，验证实现合约后，可能还需要在区块浏览器界面上手动将代理地址链接到实现合约地址。检查您使用的代理模式和区块浏览器的具体操作。`@openzeppelin/hardhat-upgrades` 插件提供了针对 OpenZeppelin 代理的验证辅助功能。

#### 4.6 验证服务的延迟或临时故障

* 有时即使提交成功，区块浏览器也需要几分钟时间处理并更新状态。稍后再刷新页面查看。
* 检查区块浏览器自身的状态页或社区，确认验证服务是否正常运行。

***

### 5. 手动验证（可选）

除了使用 Hardhat 插件进行验证外，大多数区块浏览器（包括 Juscan）也支持直接通过其网站界面手动验证合约源码。如果您在使用 Hardhat 验证时遇到困难，或者偏好图形化操作，可以尝试此方法。

大致步骤如下：

1. 在 Juscan (或其他浏览器) 上访问您已部署的合约地址页面。
2. 寻找类似 "Verify & Publish Contract Source Code" 或 "验证并发布合约源码" 的选项或按钮。
3. 根据提示选择正确的 Solidity **编译器版本** 和 **优化器设置**（必须与您部署时使用的完全一致）。
4. 将您的合约 **源代码** 粘贴到提供的文本框中。如果合约导入了其他文件，您可能需要：
   * 将所有代码"扁平化"(flatten)到一个文件中再粘贴。
   * 或者，如果浏览器支持，分别上传主合约文件和所有依赖的库/接口文件。
5. 如果您的合约有 **构造函数参数**，需要按照浏览器要求的格式输入。
6. 提交验证请求，等待浏览器处理。

手动验证对于理解验证过程的细节很有帮助，但在项目迭代中不如 Hardhat 自动化验证高效。

