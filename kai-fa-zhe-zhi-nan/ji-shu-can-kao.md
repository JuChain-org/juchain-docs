# 技术参考

### 概述

“技术参考”部分为开发者提供了 JuChain 的核心技术细节，包括 API 接口、智能合约 ABI、共识机制参数和其他关键配置信息。本章节旨在作为开发者的技术手册，帮助您与 JuChain 测试网或主网交互、调试应用并优化开发流程。JuChain 的设计注重高性能、低成本和 EVM 兼容性，所有参考信息均基于此特性构建。

***

### 1. JSON-RPC API 参考

JuChain 提供标准的 JSON-RPC API，支持与区块链交互。以下是常用方法及其参数，适用于测试网（`https://testnet-rpc.juchain.org`）和主网（未来上线后提供）。

#### 1.1 配置

* **端点**：`https://testnet-rpc.juchain.org`
* **协议**：HTTP/HTTPS 或 WebSocket（`wss://testnet-rpc.juchain.org`）

#### 1.2 常用方法

**eth\_blockNumber**

* **描述**：返回当前区块高度。
*   **请求**：

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_blockNumber",
      "params": [],
      "id": 1
    }
    ```
*   **响应**：

    ```json
    {
      "jsonrpc": "2.0",
      "id": 1,
      "result": "0x1a2b3c" // 十六进制区块高度
    }
    ```
*   **示例**（cURL）：

    ```bash
    curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' https://testnet-rpc.juchain.org
    ```

**eth\_getBalance**

* **描述**：查询指定地址的 JU 余额。
* **参数**：
  * `address`：20 字节地址（如 `"0x1234567890abcdef1234567890abcdef12345678"`）。
  * `block`：区块标签（如 `"latest"`）。
*   **请求**：

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_getBalance",
      "params": ["0x1234567890abcdef1234567890abcdef12345678", "latest"],
      "id": 2
    }
    ```
*   **响应**：

    ```json
    {
      "jsonrpc": "2.0",
      "id": 2,
      "result": "0x16345785d8a0000" // 余额（wei）
    }
    ```

**eth\_sendRawTransaction**

* **描述**：发送已签名的交易。
* **参数**：
  * `rawTx`：签名后的交易数据（十六进制字符串）。
*   **请求**：

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_sendRawTransaction",
      "params": ["0xf86c018502540be400825208941234567890abcdef1234567890abcdef1234567888016345785d8a00008025a0..."],
      "id": 3
    }
    ```
*   **响应**：

    ```json
    {
      "jsonrpc": "2.0",
      "id": 3,
      "result": "0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890" // 交易哈希
    }
    ```

**eth\_call**

* **描述**：调用智能合约的只读方法。
* **参数**：
  * `transaction`：包含 `to`（合约地址）和 `data`（方法签名及参数）。
  * `block`：区块标签。
*   **请求**：

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_call",
      "params": [{"to": "0xContractAddress", "data": "0x6d4ce63c"}, "latest"],
      "id": 4
    }
    ```
*   **响应**：

    ```json
    {
      "jsonrpc": "2.0",
      "id": 4,
      "result": "0x000000000000000000000000000000000000000000000000000000000000002a" // 返回值（如 42）
    }
    ```

#### 1.3 完整 API 列表

