# 去中心化交易所

JuChain 提供了一套去中心化交易（DEX）功能，通过 `JUV2Router02`（路由器）、`JUV2Factory`（工厂）和 `JUV2Pair`（交易对）合约实现代币交换（Swap）和流动性添加（Liquidity）。本文档将详细介绍如何在 JuChain 网络上执行代币 Swap 和添加流动性，适用于 WJU（Wrapped JU）、USDT 等代币。

#### 关键要点

* **支持的代币**：
  * WJU（Wrapped JU）：`0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`
  * USDT：`0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`
* **核心合约**：
  * **Router**：`0x6A647E09193a130b0CccBF26A1CF442491bDeCc0`（Solidity 0.6.6）
  * **Factory**：`0x66682281BdfeC17fCBcae0480C77edFb0c489339`（Solidity 0.5.16）
* **功能**：
  * Swap：支持代币间交换，包括 WJU 与 USDT。
  * Liquidity：支持为交易对添加和移除流动性。

***

### 测试网设置

在 JuChain 测试网（或其他兼容网络）上操作时，请配置以下信息：

* **RPC**：`https://testnet-rpc.juchain.org`（以官方为准）。
* **浏览器**：`https://explorer-testnet.juchain.org`。
* **链 ID**：202599。

***

### 合约地址

以下是核心合约地址：

| **合约**  | **地址**                                       | **Solidity 版本** |
| ------- | -------------------------------------------- | --------------- |
| WJU     | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3` | 0.4.26          |
| USDT    | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02` | 0.8.8           |
| Factory | `0x66682281BdfeC17fCBcae0480C77edFb0c489339` | 0.5.16          |
| Router  | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0` | 0.6.6           |

***

## 一、Swap 功能

### 功能概述

Swap 功能通过 `JUV2Router02` 实现，支持以下操作：

* 用精确输入代币换目标代币。
* 用代币换精确数量的 WJU（ETH）。
* 支持带有转账费的代币。

### Router 合约方法

以下是常用的 Swap 方法：

1. **`swapExactTokensForTokens`**
   * **功能**：用精确数量的输入代币换取目标代币。
   * **参数**：
     * `amountIn`：输入代币数量。
     * `amountOutMin`：最小输出代币数量（滑点保护）。
     * `path`：交易路径（如 `[WJU, USDT]`）。
     * `to`：接收地址。
     * `deadline`：交易截止时间（Unix 时间戳）。
   * **状态**：`nonpayable`。
2. **`swapExactETHForTokens`**
   * **功能**：用精确数量的 WJU 换取目标代币。
   * **参数**：
     * `amountOutMin`：最小输出代币数量。
     * `path`：交易路径（如 `[WJU, USDT]`）。
     * `to`：接收地址。
     * `deadline`：交易截止时间。
   * **状态**：`payable`。
3. **`swapExactTokensForETH`**
   * **功能**：用精确数量的代币换取 WJU。
   * **参数**：
     * `amountIn`：输入代币数量。
     * `amountOutMin`：最小 WJU 数量。
     * `path`：交易路径（如 `[USDT, WJU]`）。
     * `to`：接收地址。
     * `deadline`：交易截止时间。
   * **状态**：`nonpayable`。
4. **`swapExactTokensForTokensSupportingFeeOnTransferTokens`**
   * **功能**：支持带有转账费的代币的精确输入换代币。
   * **参数**：同 `swapExactTokensForTokens`。
   * **状态**：`nonpayable`。

### 使用步骤

#### 1. 准备工作

* **批准代币**：调用代币的 `approve` 方法，授权 Router 转移代币。
* **检查交易对**：确保 Factory 中存在目标交易对（如 WJU-USDT）并有流动性。

#### 2. 示例代码

**用 WJU 换 USDT（Web3.js）**

```javascript
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

const routerAddress = '0x6A647E09193a130b0CccBF26A1CF442491bDeCc0';
const wjuAddress = '0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3';
const usdtAddress = '0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02';
const routerABI = [/* Router ABI */];
const wjuABI = [/* WJU ABI */];

const router = new web3.eth.Contract(routerABI, routerAddress);
const wju = new web3.eth.Contract(wjuABI, wjuAddress);

