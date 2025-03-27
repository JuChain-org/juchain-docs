# 去中心化交易所

JuChain 提供了一套基于 Uniswap V2 模型的去中心化交易所（DEX）功能，允许用户进行代币兑换（Swap）和提供流动性（Liquidity）。本文档旨在指导开发者如何使用 Javascript (Web3.js/ethers.js) 或 Solidity 与核心 DEX 合约进行交互。

**核心组件:**

* **路由器 (`JUV2Router02`)**: 用户交互的主要入口合约（执行 Swap、添加/移除流动性）。
* **工厂 (`JUV2Factory`)**: 创建和管理交易对合约。
* **交易对 (`JUV2Pair`)**: 代表一个特定的代币交易对（例如 WJU-USDT），并持有流动性资金池。
* **WJU (Wrapped JU)**: 原生 JU 代币的封装版本，符合 ERC20 标准，以便与 DEX 交互。
* **USDT**: 网络上的一个示例 ERC20 代币。

### 1. 前置准备与网络设置

* **网络**: JuChain 测试网 (未来可能适用于主网)
  * **RPC URL**: `https://testnet-rpc.juchain.org` (请以官方渠道信息为准)
  * **链 ID (Chain ID)**: `202599`
  * **区块浏览器**: `https://explorer-testnet.juchain.org`
* **工具**: Node.js 环境及 Web3.js 或 ethers.js 库，或者 Solidity 开发环境 (如 Remix, Hardhat, Truffle)。
* **账户**: 一个拥有原生 JU（用于支付 Gas 费）以及相关代币（如 WJU, USDT）的外部账户 (EOA)。

### 2. 核心合约地址与详情

