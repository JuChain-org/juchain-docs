# 去中心化交易所

JuChain 提供了一套基于 Uniswap V2 模型的去中心化交易所（DEX）功能，允许用户进行代币兑换（Swap）和提供流动性（Liquidity）。

**核心组件:**

* **路由器 (`JUV2Router02`)**: 用户交互的主要入口合约，用于执行 Swap、添加/移除流动性。与 Uniswap V2 Router 类似。
* **工厂 (`JUV2Factory`)**: 创建和管理交易对合约。
* **交易对 (`JUV2Pair`)**: 代表一个特定的代币交易对（例如 WJU-USDT），持有流动性资金池，并作为 ERC20 LP 代币。
* **WJU (Wrapped JU)**: 原生 JU 代币的 ERC20 封装版本，用于与 DEX 交互。
* **USDT**: 网络上的一个示例 ERC20 代币。

#### 1. 前置准备与网络设置

* **网络**: JuChain 测试网
  * **RPC URL**: `https://testnet-rpc.juchain.org`
  * **链 ID (Chain ID)**: `202599`
  * **区块浏览器**: `https://testnet.juscan.io`
* **工具**:
  * Node.js 环境及 Web3.js 库 (`const { Web3 } = require('web3');`) 或类似库 (ethers.js)。
  * Solidity 开发环境 (可选，用于合约交互或理解)。
* **账户与凭证**:
  * 一个外部账户 (EOA) 地址 (`USER_ADDRESS`)。
  * 该账户的私钥 (`PRIVATE_KEY`)，**必须安全存储和管理**，切勿硬编码或公开。
  * 账户需持有原生 JU (用于支付 Gas) 和交互所需的代币 (如 WJU, USDT)。
* **基础设置**:
  * 初始化 Web3 实例: `const web3 = new Web3(RPC_URL);`
  * 创建合约实例: 使用相应的 ABI (见第 7 节) 和合约地址为 WJU, USDT, Router, Factory 创建 Web3 Contract 对象。Pair 合约实例通常在获取其地址后动态创建。

#### 2. 核心合约地址与详情