* 更多方法（如 `eth_getTransactionByHash`、`eth_getBlockByNumber`）与 Ethereum API 兼容，请参考 [Ethereum JSON-RPC 文档](https://ethereum.org/en/developers/docs/apis/json-rpc/)。

***

### 2. 智能合约 ABI

以下是 JuChain 核心合约的 ABI 示例，用于与 JU 代币和其他功能交互。

#### 2.1 JU 代币 ABI

* **合约地址**：TODO。
* **标准**：ERC-20。
*   **核心方法**：

    ```json
    [
      {
        "constant": true,
        "inputs": [{"name": "_owner", "type": "address"}],
        "name": "balanceOf",
        "outputs": [{"name": "balance", "type": "uint256"}],
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {"name": "_to", "type": "address"},
          {"name": "_value", "type": "uint256"}
        ],
        "name": "transfer",
        "outputs": [{"name": "success", "type": "bool"}],
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {"name": "_spender", "type": "address"},
          {"name": "_value", "type": "uint256"}
        ],
        "name": "approve",
        "outputs": [{"name": "success", "type": "bool"}],
        "type": "function"
      }
    ]
    ```

***

### 3. JPoSA 参数

JPoSA（JuChain Proof of Stake Authorization）是 JuChain 的共识机制，以下是关键参数：

#### 3.1 共识参数

* **区块时间**：1 秒。
* **交易最终确认**：2-3 秒（2-3 个区块）。
* **最大核心验证者数量**：21。
* **最小质押要求**：10,000 JU（测试网可调整）。
* **验证周期**：默认 7200 区块（约 6 小时），动态调整范围 3600-14400 区块。

#### 3.2 容错性

* **拜占庭容错**：可容忍 1/3 验证者故障或恶意行为（最多 7 个核心验证者）。
* **惩罚机制**：连续 100 个区块未出块，扣除 5% 质押 JU。

#### 3.3 奖励

* **区块奖励**：每区块 2 JU（测试网），动态调整。
* **委托奖励分配**：验证者与委托者按 70:30 比例分配。

***

### 4. 网络配置

#### 4.1 测试网

* **RPC 端点**：`https://testnet-rpc.juchain.org`
* **WebSocket**：`wss://testnet-ws.juchain.org`
* **网络 ID**：202599
* **货币符号**：JU
* **区块浏览器**：`https://testnet-explorer.juchain.org`

#### 4.2 Gas 参数

* **Gas 价格**：最低 1 gwei（10⁹ wei），建议 1-5 gwei。
* **Gas 限制**：
  * 单笔交易：默认 5,500,000。
  * 区块：默认 30,000,000。
* **单位换算**：1 JU = 10¹⁸ wei。

#### 4.3 节点要求

* **全节点**：
  * CPU：4 核
  * 内存：8 GB
  * 存储：500 GB SSD
* **验证者节点**：内存增至 16 GB，需 24/7 在线。

***

### 5. 错误码

以下是常见错误及其解决方法：

| **错误码** | **描述**                    | **解决方法**           |
| ------- | ------------------------- | ------------------ |
| -32000  | 余额不足（Insufficient funds）  | 检查账户余额并获取更多 JU 代币。 |
| -32603  | Gas 不足（Out of gas）        | 提高交易的 Gas 限制。      |
| 429     | 请求过于频繁（Too Many Requests） | 等待 1 分钟后重试，或检查限额。  |
| 0x      | 交易失败（Reverted）            | 检查合约逻辑或输入参数。       |

***

### 6. 工具与 SDK

#### 6.1 Truffle 配置

*   **示例**（`truffle-config.js`）：

    ```javascript
    const HDWalletProvider = require("@truffle/hdwallet-provider");
    module.exports = {
      networks: {
        juchain_testnet: {
          provider: () => new HDWalletProvider("助记词", "https://testnet-rpc.juchain.org"),
          network_id: 202599,
          gas: 5500000,
          gasPrice: 1000000000,
        },
      },
    };
    ```

***

### 7. 其他参考

* **EVM 兼容性**：JuChain 支持所有以太坊操作码（Opcodes），参考 [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)。
* **区块结构**：
  * 头部：版本、前哈希、时间戳、Merkle 根。
  * 主体：交易列表。
* **状态数据库**：基于 MPT，参考 [Ethereum Trie](https://ethereum.org/en/developers/docs/data-structures-and-encoding/)。

***

### 总结

本技术参考为开发者提供了与 JuChain 交互所需的详细技术信息，包括 API 调用、合约 ABI、共识参数和网络配置。结合\[开发者指南]和\[教程与示例]，您可以快速构建、部署和调试 dApp。如需更多支持，请访问\[社区与支持]页面。
