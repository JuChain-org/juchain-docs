# 去中心化交易所

JuChain 提供了一套基于 Uniswap V2 模型的去中心化交易所（DEX）功能，允许用户进行代币兑换（Swap）和提供流动性（Liquidity）。本文档旨在指导开发者如何使用 Javascript (如 Web3.js) 或 Solidity 与核心 DEX 合约进行交互。

**核心组件:**

* **路由器 (`JUV2Router02`)**: 用户交互的主要入口合约（执行 Swap、添加/移除流动性）。
* **工厂 (`JUV2Factory`)**: 创建和管理交易对合约。
* **交易对 (`JUV2Pair`)**: 代表一个特定的代币交易对（例如 WJU-USDT），并持有流动性资金池。其本身也是一个 ERC20 代币 (LP Token)。
* **WJU (Wrapped JU)**: 原生 JU 代币的封装版本，符合 ERC20 标准，以便与 DEX 交互。
* **USDT**: 网络上的一个示例 ERC20 代币。

**重要参考:**\
本文档描述了概念和关键函数。一个**经过验证且可运行的 Javascript (Web3.js) 交互脚本**随本文档提供，包含了创建交易对、添加流动性、查询状态和执行 Swap 的完整示例。**建议开发者参考该脚本进行具体实现。**

#### 1. 前置准备与网络设置

* **网络**: JuChain 测试网
  * **RPC URL**: `https://testnet-rpc.juchain.org`
  * **链 ID (Chain ID)**: `202599`
  * **区块浏览器**: `https://testnet.juscan.io`
* **工具**: Node.js 环境及 Web3.js 库 (如 `const { Web3 } = require('web3');`)，或者 Solidity 开发环境。
* **账户**: 一个拥有原生 JU（用于支付 Gas 费）以及相关代币（如 WJU, USDT）的外部账户 (EOA)，并安全地管理其私钥。

#### 2. 核心合约地址与详情