| 合约               | 地址                                                                   | Solidity 版本 | 说明                                                              |
| ---------------- | -------------------------------------------------------------------- | ----------- | --------------------------------------------------------------- |
| **WJU**          | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`                         | `0.4.26`    | 封装版 JU。使用 `deposit()` (payable) 封装原生 JU，`withdraw()` 解封装回原生 JU。 |
| **USDT**         | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`                         | `0.8.8`     | 标准 ERC20 代币 (基于 OpenZeppelin)。                                  |
| **工厂 (Factory)** | `0x66682281BdfeC17fCBcae0480C77edFb0c489339`                         | `0.5.16`    | 部署并追踪 `JUV2Pair` 交易对合约。                                         |
| **路由器 (Router)** | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0`                         | `0.6.6`     | **Swap 和流动性管理的主要交互入口。** 内部将 WJU 地址视为其 `WETH` 地址。                |
| **交易对初始化哈希**     | `0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b` | N/A         | 用于计算交易对地址 (参考 `JUV2Library.pairFor` 逻辑)。                        |

**重要提示:** 路由器 (`JUV2Router02`) 合约内部使用了 `WETH` 地址的概念。在 JuChain 上，这个 `WETH` 地址**必须指向 WJU 的地址** (`0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`)，这样路由器中带有 `ETH` 后缀的函数 (例如 `addLiquidityETH`, `swapExactETHForTokens`) 才能正确地处理封装后的 JU 代币。本文档假设此配置是正确的。

### 3. 与 WJU 交互 (封装/解封装)

由于 DEX 主要与 ERC20 代币交互，您需要先将原生 JU 封装成 WJU。

*   **封装 JU (Wrap)**: 向 WJU 合约发送原生 JU。可以直接发送（触发 fallback 函数），或者调用其 `deposit()` payable 函数。

    ```javascript
    // Web3.js 示例: 封装 1 个原生 JU
    const { Web3 } = require('web3');
    const web3 = new Web3('https://testnet-rpc.juchain.org');
    const wjuAddress = '0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3';
    const wjuABI = [ /* 参见文档第 7 节 WJU ABI */ ];
    const wjuContract = new web3.eth.Contract(wjuABI, wjuAddress);
    const account = '0xYourAddress'; // 您的账户地址
    const privateKey = '0xYourPrivateKey'; // 安全地管理您的私钥!
    const amountToWrap = web3.utils.toWei('1', 'ether'); // 要封装的数量 (1 JU)

    async function wrapJU() {
      const tx = {
        to: wjuAddress, value: amountToWrap, gas: 300000, from: account
      };
      const signedTx = await web3.eth.accounts.signTransaction(tx, privateKey);
      const receipt = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
      console.log('JU 封装成功，交易哈希:', receipt.transactionHash);
    }
    wrapJU();
    ```
*   **解封装 WJU (Unwrap)**: 调用 WJU 合约的 `withdraw()` 函数，指定要换回原生 JU 的 WJU 数量。

    ```javascript
    // Web3.js 示例: 解封装 1 WJU
    async function unwrapWJU() {
      const amountToUnwrap = web3.utils.toWei('1', 'ether'); // 要解封装的数量 (1 WJU)
      const tx = wjuContract.methods.withdraw(amountToUnwrap);
      const gas = await tx.estimateGas({ from: account });
      const data = tx.encodeABI();
      const signedTx = await web3.eth.accounts.signTransaction({
          to: wjuAddress, data, gas, from: account
      }, privateKey);
      const receipt = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
      console.log('WJU 解封装成功，交易哈希:', receipt.transactionHash);
    }
    unwrapWJU();
    ```

### 4. Swap (兑换) 功能 (使用路由器)

Swap 功能允许用户通过已存在的流动性池将一种代币兑换成另一种代币。

**关键的路由器 Swap 函数:** (ABI 见第 7 节)

* `swapExactTokensForTokens`: 用 **精确数量** 的输入代币，兑换 **至少** 某个数量的输出代币。
* `swapTokensForExactTokens`: 用 **最多** 某个数量的输入代币，兑换 **精确数量** 的输出代币。(ABI 片段未包含)
* `swapExactETHForTokens`: 用 **精确数量** 的 **WJU** (作为 ETH)，兑换 **至少** 某个数量的输出代币。
* `swapTokensForExactETH`: 用 **最多** 某个数量的输入代币，兑换 **精确数量** 的 **WJU** (作为 ETH)。(ABI 片段未包含)
* `swapExactTokensForETH`: 用 **精确数量** 的输入代币，兑换 **至少** 某个数量的 **WJU** (作为 ETH)。
* `swapETHForExactTokens`: 用 **最多** 某个数量的 **WJU** (随交易发送)，兑换 **精确数量** 的输出代币。(ABI 片段未包含)
* `*SupportingFeeOnTransferTokens`: 为支持“转账即收费”代币设计的对应函数变种。(ABI 片段未包含)

**Swap 步骤:**

1. **批准路由器 (Approve Router)**: 在兑换 ERC20 代币之前（例如 USDT，或者 WJU 作为 `swapExactTokensFor*` 的 _输入_ 代币时），您必须先调用该代币合约的 `approve(routerAddress, amount)` 函数，授权路由器代表您花费该代币 (使用代币 ABI，见第 7 节)。
2. **调用 Swap 函数**: 调用路由器合约上对应的 Swap 函数 (使用 Router ABI，见第 7 节)。

**示例 1: 用精确数量的 WJU 兑换 USDT (使用 `swapExactETHForTokens`)**

```javascript
// --- Web3.js 示例 ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

const routerAddress = '0x6A647E09193a130b0CccBF26A1CF442491bDeCc0';
const wjuAddress = '0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3';
const usdtAddress = '0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02';
const routerABI = [ /* 参见文档第 7 节 Router ABI 片段 */ ];
const usdtABI = [ /* 参见文档第 7 节 USDT ABI */ ]; // 虽然此例不用，但获取 USDT 余额等会用到

const routerContract = new web3.eth.Contract(routerABI, routerAddress);
const account = '0xYourAddress';
const privateKey = '0xYourPrivateKey';

async function swapWJUForUSDT(wjuAmountIn, usdtAmountOutMin) {
  const path = [wjuAddress, usdtAddress];
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10;

  const tx = routerContract.methods.swapExactETHForTokens(
    usdtAmountOutMin, path, account, deadline
  );
  const gas = await tx.estimateGas({ from: account, value: wjuAmountIn });
  const data = tx.encodeABI();
  const signedTx = await web3.eth.accounts.signTransaction({
      to: routerAddress, data, gas, value: wjuAmountIn, from: account
  }, privateKey);
  const receipt = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
  console.log(`Swap WJU -> USDT 成功，交易哈希: ${receipt.transactionHash}`);
}