| 合约               | 地址                                                                   | 说明                                                        |
| ---------------- | -------------------------------------------------------------------- | --------------------------------------------------------- |
| **WJU**          | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`                         | 封装版 JU。`deposit()` (payable) 封装, `withdraw()` 解封装。18 位小数。 |
| **USDT**         | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`                         | 标准 ERC20 代币。通常为 18 位小数 (请以链上 `decimals()` 确认)。            |
| **工厂 (Factory)** | `0x66682281BdfeC17fCBcae0480C77edFb0c489339`                         | 部署并追踪 `JUV2Pair` 交易对合约。                                   |
| **路由器 (Router)** | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0`                         | **交互主入口**。内部将 WJU 地址视为其 `WETH` 地址 (`router.WETH()` 可验证)。  |
| **交易对初始化哈希**     | `0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b` | 用于确定性计算交易对地址 (配合 `create2`)。                              |

**关键配置:** 路由器合约 (`JUV2Router02`) 中的 `WETH` 地址**必须**指向 JuChain 上的 WJU 地址 (`0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`)。这使得路由器中带有 `ETH` 后缀的函数能够正确处理 WJU。

#### 3. 与 WJU 交互 (封装/解封装)

将原生 JU 转换为 ERC20 兼容的 WJU 是与 DEX 交互的前提。

* **封装 JU (Wrap)**: 调用 WJU 合约的 `deposit()` payable 函数。
  * **操作:** 构造一个发往 WJU 合约地址的交易，调用 `deposit()` (ABI 中提供)，并在交易中附带 `value` 字段，其值为您希望封装的原生 JU 数量 (单位: Wei)。
* **解封装 WJU (Unwrap)**: 调用 WJU 合约的 `withdraw(uint256 wad)` 函数。
  * **操作:** 调用 `withdraw` 函数，传入参数 `wad` 为您希望解封装换回原生 JU 的 WJU 数量 (单位: Wei)。此交易不附带 `value`。

#### 4. Swap (兑换) 功能 (使用路由器)

通过路由器合约，利用已存在的流动性池进行代币兑换。

**常用 Swap 函数:**

* `swapExactETHForTokens`: 用 **精确数量** 的 WJU 兑换 **至少** 某个数量的 ERC20 代币。
* `swapExactTokensForETH`: 用 **精确数量** 的 ERC20 代币 兑换 **至少** 某个数量的 WJU。
* `swapExactTokensForTokens`: 用 **精确数量** 的输入 ERC20 代币，兑换 **至少** 某个数量的输出 ERC20 代币。

**核心参数:**

* `amountIn` / `amountOut`: 涉及的数量。
* `amountOutMin` / `amountInMax`: 滑点控制，保证收到的不少于/付出的不多于此值。
* `path`: 地址数组，定义兑换路径，如 `[WJU_ADDRESS, USDT_ADDRESS]` 或 `[USDT_ADDRESS, WJU_ADDRESS]`。
* `to`: 接收代币的地址。
* `deadline`: Unix 时间戳，交易在此时间之后失效。

**流程: 使用 WJU 兑换 USDT (调用 `swapExactETHForTokens`)**

1. **确定输入:** 决定要花费的精确 WJU 数量 (`wjuAmountIn`，单位 Wei)。
2. **计算最小输出 (滑点控制):**
   * 调用路由器的 `getAmountsOut(wjuAmountIn, [WJU_ADDRESS, USDT_ADDRESS])` 视图函数，获取预期的 USDT 输出量 `expectedUsdtAmount`。
   * 根据可接受的滑点百分比 (e.g., 1% or 0.01) 计算 `usdtAmountOutMin = expectedUsdtAmount * (1 - slippageTolerance)`。**注意：** 处理大数运算时需使用 BigInt。
3. **准备交易:**
   * 设置 `deadline` (例如，当前时间 + 10 分钟)。
   * 构造 `swapExactETHForTokens` 函数调用，参数为：`usdtAmountOutMin`, `path = [WJU_ADDRESS, USDT_ADDRESS]`, `to = USER_ADDRESS`, `deadline`。
4. **发送交易:**
   * 对构造好的交易进行签名 (使用 `PRIVATE_KEY`)。
   * 发送交易时，**必须附加 `value: wjuAmountIn`**，因为 WJU 是通过原生代币方式传递的。
   * 建议先使用 `estimateGas` 估算 Gas Limit，并可适当增加冗余 (e.g., \* 1.1)。

**流程: 使用 USDT 兑换 WJU (调用 `swapExactTokensForETH`)**

1. **确定输入:** 决定要花费的精确 USDT 数量 (`usdtAmountIn`，单位 Wei)。
2. **批准 Router:**
   * 调用 **USDT 合约** 的 `approve(ROUTER_ADDRESS, usdtAmountIn)` 函数。
   * **必须等待**此批准交易成功确认后才能进行下一步。
   * _优化:_ 可先调用 `usdtContract.methods.allowance(USER_ADDRESS, ROUTER_ADDRESS)` 检查现有额度，若足够则跳过批准。
3. **计算最小输出 (滑点控制):**
   * 调用路由器的 `getAmountsOut(usdtAmountIn, [USDT_ADDRESS, WJU_ADDRESS])` 获取预期的 WJU 输出量 `expectedWjuAmount`。
   * 计算 `wjuAmountOutMin = expectedWjuAmount * (1 - slippageTolerance)`。
4. **准备交易:**
   * 设置 `deadline`。
   * 构造 `swapExactTokensForETH` 函数调用，参数为：`usdtAmountIn`, `wjuAmountOutMin`, `path = [USDT_ADDRESS, WJU_ADDRESS]`, `to = USER_ADDRESS`, `deadline`。
5. **发送交易:**
   * 签名并发送交易。
   * 此交易**不附加 `value`**。
   * 同样建议估算 Gas。

**Swap 注意事项:**

* **滑点:** `amountOutMin`/`amountInMax` 是防止价格不利变动的重要保护措施。
* **Gas:** 所有写操作都需要 Gas (原生 JU)。
* **Deadline:** 防止交易卡在交易池中过久而导致执行时价格与预期偏差过大。
* **路径 (`path`):** 确保路径正确，对于复杂兑换可能需要经过 WJU 作为中间路由。

#### 5. 流动性管理 (使用路由器)

向交易对提供流动性以赚取交易手续费，并获得代表份额的 LP 代币。

**常用流动性函数:**

* `addLiquidityETH`: 添加 WJU 和一种 ERC20 代币的流动性。
* `removeLiquidityETH`: 移除 WJU 和一种 ERC20 代币的流动性。
* `addLiquidity`: 添加两种 ERC20 代币的流动性。
* `removeLiquidity`: 移除两种 ERC20 代币的流动性。

**流程: 添加 WJU-USDT 流动性 (调用 `addLiquidityETH`)**

1. **确定期望数量:** 决定期望提供的 WJU 数量 (`wjuAmountDesired`) 和 USDT 数量 (`usdtAmountDesired`)。注意：实际添加比例将由当前池子决定，您提供的数量只是上限和期望比例。
2. **计算最小接受数量 (滑点控制):** 根据可接受的滑点计算 `wjuAmountMin` 和 `usdtAmountMin`。
3. **检查余额:** 确保账户中有足够的 WJU (通过 `value` 提供) 和 USDT。
4. **批准 Router (针对 USDT):**
   * 调用 **USDT 合约** 的 `approve(ROUTER_ADDRESS, usdtAmountDesired)` 函数。
   * 等待批准交易确认。
   * _优化:_ 可先检查 `allowance`，若足够则跳过。
5. **准备交易:**
   * 设置 `deadline`。
   * 构造 `addLiquidityETH` 函数调用，参数为：`token = USDT_ADDRESS`, `amountTokenDesired = usdtAmountDesired`, `amountTokenMin = usdtAmountMin`, `amountETHMin = wjuAmountMin`, `to = USER_ADDRESS`, `deadline`。
6. **发送交易:**
   * 签名并发送交易。
   * **必须附加 `value: wjuAmountDesired`** 来提供 WJU。
   * 建议估算 Gas。交易成功后，`USER_ADDRESS` 会收到 WJU-USDT 的 LP 代币。

**流程: 移除 WJU-USDT 流动性 (调用 `removeLiquidityETH`)**

1. **确定移除数量:** 决定要移除的 LP 代币数量 (`liquidityAmount`，单位 Wei)。LP 代币通常也是 18 位小数。
2. **计算最小取回数量 (滑点控制):** 根据当前池子比例和可接受滑点，估算最少能取回的 WJU 数量 (`wjuAmountMin`) 和 USDT 数量 (`usdtAmountMin`)。可以通过查询 Pair 合约的 `getReserves` 和 LP 代币的 `totalSupply` 来辅助计算。
3. **获取 Pair 地址:** 调用 `factoryContract.methods.getPair(WJU_ADDRESS, USDT_ADDRESS)` 获取 WJU-USDT 的 Pair 合约地址 (`pairAddress`)。
4. **批准 Router (针对 LP 代币):**
   * 创建 Pair 合约实例: `const pairContract = new web3.eth.Contract(ABIs.JUV2PAIR_ABI, pairAddress);`
   * 调用 **Pair 合约** 的 `approve(ROUTER_ADDRESS, liquidityAmount)` 函数。
   * 等待批准交易确认。
5. **准备交易:**
   * 设置 `deadline`。
   * 构造 `removeLiquidityETH` 函数调用，参数为：`token = USDT_ADDRESS`, `liquidity = liquidityAmount`, `amountTokenMin = usdtAmountMin`, `amountETHMin = wjuAmountMin`, `to = USER_ADDRESS`, `deadline`。
6. **发送交易:**
   * 签名并发送交易。
   * 此交易**不附加 `value`**。
   * 建议估算 Gas。交易成功后，`USER_ADDRESS` 会收到 WJU 和 USDT。

**流动性管理注意事项:**

* **无常损失:** 提供流动性的固有风险，当代币相对价格变动时可能发生。
* **LP 代币:** LP 代币本身也是 ERC20，可以转移或用于其他 DeFi 协议 (如果支持)。
* **首次添加:** 如果是第一个提供者，将设定初始汇率。

#### 6. 使用工厂 (Factory)

工厂合约主要用于创建和查询交易对。

* **查询交易对地址:**
  * 调用 `getPair(address tokenA, address tokenB)`。
  * 如果返回非零地址，则交易对已存在。
  * 如果返回 `0x000...000`，则交易对不存在。
* **创建交易对:**
  * 如果通过 `getPair` 确认交易对不存在，可以调用 `createPair(address tokenA, address tokenB)`。
  * `tokenA` 和 `tokenB` 地址无序。
  * 工厂会使用 `create2` 部署新的 Pair 合约，并发出 `PairCreated` 事件。
  * 通常，`addLiquidity*` 函数在交易对不存在时会自动调用 `createPair`，但手动调用可以确保其存在。
* **获取所有交易对信息:** `allPairsLength()` 和 `allPairs(uint index)` 可用，但遍历可能成本较高。
* **确定性计算交易对地址:** 使用 `pairFor` 逻辑 (参见下方 Solidity 示例)，结合工厂地址、排序后的 Token 地址和 `initCodeHash`，可以在链下预先计算出交易对地址。

**示例: 计算交易对地址 (Solidity 辅助库)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

library PairUtil {
    // 计算给定工厂和代币对的交易对地址
    function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair) {
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        // JuChain DEX Pair 合约初始化代码哈希
        bytes32 initCodeHash = 0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b;
        pair = address(uint160(uint(keccak256(abi.encodePacked(
                hex'ff', factory, salt, initCodeHash
            )))));
    }
}

contract MyDEXInteractor {
    address constant FACTORY = 0x66682281BdfeC17fCBcae0480C77edFb0c489339;
    address constant WJU = 0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3;
    address constant USDT = 0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02;

    // 获取 WJU-USDT 交易对的预期地址
    function getWjuUsdtPairAddress() external pure returns (address) {
        return PairUtil.pairFor(FACTORY, WJU, USDT);
    }
}
```