async function swapWJUForUSDT(amountIn, amountOutMin, account, privateKey) {
  const path = [wjuAddress, usdtAddress];
  const deadline = Math.floor(Date.now() / 1000) + 60 * 20;

  // 批准 Router 使用 WJU
  await wju.methods.approve(routerAddress, amountIn).send({ from: account });

  const tx = router.methods.swapExactTokensForTokens(
    amountIn,
    amountOutMin,
    path,
    account,
    deadline
  );
  const gas = await tx.estimateGas({ from: account });
  const receipt = await tx.send({ from: account, gas });

  console.log(`Swap 成功，交易哈希: ${receipt.transactionHash}`);
}

// 示例调用：用 1 WJU 换 USDT
swapWJUForUSDT(web3.utils.toWei('1', 'ether'), web3.utils.toWei('0.9', 'ether'), '0xYourAddress', '0xYourPrivateKey');
```

**用 USDT 换 WJU（Solidity）**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.6;

interface IJUV2Router02 {
    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)
        external returns (uint[] memory amounts);
}

interface IERC20 {
    function approve(address spender, uint value) external returns (bool);
}

contract SwapExample {
    address constant ROUTER = 0x6A647E09193a130b0CccBF26A1CF442491bDeCc0;
    address constant USDT = 0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02;
    address constant WJU = 0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3;

    IJUV2Router02 public router = IJUV2Router02(ROUTER);

    function swapUSDTForWJU(uint amountIn, uint amountOutMin) external {
        IERC20(USDT).approve(ROUTER, amountIn);
        address[] memory path = new address[](2);
        path[0] = USDT;
        path[1] = WJU;
        router.swapExactTokensForETH(amountIn, amountOutMin, path, msg.sender, block.timestamp + 1200);
    }
}
```

### 注意事项

* **滑点保护**：设置 `amountOutMin` 防止价格波动。
* **交易对流动性**：若流动性不足，Swap 可能失败。
* **Gas 优化**：估算 Gas 并留出余量。

***

## 二、Liquidity 添加与移除功能

### 功能概述

Liquidity 功能允许用户为交易对（如 WJU-USDT）添加或移除流动性，从而支持 Swap 操作。添加流动性会生成 LP 代币（Liquidity Provider Tokens），移除流动性则返还代币。

### Router 合约方法

1. **`addLiquidity`**
   * **功能**：为代币对添加流动性。
   * **参数**：
     * `tokenA`：代币 A 地址。
     * `tokenB`：代币 B 地址。
     * `amountADesired`：期望输入的代币 A 数量。
     * `amountBDesired`：期望输入的代币 B 数量。
     * `amountAMin`：最小输入代币 A 数量。
     * `amountBMin`：最小输入代币 B 数量。
     * `to`：接收 LP 代币的地址。
     * `deadline`：交易截止时间。
   * **状态**：`nonpayable`。
   * **返回**：`(amountA, amountB, liquidity)`。
2. **`addLiquidityETH`**
   * **功能**：用 WJU 和代币添加流动性。
   * **参数**：
     * `token`：代币地址。
     * `amountTokenDesired`：期望输入的代币数量。
     * `amountTokenMin`：最小代币数量。
     * `amountETHMin`：最小 WJU 数量。
     * `to`：接收 LP 代币的地址。
     * `deadline`：交易截止时间。
   * **状态**：`payable`。
   * **返回**：`(amountToken, amountETH, liquidity)`。
3. **`removeLiquidity`**
   * **功能**：移除流动性并返还代币。
   * **参数**：
     * `tokenA`：代币 A 地址。
     * `tokenB`：代币 B 地址。
     * `liquidity`：要移除的 LP 代币数量。
     * `amountAMin`：最小返还代币 A 数量。
     * `amountBMin`：最小返还代币 B 数量。
     * `to`：接收地址。
     * `deadline`：交易截止时间。
   * **状态**：`nonpayable`。
   * **返回**：`(amountA, amountB)`。
4. **`removeLiquidityETH`**
   * **功能**：移除 WJU-代币对的流动性。
   * **参数**：
     * `token`：代币地址。
     * `liquidity`：要移除的 LP 代币数量。
     * `amountTokenMin`：最小返还代币数量。
     * `amountETHMin`：最小返还 WJU 数量。
     * `to`：接收地址。
     * `deadline`：交易截止时间。
   * **状态**：`nonpayable`。
   * **返回**：`(amountToken, amountETH)`。

### 使用步骤

#### 1. 准备工作

* **批准代币**：为 Router 批准代币和 LP 代币。
* **创建交易对**：若交易对不存在，需通过 Factory 的 `createPair` 创建。