const oneWJU = web3.utils.toWei('1', 'ether');
const pointNineUSDT = web3.utils.toWei('0.9', 'ether'); // 假设 USDT 18 位小数
swapWJUForUSDT(oneWJU, pointNineUSDT);
```

**示例 2: 用精确数量的 USDT 兑换 WJU (使用 `swapExactTokensForETH`)**

```javascript
// --- Web3.js 示例 ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// 地址同上...
const routerABI = [ /* 参见文档第 7 节 Router ABI 片段 */ ];
const usdtABI = [ /* 参见文档第 7 节 USDT ABI */ ];

const routerContract = new web3.eth.Contract(routerABI, routerAddress);
const usdtContract = new web3.eth.Contract(usdtABI, usdtAddress);
const account = '0xYourAddress';
const privateKey = '0xYourPrivateKey';

async function swapUSDTForWJU(usdtAmountIn, wjuAmountOutMin) {
  const path = [usdtAddress, wjuAddress];
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10;

  // 1. 批准
  console.log(`正在批准路由器花费 ${web3.utils.fromWei(usdtAmountIn)} USDT...`);
  const approveTx = usdtContract.methods.approve(routerAddress, usdtAmountIn);
  const approveGas = await approveTx.estimateGas({ from: account });
  const approveData = approveTx.encodeABI();
  const signedApproveTx = await web3.eth.accounts.signTransaction({
      to: usdtAddress, data: approveData, gas: approveGas, from: account
  }, privateKey);
  const approveReceipt = await web3.eth.sendSignedTransaction(signedApproveTx.rawTransaction);
  console.log(`USDT 批准成功: ${approveReceipt.transactionHash}`);

  // 2. Swap
  console.log(`正在用 ${web3.utils.fromWei(usdtAmountIn)} USDT 兑换 WJU...`);
  const swapTx = routerContract.methods.swapExactTokensForETH(
    usdtAmountIn, wjuAmountOutMin, path, account, deadline
  );
  const swapGas = await swapTx.estimateGas({ from: account });
  const swapData = swapTx.encodeABI();
  const signedSwapTx = await web3.eth.accounts.signTransaction({
      to: routerAddress, data: swapData, gas: swapGas, from: account
  }, privateKey);
  const swapReceipt = await web3.eth.sendSignedTransaction(signedSwapTx.rawTransaction);
  console.log(`Swap USDT -> WJU 成功，交易哈希: ${swapReceipt.transactionHash}`);
}

const oneUSDT = web3.utils.toWei('1', 'ether');
const pointNineWJU = web3.utils.toWei('0.9', 'ether');
swapUSDTForWJU(oneUSDT, pointNineWJU);
```

**Swap 注意事项:** (同上节)

### 5. 流动性管理 (使用路由器)

用户可以向交易对（如 WJU-USDT）提供流动性，并从该交易对发生的 Swap 中赚取手续费。提供流动性后，用户会收到代表其份额的 LP (Liquidity Provider) 代币。

**关键的路由器流动性函数:** (ABI 见第 7 节)

* `addLiquidity`: 为两个 ERC20 代币组成的交易对添加流动性。
* `addLiquidityETH`: 为涉及 WJU (作为 WETH) 和另一个 ERC20 代币的交易对添加流动性。WJU 通过交易的 `value` 发送。
* `removeLiquidity`: 通过销毁 LP 代币，为两个 ERC20 代币组成的交易对移除流动性，取回两种基础代币。
* `removeLiquidityETH`: 通过销毁 LP 代币，为 WJU (作为 WETH) / ERC20 交易对移除流动性，取回 WJU 和该 ERC20 代币。
* `*WithPermit`: 允许使用链下签名 (EIP-712) 授权移除流动性的函数变种。(ABI 片段未包含)

**添加流动性步骤:**

1. **批准路由器 (Approve Router)**: 路由器需要权限来转移您提供的 ERC20 代币。对您要提供的 _每种_ ERC20 代币（除非是通过 `addLiquidityETH` 提供 WJU）调用 `approve(routerAddress, amount)` (使用代币 ABI，见第 7 节)。
2. **调用添加函数**: 调用 `addLiquidity` 或 `addLiquidityETH`。您需要指定 _期望_ 添加的数量 (`amountADesired`, `amountBDesired`)，但合约会根据当前池子的价格比例计算实际需要添加的数量。设置最小数量 (`amountAMin`, `amountBMin`, `amountETHMin`) 以控制滑点 (使用 Router ABI，见第 7 节)。

**示例: 为 WJU-USDT 添加流动性 (使用 `addLiquidityETH`)**

```javascript
// --- Web3.js 示例 ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// 地址同上...
const routerABI = [ /* 参见文档第 7 节 Router ABI 片段 */ ];
const usdtABI = [ /* 参见文档第 7 节 USDT ABI */ ];