#### 7. 合约 ABI

**强烈建议:** 始终从 JuChain 区块浏览器 (`https://testnet.juscan.io`) 验证并获取指定合约地址的最新、最完整的官方 ABI。

```json
{
  "ABIs": {
    "WJU_ABI": [{ "constant": false, "inputs": [ { "name": "guy", "type": "address" }, { "name": "wad", "type": "uint256" } ], "name": "approve", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [], "name": "deposit", "outputs": [], "payable": true, "stateMutability": "payable", "type": "function" }, { "constant": false, "inputs": [ { "name": "dst", "type": "address" }, { "name": "wad", "type": "uint256" } ], "name": "transfer", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "src", "type": "address" }, { "name": "dst", "type": "address" }, { "name": "wad", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "wad", "type": "uint256" } ], "name": "withdraw", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "payable": true, "stateMutability": "payable", "type": "fallback" }, { "anonymous": false, "inputs": [ { "indexed": true, "name": "src", "type": "address" }, { "indexed": true, "name": "guy", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Approval", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "name": "src", "type": "address" }, { "indexed": true, "name": "dst", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Transfer", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "name": "dst", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Deposit", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "name": "src", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Withdrawal", "type": "event" }, { "constant": true, "inputs": [ { "name": "", "type": "address" }, { "name": "", "type": "address" } ], "name": "allowance", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "", "type": "address" } ], "name": "balanceOf", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "decimals", "outputs": [ { "name": "", "type": "uint8" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "name", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "symbol", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "totalSupply", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" } ],
    "USDT_ABI": [{ "inputs": [], "stateMutability": "nonpayable", "type": "constructor" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "owner", "type": "address" }, { "indexed": true, "internalType": "address", "name": "spender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Approval", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "from", "type": "address" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Transfer", "type": "event" }, { "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" } ], "name": "allowance", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "approve", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "account", "type": "address" } ], "name": "balanceOf", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "decimals", "outputs": [ { "internalType": "uint8", "name": "", "type": "uint8" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "subtractedValue", "type": "uint256" } ], "name": "decreaseAllowance", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "addedValue", "type": "uint256" } ], "name": "increaseAllowance", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [], "name": "name", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "symbol", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "totalSupply", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "transfer", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "from", "type": "address" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" } ],
    "FACTORY_ABI": [{ "inputs": [ { "internalType": "address", "name": "_feeToSetter", "type": "address" } ], "payable": false, "stateMutability": "nonpayable", "type": "constructor" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "token0", "type": "address" }, { "indexed": true, "internalType": "address", "name": "token1", "type": "address" }, { "indexed": false, "internalType": "address", "name": "pair", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "", "type": "uint256" } ], "name": "PairCreated", "type": "event" }, { "constant": true, "inputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "name": "allPairs", "outputs": [ { "internalType": "address", "name": "pair", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "allPairsLength", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "tokenA", "type": "address" }, { "internalType": "address", "name": "tokenB", "type": "address" } ], "name": "createPair", "outputs": [ { "internalType": "address", "name": "pair", "type": "address" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [], "name": "feeTo", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "feeToSetter", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "internalType": "address", "name": "", "type": "address" }, { "internalType": "address", "name": "", "type": "address" } ], "name": "getPair", "outputs": [ { "internalType": "address", "name": "pair", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "_feeTo", "type": "address" } ], "name": "setFeeTo", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "_feeToSetter", "type": "address" } ], "name": "setFeeToSetter", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" } ],
    "ROUTER_ABI": [{ "inputs": [ { "internalType": "address", "name": "_factory", "type": "address" }, { "internalType": "address", "name": "_WETH", "type": "address" } ], "stateMutability": "nonpayable", "type": "constructor" }, { "inputs": [], "name": "WETH", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "stateMutability": "pure", "type": "function" }, { "inputs": [], "name": "factory", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "stateMutability": "pure", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "tokenA", "type": "address" }, { "internalType": "address", "name": "tokenB", "type": "address" }, { "internalType": "uint256", "name": "amountADesired", "type": "uint256" }, { "internalType": "uint256", "name": "amountBDesired", "type": "uint256" }, { "internalType": "uint256", "name": "amountAMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountBMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "addLiquidity", "outputs": [ { "internalType": "uint256", "name": "amountA", "type": "uint256" }, { "internalType": "uint256", "name": "amountB", "type": "uint256" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "token", "type": "address" }, { "internalType": "uint256", "name": "amountTokenDesired", "type": "uint256" }, { "internalType": "uint256", "name": "amountTokenMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountETHMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "addLiquidityETH", "outputs": [ { "internalType": "uint256", "name": "amountToken", "type": "uint256" }, { "internalType": "uint256", "name": "amountETH", "type": "uint256" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" } ], "stateMutability": "payable", "type": "function" }, { "inputs": [ { "internalType": "uint256", "name": "amountOut", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" } ], "name": "getAmountsIn", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "uint256", "name": "amountIn", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" } ], "name": "getAmountsOut", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "tokenA", "type": "address" }, { "internalType": "address", "name": "tokenB", "type": "address" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" }, { "internalType": "uint256", "name": "amountAMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountBMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "removeLiquidity", "outputs": [ { "internalType": "uint256", "name": "amountA", "type": "uint256" }, { "internalType": "uint256", "name": "amountB", "type": "uint256" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "token", "type": "address" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" }, { "internalType": "uint256", "name": "amountTokenMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountETHMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "removeLiquidityETH", "outputs": [ { "internalType": "uint256", "name": "amountToken", "type": "uint256" }, { "internalType": "uint256", "name": "amountETH", "type": "uint256" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "swapExactETHForTokens", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "payable", "type": "function" }, { "inputs": [ { "internalType": "uint256", "name": "amountIn", "type": "uint256" }, { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "swapExactTokensForETH", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "uint256", "name": "amountIn", "type": "uint256" }, { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "swapExactTokensForTokens", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "nonpayable", "type": "function" }, { "stateMutability": "payable", "type": "receive" } ],
    "JUV2PAIR_ABI": { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "owner", "type": "address" }, { "indexed": true, "internalType": "address", "name": "spender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Approval", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "sender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount0", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1", "type": "uint256" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" } ], "name": "Burn", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "sender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount0", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1", "type": "uint256" } ], "name": "Mint", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "sender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount0In", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1In", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount0Out", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1Out", "type": "uint256" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" } ], "name": "Swap", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": false, "internalType": "uint112", "name": "reserve0", "type": "uint112" }, { "indexed": false, "internalType": "uint112", "name": "reserve1", "type": "uint112" } ], "name": "Sync", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "from", "type": "address" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Transfer", "type": "event" }, { "constant": true, "inputs": [], "name": "DOMAIN_SEPARATOR", "outputs": [ { "internalType": "bytes32", "name": "", "type": "bytes32" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "MINIMUM_LIQUIDITY", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "pure", "type": "function" }, { "constant": true, "inputs": [], "name": "PERMIT_TYPEHASH", "outputs": [ { "internalType": "bytes32", "name": "", "type": "bytes32" } ], "payable": false, "stateMutability": "pure", "type": "function" }, { "constant": true, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" } ], "name": "allowance", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "approve", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" } ], "name": "balanceOf", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "burn", "outputs": [ { "internalType": "uint256", "name": "amount0", "type": "uint256" }, { "internalType": "uint256", "name": "amount1", "type": "uint256" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [], "name": "decimals", "outputs": [ { "internalType": "uint8", "name": "", "type": "uint8" } ], "payable": false, "stateMutability": "pure", "type": "function" }, { "constant": true, "inputs": [], "name": "factory", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "getReserves", "outputs": [ { "internalType": "uint112", "name": "_reserve0", "type": "uint112" }, { "internalType": "uint112", "name": "_reserve1", "type": "uint112" }, { "internalType": "uint32", "name": "_blockTimestampLast", "type": "uint32" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "_token0", "type": "address" }, { "internalType": "address", "name": "_token1", "type": "address" } ], "name": "initialize", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [], "name": "kLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "mint", "outputs": [ { "internalType": "uint256", "name": "liquidity", "type": "uint256" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [], "name": "name", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "payable": false, "stateMutability": "pure", "type": "function" }, { "constant": true, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" } ], "name": "nonces", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" }, { "internalType": "uint8", "name": "v", "type": "uint8" }, { "internalType": "bytes32", "name": "r", "type": "bytes32" }, { "internalType": "bytes32", "name": "s", "type": "bytes32" } ], "name": "permit", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [], "name": "price0CumulativeLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "price1CumulativeLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "skim", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "uint256", "name": "amount0Out", "type": "uint256" }, { "internalType": "uint256", "name": "amount1Out", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "bytes", "name": "data", "type": "bytes" } ], "name": "swap", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [], "name": "symbol", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "payable": false, "stateMutability": "pure", "type": "function" }, { "constant": false, "inputs": [], "name": "sync", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [], "name": "token0", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "token1", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "totalSupply", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "transfer", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "internalType": "address", "name": "from", "type": "address" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" } ]
  }
}
```