| 合约               | 地址                                                                   | Solidity 版本 (参考) | 说明                                                        |
| ---------------- | -------------------------------------------------------------------- | ---------------- | --------------------------------------------------------- |
| **WJU**          | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`                         | `0.4.x`          | 封装版 JU。使用 `deposit()` (payable) 封装原生 JU，`withdraw()` 解封装。 |
| **USDT**         | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`                         | `0.8.x`          | 标准 ERC20 代币。                                              |
| **工厂 (Factory)** | `0x66682281BdfeC17fCBcae0480C77edFb0c489339`                         | `0.5.x`          | 部署并追踪 `JUV2Pair` 交易对合约。                                   |
| **路由器 (Router)** | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0`                         | `0.6.x`          | **Swap 和流动性管理的主要交互入口。** 内部将 WJU 地址视为其 `WETH` 地址。          |
| **交易对初始化哈希**     | `0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b` |                  | 用于确定性计算交易对地址 (参考 `JUV2Library.pairFor` 逻辑)。               |

**重要提示:** 路由器 (`JUV2Router02`) 合约内部使用了 `WETH` 地址的概念。在 JuChain 上，这个 `WETH` 地址**必须指向 WJU 的地址** (`0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`)，这样路由器中带有 `ETH` 后缀的函数 (例如 `addLiquidityETH`, `swapExactETHForTokens`) 才能正确地处理 WJU。

#### 3. 与 WJU 交互 (封装/解封装)

由于 DEX 主要与 ERC20 代币交互，您需要先将原生 JU 封装成 WJU。

* **封装 JU (Wrap)**: 向 WJU 合约发送原生 JU。可以通过调用其 `deposit()` payable 函数，或直接向合约地址发送原生 JU (触发 fallback 函数) 来实现。
  * **实现参考:** 参见随附 JS 脚本中的 WJU 合约交互逻辑（虽然脚本未直接展示封装，但原理相同）。发送交易时，`to` 设为 WJU 地址，并附带要封装的 `value` (原生 JU 数量)。
* **解封装 WJU (Unwrap)**: 调用 WJU 合约的 `withdraw(uint256 amount)` 函数，指定要换回原生 JU 的 WJU 数量。
  * **实现参考:** 参见随附 JS 脚本中的 WJU 合约交互逻辑（虽然脚本未直接展示解封装，但原理相同）。构造 `withdraw` 函数调用的交易数据并发送。

#### 4. Swap (兑换) 功能 (使用路由器)

Swap 功能允许用户通过已存在的流动性池将一种代币兑换成另一种代币。

**关键的路由器 Swap 函数 (常用):** (完整 ABI 见第 7 节)

* `swapExactTokensForTokens`: 用 **精确数量** 的输入 ERC20 代币，兑换 **至少** 某个数量的输出 ERC20 代币。
* `swapExactETHForTokens`: 用 **精确数量** 的 **WJU** (作为 ETH，通过 `value` 传递)，兑换 **至少** 某个数量的输出 ERC20 代币。
* `swapExactTokensForETH`: 用 **精确数量** 的输入 ERC20 代币，兑换 **至少** 某个数量的 **WJU** (作为 ETH)。

**通用 Swap 步骤:**

1. **(若输入为 ERC20) 批准路由器 (Approve Router)**: 如果您要花费的是 ERC20 代币（包括 WJU，当它作为 `swapExactTokensFor*` 的输入时），必须先调用该 ERC20 代币合约的 `approve(routerAddress, amount)` 函数，授权路由器代表您花费该代币。
2. **(可选) 查询预期输出**: 调用路由器的 `getAmountsOut(amountIn, path)` 来预估给定输入能获得多少输出，用于计算最小可接受数量 (`amountOutMin`) 以防止过大的滑点。
3. **调用 Swap 函数**: 调用路由器合约上对应的 Swap 函数 (如 `swapExactETHForTokens`, `swapExactTokensForETH`)。
   * **参数:** 通常包括最小输出量 (`amountOutMin`)、兑换路径 (`path` - 地址数组，如 `[WJU_ADDRESS, USDT_ADDRESS]`)、接收地址 (`to`) 和截止时间 (`deadline`)。
   * **对于 `swapExactETHFor*`:** 需要在交易中附加 `value`，其值为要花费的 WJU 数量。
   * **对于 `swapExactTokensFor*`:** 需要确保步骤 1 的批准已完成。

**示例: 用精确数量的 WJU 兑换 USDT (使用 `swapExactETHForTokens`)**

* **概念步骤:**
  1. 确定要花费的 WJU 数量 (`wjuAmountIn`)。
  2. (可选) 调用 `getAmountsOut` 预测 USDT 输出量，计算 `usdtAmountOutMin` (考虑滑点)。
  3. 调用 `routerContract.methods.swapExactETHForTokens(usdtAmountOutMin, [WJU_ADDRESS, USDT_ADDRESS], userAddress, deadline)`。
  4. 发送交易，并附加 `value: wjuAmountIn`。
* **实现参考:** 参见随附 JS 脚本中的 `swapWJUForUSDT` 函数。

**示例: 用精确数量的 USDT 兑换 WJU (使用 `swapExactTokensForETH`)**

* **概念步骤:**
  1. 确定要花费的 USDT 数量 (`usdtAmountIn`)。
  2. 调用 `usdtContract.methods.approve(ROUTER_ADDRESS, usdtAmountIn)` 并等待交易确认。
  3. (可选) 调用 `getAmountsOut` 预测 WJU 输出量，计算 `wjuAmountOutMin`。
  4. 调用 `routerContract.methods.swapExactTokensForETH(usdtAmountIn, wjuAmountOutMin, [USDT_ADDRESS, WJU_ADDRESS], userAddress, deadline)`。
  5. 发送交易 (无需附加 `value`)。
* **实现参考:** 虽然随附 JS 脚本未直接包含此示例，但逻辑与 `swapWJUForUSDT` 类似，只是调用的函数不同，且需要预先执行 Approve 操作。

**Swap 注意事项:**

* **滑点 (`amountOutMin`/`amountInMax`):** 设置合理的最小/最大值以保护自己免受价格剧烈波动的影响。
* **路径 (`path`):** 对于不直接相连的代币，路径可能包含中间代币 (例如 `[TOKEN_A, WJU, TOKEN_B]`)。
* **Gas 费:** Swap 交易会消耗 Gas，确保账户中有足够的原生 JU。
* **Deadline:** 防止交易因网络拥堵而被延迟执行，导致价格变化过大。

#### 5. 流动性管理 (使用路由器)

用户可以向交易对（如 WJU-USDT）提供流动性，并从该交易对发生的 Swap 中赚取手续费。提供流动性后，用户会收到代表其份额的 LP (Liquidity Provider) 代币。

**关键的路由器流动性函数 (常用):** (完整 ABI 见第 7 节)

* `addLiquidity`: 为两个 ERC20 代币组成的交易对添加流动性。
* `addLiquidityETH`: 为涉及 WJU (作为 ETH) 和另一个 ERC20 代币的交易对添加流动性。WJU 通过交易的 `value` 发送。
* `removeLiquidity`: 通过销毁 LP 代币，为两个 ERC20 代币组成的交易对移除流动性，取回两种基础代币。
* `removeLiquidityETH`: 通过销毁 LP 代币，为 WJU (作为 ETH) / ERC20 交易对移除流动性，取回 WJU 和该 ERC20 代币。

**添加流动性步骤:**

1. **(若提供 ERC20) 批准路由器 (Approve Router)**: 路由器需要权限来转移您提供的 ERC20 代币。对您要提供的 _每种_ ERC20 代币（**除非**是通过 `addLiquidityETH` 提供 WJU）调用其代币合约的 `approve(routerAddress, amount)`。
2. **调用添加函数**: 调用 `addLiquidity` 或 `addLiquidityETH`。
   * **参数:** 指定 _期望_ 添加的数量 (`amountADesired`, `amountBDesired`)，以及为控制滑点而设置的最小可接受数量 (`amountAMin`, `amountBMin`, `amountETHMin`)、接收 LP 代币的地址 (`to`) 和截止时间 (`deadline`)。
   * **对于 `addLiquidityETH`:** WJU 数量通过交易的 `value` 字段传递，参数中只需提供 ERC20 代币的信息 (`tokenAddress`, `amountTokenDesired`, `amountTokenMin`) 和 WJU 的最小数量 (`amountETHMin`)。

**示例: 为 WJU-USDT 添加流动性 (使用 `addLiquidityETH`)**

* **概念步骤:**
  1. 确定期望添加的 WJU 数量 (`wjuAmountDesired`) 和 USDT 数量 (`usdtAmountDesired`)。
  2. 计算最小可接受的 WJU 数量 (`wjuAmountMin`) 和 USDT 数量 (`usdtAmountMin`) (考虑滑点)。
  3. 调用 `usdtContract.methods.approve(ROUTER_ADDRESS, usdtAmountDesired)` 并等待交易确认。
  4. 调用 `routerContract.methods.addLiquidityETH(USDT_ADDRESS, usdtAmountDesired, usdtAmountMin, wjuAmountMin, userAddress, deadline)`。
  5. 发送交易，并附加 `value: wjuAmountDesired`。
* **实现参考:** 参见随附 JS 脚本中的 `addLiquidityWJUUSDT` 函数。

**移除流动性步骤:**

1. **获取交易对地址**: 找到您拥有 LP 代币的交易对合约地址。可以通过工厂合约的 `getPair(tokenA, tokenB)` 函数查询。
2. **批准路由器花费 LP 代币**: 您需要授权路由器销毁您的 LP 代币。调用 **交易对合约 (Pair Contract)** 的 `approve(routerAddress, liquidityAmount)` 函数 (使用 Pair ABI，见第 7 节)，其中 `liquidityAmount` 是您要移除的 LP 代币数量。
3. **调用移除函数**: 调用 `removeLiquidity` 或 `removeLiquidityETH`。
   * **参数:** 指定要销毁的 LP 代币数量 (`liquidity`)，以及您期望取回的两种基础代币的最小数量 (`amountAMin`, `amountBMin`, `amountETHMin`)、接收基础代币的地址 (`to`) 和截止时间 (`deadline`)。

**示例: 移除 WJU-USDT 流动性 (使用 `removeLiquidityETH`)**

* **概念步骤:**
  1. 确定要移除的 LP 代币数量 (`liquidityAmount`)。
  2. 计算期望取回的最小 WJU 数量 (`wjuAmountMin`) 和 USDT 数量 (`usdtAmountMin`)。
  3. 调用 `factoryContract.methods.getPair(WJU_ADDRESS, USDT_ADDRESS)` 获取 `pairAddress`。
  4. 创建 Pair 合约实例: `pairContract = new web3.eth.Contract(pairABI, pairAddress)`。
  5. 调用 `pairContract.methods.approve(ROUTER_ADDRESS, liquidityAmount)` 并等待交易确认。
  6. 调用 `routerContract.methods.removeLiquidityETH(USDT_ADDRESS, liquidityAmount, usdtAmountMin, wjuAmountMin, userAddress, deadline)`。
  7. 发送交易 (移除流动性通常不附加 `value`)。
* **实现参考:** 虽然随附 JS 脚本未直接包含此移除示例，但上述步骤和参数是正确的。

**流动性管理注意事项:**

* **无常损失:** 提供流动性存在无常损失的风险，当池中代币价格相对变化时可能发生。
* **首次添加:** 如果您是第一个为某个交易对提供流动性的人，您将设定初始价格。
* **Gas 费:** 添加和移除流动性都需要 Gas。

#### 6. 使用工厂 (Factory)

虽然大部分交互通过路由器进行，但有时您可能需要直接与工厂合约交互。

* **检查/获取交易对地址:** 调用 `getPair(address tokenA, address tokenB)`。如果交易对已存在，返回其地址；否则返回零地址 (`address(0)`)。
  * **实现参考:** 参见随附 JS 脚本中的 `testQueryPool` 和 `createWJUUSDTPair` 函数。
* **创建交易对:** 调用 `createPair(address tokenA, address tokenB)`。如果交易对不存在，工厂会部署一个新的交易对合约并发出 `PairCreated` 事件。通常，在首次为新交易对调用 `addLiquidity` 时，路由器会自动为您调用此函数，但也可以手动调用。
  * **实现参考:** 参见随附 JS 脚本中的 `createWJUUSDTPair` 函数。
* **获取所有交易对信息:** 可以通过 `allPairsLength()` 获取总数，并通过 `allPairs(uint index)` 遍历所有已创建的交易对地址 (可能消耗较多资源)。
* **确定性地计算交易对地址:** 使用 `pairFor` 逻辑（参见下方 Solidity 示例）。这依赖于 `create2` 操作码、工厂地址、两种代币地址（排序后）以及**交易对初始化哈希** (`0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b`)。

**示例: 计算交易对地址 (Solidity 辅助库)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

library PairUtil {
    // 计算给定工厂和代币对的交易对地址
    function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair) {
        // 确保代币地址排序一致
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        // 计算盐值 (salt)
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        // JuChain DEX 的 Pair 合约初始化代码哈希
        bytes32 initCodeHash = 0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b;
        // 使用 create2 地址计算公式
        pair = address(uint160(uint(keccak256(abi.encodePacked(
                hex'ff', // create2 前缀
                factory, // 工厂地址
                salt,    // 盐值
                initCodeHash // 初始化代码哈希
            )))));
    }
}

// 示例合约如何使用
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

以下 ABI 来自经过验证的 Javascript 交互脚本，可用于开发。

**强烈建议:** 始终从 JuChain 区块浏览器 (`https://testnet.juscan.io`) 验证并获取指定合约地址的最新、最完整的官方 ABI。