const routerContract = new web3.eth.Contract(routerABI, routerAddress);
const usdtContract = new web3.eth.Contract(usdtABI, usdtAddress);
const account = '0xYourAddress';
const privateKey = '0xYourPrivateKey';

async function addLiquidityWJUUSDT(wjuAmountDesired, usdtAmountDesired, wjuAmountMin, usdtAmountMin) {
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10;

  // 1. 批准 USDT
  console.log(`正在批准路由器花费 ${web3.utils.fromWei(usdtAmountDesired)} USDT...`);
  const approveTx = usdtContract.methods.approve(routerAddress, usdtAmountDesired);
  const approveGas = await approveTx.estimateGas({ from: account });
  const approveData = approveTx.encodeABI();
  const signedApproveTx = await web3.eth.accounts.signTransaction({
      to: usdtAddress, data: approveData, gas: approveGas, from: account
  }, privateKey);
  const approveReceipt = await web3.eth.sendSignedTransaction(signedApproveTx.rawTransaction);
  console.log(`USDT 批准成功: ${approveReceipt.transactionHash}`);

  // 2. 添加流动性
  console.log(`正在添加流动性: 约 ${web3.utils.fromWei(wjuAmountDesired)} WJU 和 ${web3.utils.fromWei(usdtAmountDesired)} USDT...`);
  const addLiqTx = routerContract.methods.addLiquidityETH(
    usdtAddress, usdtAmountDesired, usdtAmountMin, wjuAmountMin, account, deadline
  );
  const addLiqGas = await addLiqTx.estimateGas({ from: account, value: wjuAmountDesired });
  const addLiqData = addLiqTx.encodeABI();
  const signedAddLiqTx = await web3.eth.accounts.signTransaction({
      to: routerAddress, data: addLiqData, gas: addLiqGas, value: wjuAmountDesired, from: account
  }, privateKey);
  const addLiqReceipt = await web3.eth.sendSignedTransaction(signedAddLiqTx.rawTransaction);
  console.log(`添加 WJU-USDT 流动性成功: ${addLiqReceipt.transactionHash}`);
}

const wjuToAdd = web3.utils.toWei('1', 'ether');
const usdtToAdd = web3.utils.toWei('100', 'ether'); // 假设 USDT 18 位小数
const wjuMin = web3.utils.toWei('0.99', 'ether');
const usdtMin = web3.utils.toWei('99', 'ether');
addLiquidityWJUUSDT(wjuToAdd, usdtToAdd, wjuMin, usdtMin);
```

**移除流动性步骤:**

1. **获取交易对地址**: 找到您拥有 LP 代币的交易对合约地址（例如 WJU-USDT 交易对）。可以通过工厂合约的 `getPair(tokenA, tokenB)` 函数查询 (使用 Factory ABI，见第 7 节)，或者使用 `JUV2Library.pairFor` 逻辑计算。
2. **批准路由器花费 LP 代币**: 您需要授权路由器销毁您的 LP 代币。调用 **交易对合约** (Pair Contract) 的 `approve(routerAddress, liquidityAmount)` 函数 (使用 Pair ABI，见第 7 节)。
3. **调用移除函数**: 调用 `removeLiquidity` 或 `removeLiquidityETH`，指定要销毁的 LP 代币数量 (`liquidity`)，以及您期望取回的两种基础代币的最小数量 (`amountAMin`, `amountBMin`, `amountETHMin`) (使用 Router ABI，见第 7 节)。

**示例: 移除 WJU-USDT 流动性 (使用 `removeLiquidityETH`)**

```javascript
// --- Web3.js 示例 ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// 地址同上...
const routerABI = [ /* 参见文档第 7 节 Router ABI 片段 */ ];
const factoryABI = [ /* 参见文档第 7 节 Factory ABI */ ];
const pairABI = [ /* 参见文档第 7 节 JUV2Pair ABI */ ];