#### 2. 示例代码

**添加 WJU-USDT 流动性（Web3.js）**

```javascript
async function addLiquidityWJUUSDT(amountWJU, amountUSDT, account, privateKey) {
  const usdt = new web3.eth.Contract(usdtABI, usdtAddress);
  const deadline = Math.floor(Date.now() / 1000) + 60 * 20;

  // 批准 Router 使用 USDT
  await usdt.methods.approve(routerAddress, amountUSDT).send({ from: account });

  const tx = router.methods.addLiquidityETH(
    usdtAddress,
    amountUSDT,
    amountUSDT, // amountTokenMin
    amountWJU,  // amountETHMin
    account,
    deadline
  );
  const gas = await tx.estimateGas({ from: account, value: amountWJU });
  const receipt = await tx.send({ from: account, gas, value: amountWJU });

  console.log(`添加流动性成功，交易哈希: ${receipt.transactionHash}`);
}

// 示例调用：添加 1 WJU 和 1 USDT
addLiquidityWJUUSDT(web3.utils.toWei('1', 'ether'), web3.utils.toWei('1', 'ether'), '0xYourAddress', '0xYourPrivateKey');
```

**移除 WJU-USDT 流动性（Solidity）**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.6;

interface IJUV2Router02 {
    function removeLiquidityETH(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external returns (uint amountToken, uint amountETH);
}

interface IJUV2Pair {
    function approve(address spender, uint value) external returns (bool);
}

contract LiquidityExample {
    address constant ROUTER = 0x6A647E09193a130b0CccBF26A1CF442491bDeCc0;
    address constant USDT = 0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02;
    address constant WJU = 0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3;
    IJUV2Router02 public router = IJUV2Router02(ROUTER);

    function removeLiquidity(uint liquidity, uint amountUSDTMin, uint amountWJUMin) external {
        address pairAddress = pairFor(USDT, WJU); // 计算交易对地址
        IJUV2Pair(pairAddress).approve(ROUTER, liquidity);

        router.removeLiquidityETH(
            USDT,
            liquidity,
            amountUSDTMin,
            amountWJUMin,
            msg.sender,
            block.timestamp + 1200
        );
    }

    // 简化的 pairFor 函数（实际应使用 JUV2Library）
    function pairFor(address tokenA, address tokenB) internal pure returns (address pair) {
        // 请参考 JUV2Library 的 pairFor 实现
        pair = address(uint(keccak256(abi.encodePacked(
            hex'ff',
            0x66682281BdfeC17fCBcae0480C77edFb0c489339,
            keccak256(abi.encodePacked(tokenA < tokenB ? tokenA : tokenB, tokenA < tokenB ? tokenB : tokenA)),
            hex'2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b'
        ))));
    }
}
```

### 注意事项

* **比例匹配**：添加流动性时需按交易对当前储备比例提供代币。
* **LP 代币**：添加流动性后会收到 LP 代币，移除时需批准 Router 使用。
* **最小金额**：设置 `amountAMin` 和 `amountBMin` 以防滑点。
* **首次添加**：若交易对无流动性，初始比例由首次添加决定。

***

### ABI 数据

以下是 Router 的部分关键 ABI：

```json
[
  {
    "inputs": [
      {"internalType": "address", "name": "token", "type": "address"},
      {"internalType": "uint256", "name": "amountTokenDesired", "type": "uint256"},
      {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"},
      {"internalType": "address", "name": "to", "type": "address"},
      {"internalType": "uint256", "name": "deadline", "type": "uint256"}
    ],
    "name": "addLiquidityETH",
    "outputs": [
      {"internalType": "uint256", "name": "amountToken", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETH", "type": "uint256"},
      {"internalType": "uint256", "name": "liquidity", "type": "uint256"}
    ],
    "stateMutability": "payable",
    "type": "function"
  },
  {
    "inputs": [
      {"internalType": "address", "name": "token", "type": "address"},
      {"internalType": "uint256", "name": "liquidity", "type": "uint256"},
      {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"},
      {"internalType": "address", "name": "to", "type": "address"},
      {"internalType": "uint256", "name": "deadline", "type": "uint256"}
    ],
    "name": "removeLiquidityETH",
    "outputs": [
      {"internalType": "uint256", "name": "amountToken", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETH", "type": "uint256"}
    ],
    "stateMutability": "nonpayable",
    "type": "function"
  }
]
```