```json
{
  "ABIs": {
    "WJU_ABI": [
      { "constant": false, "inputs": [ { "name": "guy", "type": "address" }, { "name": "wad", "type": "uint256" } ], "name": "approve", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": false, "inputs": [], "name": "deposit", "outputs": [], "payable": true, "stateMutability": "payable", "type": "function" },
      { "constant": false, "inputs": [ { "name": "dst", "type": "address" }, { "name": "wad", "type": "uint256" } ], "name": "transfer", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": false, "inputs": [ { "name": "src", "type": "address" }, { "name": "dst", "type": "address" }, { "name": "wad", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": false, "inputs": [ { "name": "wad", "type": "uint256" } ], "name": "withdraw", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "payable": true, "stateMutability": "payable", "type": "fallback" },
      { "anonymous": false, "inputs": [ { "indexed": true, "name": "src", "type": "address" }, { "indexed": true, "name": "guy", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Approval", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "name": "src", "type": "address" }, { "indexed": true, "name": "dst", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Transfer", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "name": "dst", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Deposit", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "name": "src", "type": "address" }, { "indexed": false, "name": "wad", "type": "uint256" } ], "name": "Withdrawal", "type": "event" },
      { "constant": true, "inputs": [ { "name": "", "type": "address" }, { "name": "", "type": "address" } ], "name": "allowance", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [ { "name": "", "type": "address" } ], "name": "balanceOf", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "decimals", "outputs": [ { "name": "", "type": "uint8" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "name", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "symbol", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "totalSupply", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }
    ],
    "USDT_ABI": [
      { "inputs": [], "stateMutability": "nonpayable", "type": "constructor" },
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "owner", "type": "address" }, { "indexed": true, "internalType": "address", "name": "spender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Approval", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "from", "type": "address" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Transfer", "type": "event" },
      { "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" } ], "name": "allowance", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "approve", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "account", "type": "address" } ], "name": "balanceOf", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [], "name": "decimals", "outputs": [ { "internalType": "uint8", "name": "", "type": "uint8" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "subtractedValue", "type": "uint256" } ], "name": "decreaseAllowance", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "addedValue", "type": "uint256" } ], "name": "increaseAllowance", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [], "name": "name", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [], "name": "symbol", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [], "name": "totalSupply", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "transfer", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "from", "type": "address" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }
    ],
    "FACTORY_ABI": [
      { "inputs": [ { "internalType": "address", "name": "_feeToSetter", "type": "address" } ], "payable": false, "stateMutability": "nonpayable", "type": "constructor" },
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "token0", "type": "address" }, { "indexed": true, "internalType": "address", "name": "token1", "type": "address" }, { "indexed": false, "internalType": "address", "name": "pair", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "", "type": "uint256" } ], "name": "PairCreated", "type": "event" },
      { "constant": true, "inputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "name": "allPairs", "outputs": [ { "internalType": "address", "name": "pair", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "allPairsLength", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "tokenA", "type": "address" }, { "internalType": "address", "name": "tokenB", "type": "address" } ], "name": "createPair", "outputs": [ { "internalType": "address", "name": "pair", "type": "address" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [], "name": "feeTo", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "feeToSetter", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [ { "internalType": "address", "name": "", "type": "address" }, { "internalType": "address", "name": "", "type": "address" } ], "name": "getPair", "outputs": [ { "internalType": "address", "name": "pair", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "_feeTo", "type": "address" } ], "name": "setFeeTo", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "_feeToSetter", "type": "address" } ], "name": "setFeeToSetter", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }
     ],
    "ROUTER_ABI": [ // 注意：这是常用函数片段，并非完整 ABI！建议获取完整版。
      { "inputs": [ { "internalType": "address", "name": "_factory", "type": "address" }, { "internalType": "address", "name": "_WETH", "type": "address" } ], "stateMutability": "nonpayable", "type": "constructor" },
      { "inputs": [], "name": "WETH", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "stateMutability": "pure", "type": "function" },
      { "inputs": [], "name": "factory", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "stateMutability": "pure", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "tokenA", "type": "address" }, { "internalType": "address", "name": "tokenB", "type": "address" }, { "internalType": "uint256", "name": "amountADesired", "type": "uint256" }, { "internalType": "uint256", "name": "amountBDesired", "type": "uint256" }, { "internalType": "uint256", "name": "amountAMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountBMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "addLiquidity", "outputs": [ { "internalType": "uint256", "name": "amountA", "type": "uint256" }, { "internalType": "uint256", "name": "amountB", "type": "uint256" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "token", "type": "address" }, { "internalType": "uint256", "name": "amountTokenDesired", "type": "uint256" }, { "internalType": "uint256", "name": "amountTokenMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountETHMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "addLiquidityETH", "outputs": [ { "internalType": "uint256", "name": "amountToken", "type": "uint256" }, { "internalType": "uint256", "name": "amountETH", "type": "uint256" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" } ], "stateMutability": "payable", "type": "function" },
      { "inputs": [ { "internalType": "uint256", "name": "amountOut", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" } ], "name": "getAmountsIn", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [ { "internalType": "uint256", "name": "amountIn", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" } ], "name": "getAmountsOut", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "view", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "tokenA", "type": "address" }, { "internalType": "address", "name": "tokenB", "type": "address" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" }, { "internalType": "uint256", "name": "amountAMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountBMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "removeLiquidity", "outputs": [ { "internalType": "uint256", "name": "amountA", "type": "uint256" }, { "internalType": "uint256", "name": "amountB", "type": "uint256" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [ { "internalType": "address", "name": "token", "type": "address" }, { "internalType": "uint256", "name": "liquidity", "type": "uint256" }, { "internalType": "uint256", "name": "amountTokenMin", "type": "uint256" }, { "internalType": "uint256", "name": "amountETHMin", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "removeLiquidityETH", "outputs": [ { "internalType": "uint256", "name": "amountToken", "type": "uint256" }, { "internalType": "uint256", "name": "amountETH", "type": "uint256" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [ { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "swapExactETHForTokens", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "payable", "type": "function" },
      { "inputs": [ { "internalType": "uint256", "name": "amountIn", "type": "uint256" }, { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "swapExactTokensForETH", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "nonpayable", "type": "function" },
      { "inputs": [ { "internalType": "uint256", "name": "amountIn", "type": "uint256" }, { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" }, { "internalType": "address[]", "name": "path", "type": "address[]" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" } ], "name": "swapExactTokensForTokens", "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ], "stateMutability": "nonpayable", "type": "function" },
      { "stateMutability": "payable", "type": "receive" }
    ],
    "JUV2PAIR_ABI": [ // Pair ABI - 用于 LP 代币交互 (如 approve) 和查询储备
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "owner", "type": "address" }, { "indexed": true, "internalType": "address", "name": "spender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Approval", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "sender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount0", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1", "type": "uint256" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" } ], "name": "Burn", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "sender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount0", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1", "type": "uint256" } ], "name": "Mint", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "sender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount0In", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1In", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount0Out", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "amount1Out", "type": "uint256" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" } ], "name": "Swap", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": false, "internalType": "uint112", "name": "reserve0", "type": "uint112" }, { "indexed": false, "internalType": "uint112", "name": "reserve1", "type": "uint112" } ], "name": "Sync", "type": "event" },
      { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "from", "type": "address" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Transfer", "type": "event" },
      { "constant": true, "inputs": [], "name": "DOMAIN_SEPARATOR", "outputs": [ { "internalType": "bytes32", "name": "", "type": "bytes32" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "MINIMUM_LIQUIDITY", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "pure", "type": "function" },
      { "constant": true, "inputs": [], "name": "PERMIT_TYPEHASH", "outputs": [ { "internalType": "bytes32", "name": "", "type": "bytes32" } ], "payable": false, "stateMutability": "pure", "type": "function" },
      { "constant": true, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" } ], "name": "allowance", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "approve", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" } ], "name": "balanceOf", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "burn", "outputs": [ { "internalType": "uint256", "name": "amount0", "type": "uint256" }, { "internalType": "uint256", "name": "amount1", "type": "uint256" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [], "name": "decimals", "outputs": [ { "internalType": "uint8", "name": "", "type": "uint8" } ], "payable": false, "stateMutability": "pure", "type": "function" },
      { "constant": true, "inputs": [], "name": "factory", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "getReserves", "outputs": [ { "internalType": "uint112", "name": "_reserve0", "type": "uint112" }, { "internalType": "uint112", "name": "_reserve1", "type": "uint112" }, { "internalType": "uint32", "name": "_blockTimestampLast", "type": "uint32" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "_token0", "type": "address" }, { "internalType": "address", "name": "_token1", "type": "address" } ], "name": "initialize", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [], "name": "kLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "mint", "outputs": [ { "internalType": "uint256", "name": "liquidity", "type": "uint256" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [], "name": "name", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "payable": false, "stateMutability": "pure", "type": "function" },
      { "constant": true, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" } ], "name": "nonces", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" }, { "internalType": "uint8", "name": "v", "type": "uint8" }, { "internalType": "bytes32", "name": "r", "type": "bytes32" }, { "internalType": "bytes32", "name": "s", "type": "bytes32" } ], "name": "permit", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [], "name": "price0CumulativeLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "price1CumulativeLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "skim", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "uint256", "name": "amount0Out", "type": "uint256" }, { "internalType": "uint256", "name": "amount1Out", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "bytes", "name": "data", "type": "bytes" } ], "name": "swap", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [], "name": "symbol", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "payable": false, "stateMutability": "pure", "type": "function" },
      { "constant": false, "inputs": [], "name": "sync", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": true, "inputs": [], "name": "token0", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "token1", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": true, "inputs": [], "name": "totalSupply", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "transfer", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
      { "constant": false, "inputs": [ { "internalType": "address", "name": "from", "type": "address" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }
    ]
  }
}
```

{% hint style="success" %}
**实践出真知:** 本文档提供了与 JuChain DEX 交互的核心概念和流程。**强烈建议参考随附的、经过验证的 Javascript 交互脚本**作为实际编码的起点和范例。该脚本包含了处理 Gas、Nonce、签名、错误处理以及具体函数调用的最佳实践。
{% endhint %}

{% hint style="info" %}
在部署到主网之前，请务必在测试网上进行充分的测试。始终参考 JuChain 官方文档和区块浏览器以获取最新、最权威的信息和完整的 ABI。
{% endhint %}