const routerContract = new web3.eth.Contract(routerABI, routerAddress);
const factoryContract = new web3.eth.Contract(factoryABI, factoryAddress);
const account = '0xYourAddress';
const privateKey = '0xYourPrivateKey';

async function removeLiquidityWJUUSDT(liquidityAmount, usdtAmountMin, wjuAmountMin) {
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10;

  // 1. 获取 Pair 地址
  console.log("正在查询交易对地址...");
  const pairAddress = await factoryContract.methods.getPair(wjuAddress, usdtAddress).call();
  if (pairAddress === '0x0000000000000000000000000000000000000000') {
    console.error("错误：WJU-USDT 交易对不存在！"); return;
  }
  console.log(`交易对地址: ${pairAddress}`);
  const pairContract = new web3.eth.Contract(pairABI, pairAddress);

  // 2. 批准 LP 代币
  // !! 确认 LP 代币的小数位数，通常为 18
  console.log(`正在批准路由器花费 ${liquidityAmount} LP 代币...`);
  const approveTx = pairContract.methods.approve(routerAddress, liquidityAmount);
  const approveGas = await approveTx.estimateGas({ from: account });
  const approveData = approveTx.encodeABI();
  const signedApproveTx = await web3.eth.accounts.signTransaction({
      to: pairAddress, data: approveData, gas: approveGas, from: account
  }, privateKey);
  const approveReceipt = await web3.eth.sendSignedTransaction(signedApproveTx.rawTransaction);
  console.log(`LP 代币批准成功: ${approveReceipt.transactionHash}`);

  // 3. 移除流动性
  console.log(`正在移除 ${liquidityAmount} 数量的流动性...`);
  const removeLiqTx = routerContract.methods.removeLiquidityETH(
    usdtAddress, liquidityAmount, usdtAmountMin, wjuAmountMin, account, deadline
  );
  const removeLiqGas = await removeLiqTx.estimateGas({ from: account });
  const removeLiqData = removeLiqTx.encodeABI();
  const signedRemoveLiqTx = await web3.eth.accounts.signTransaction({
      to: routerAddress, data: removeLiqData, gas: removeLiqGas, from: account
  }, privateKey);
  const removeLiqReceipt = await web3.eth.sendSignedTransaction(signedRemoveLiqTx.rawTransaction);
  console.log(`移除 WJU-USDT 流动性成功: ${removeLiqReceipt.transactionHash}`);
}

const oneLpToken = web3.utils.toWei('1', 'ether'); // 假设 LP 代币 18 位小数
const minUsdtExpected = web3.utils.toWei('49', 'ether');
const minWjuExpected = web3.utils.toWei('0.49', 'ether');
removeLiquidityWJUUSDT(oneLpToken, minUsdtExpected, minWjuExpected);
```

**流动性管理注意事项:** (同上节)

### 6. 使用工厂 (Factory)

虽然大部分交互通过路由器进行，但有时您可能需要直接与工厂合约交互 (使用 Factory ABI，见第 7 节)：

* **检查交易对是否存在:** 调用 `getPair(address tokenA, address tokenB)`，如果存在则返回交易对地址，否则返回零地址 (`address(0)`)。
* **确定性地计算交易对地址:** 使用 `pairFor` 逻辑（参见下方 Solidity 示例或路由器代码中的 `JUV2Library`）。这依赖于 `create2` 操作码、工厂地址、两种代币地址（排序后）以及**交易对初始化哈希** (`0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b`)。
* **创建交易对:** 调用 `createPair(address tokenA, address tokenB)`。如果交易对不存在，工厂会部署一个新的交易对合约。通常，在首次为新交易对调用 `addLiquidity` 时，路由器会自动为您调用此函数。

**示例: 计算交易对地址 (Solidity 辅助库)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

library PairUtil {
    function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair) {
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
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

    function getWjuUsdtPairAddress() external pure returns (address) {
        return PairUtil.pairFor(FACTORY, WJU, USDT);
    }
}
```