{% hint style="info" %}
**交互最佳实践 (参考实现体现):**

* **交易发送封装:** 使用辅助函数处理 nonce 获取、gas price 设置、交易签名 (`web3.eth.accounts.signTransaction`) 和发送 (`web3.eth.sendSignedTransaction`)，并包含 `try...catch` 错误处理。
* **Gas 估算:** 在发送交易前使用合约方法的 `.estimateGas()`，并可增加少量 buffer (e.g., 10-20%) 防止因链上状态变化导致 Gas 不足。
* **滑点控制:** 对于 Swap 和流动性操作，务必计算并使用 `amount*Min` 或 `amount*Max` 参数。
* **Approve 检查:** 在执行需要花费 ERC20 代币的操作前，检查现有 `allowance`，避免不必要的 `approve` 交易。
* **Deadline:** 为有时限的操作 (Swap, Liquidity) 设置合理的 `deadline`。
* **大数处理:** 使用 `BigInt` (Javascript 原生) 或 `web3.utils.toWei/fromWei` 以及 BN.js (Web3.js 内置) 处理代币数量。
{% endhint %}

{% hint style="warning" %}
在部署到主网或处理实际资产前，请务必在测试网上进行充分的测试。始终参考 JuChain 官方文档和区块浏览器以获取最新、最权威的信息和完整的 ABI。安全地管理您的私钥！
{% endhint %}