### 7. 合约 ABI

以下是 WJU、USDT、Factory、Router (关键片段) 和 JUV2Pair (关键部分) 的 ABI，供开发时使用。

**强烈建议:** 从 JuChain 区块浏览器验证并获取指定合约地址的最新、最完整的官方 ABI。

```json
{
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
  "ROUTER_ABI_SNIPPETS": [ // 注意：这是常用函数片段，并非完整 ABI！建议获取完整版。
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
  "JUV2PAIR_ABI": [ // 基于接口和实现推断的关键部分 ABI
    { "inputs": [], "payable": false, "stateMutability": "nonpayable", "type": "constructor" }, // Pair 构造函数通常由 Factory 调用
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
    { "constant": true, "inputs": [], "name": "decimals", "outputs": [ { "internalType": "uint8", "name": "", "type": "uint8" } ], "payable": false, "stateMutability": "pure", "type": "function" }, // JUV2ERC20 has it as pure
    { "constant": true, "inputs": [], "name": "factory", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [], "name": "getReserves", "outputs": [ { "internalType": "uint112", "name": "_reserve0", "type": "uint112" }, { "internalType": "uint112", "name": "_reserve1", "type": "uint112" }, { "internalType": "uint32", "name": "_blockTimestampLast", "type": "uint32" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": false, "inputs": [ { "internalType": "address", "name": "_token0", "type": "address" }, { "internalType": "address", "name": "_token1", "type": "address" } ], "name": "initialize", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": true, "inputs": [], "name": "kLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "mint", "outputs": [ { "internalType": "uint256", "name": "liquidity", "type": "uint256" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": true, "inputs": [], "name": "name", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "payable": false, "stateMutability": "pure", "type": "function" }, // JUV2ERC20 has it as pure
    { "constant": true, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" } ], "name": "nonces", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": false, "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" }, { "internalType": "uint256", "name": "deadline", "type": "uint256" }, { "internalType": "uint8", "name": "v", "type": "uint8" }, { "internalType": "bytes32", "name": "r", "type": "bytes32" }, { "internalType": "bytes32", "name": "s", "type": "bytes32" } ], "name": "permit", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": true, "inputs": [], "name": "price0CumulativeLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [], "name": "price1CumulativeLast", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" } ], "name": "skim", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": false, "inputs": [ { "internalType": "uint256", "name": "amount0Out", "type": "uint256" }, { "internalType": "uint256", "name": "amount1Out", "type": "uint256" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "bytes", "name": "data", "type": "bytes" } ], "name": "swap", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": true, "inputs": [], "name": "symbol", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "payable": false, "stateMutability": "pure", "type": "function" }, // JUV2ERC20 has it as pure
    { "constant": false, "inputs": [], "name": "sync", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": true, "inputs": [], "name": "token0", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [], "name": "token1", "outputs": [ { "internalType": "address", "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": true, "inputs": [], "name": "totalSupply", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" },
    { "constant": false, "inputs": [ { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "transfer", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" },
    { "constant": false, "inputs": [ { "internalType": "address", "name": "from", "type": "address" }, { "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function" }
  ]
}
```

{% hint style="info" %}
在部署到主网之前，请务必在测试网上进行充分的测试。始终参考 JuChain 官方文档和区块浏览器以获取最新、最权威的信息和完整的 ABI。
{% endhint %}

