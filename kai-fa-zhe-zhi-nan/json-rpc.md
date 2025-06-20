# JSON-RPC

本篇文档介绍 JuChain 节点搭建与 JSON-RPC API 调用的常见问题与操作方法，适用于开发者快速集成与排查。

***

### 节点搭建

#### 1. 支持的操作系统

JuChain 节点支持主流 Linux 发行版（如 Ubuntu、CentOS）、macOS 以及 Windows（推荐使用 WSL）。

#### 2. 获取节点程序



* **端点**：`https://testnet-rpc.juchain.org`
*   **示例**（查询区块高度）：

    ```bash
    curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' https://testnet-rpc.juchain.org
    ```
*   **响应**：

    ```json
    {"jsonrpc":"2.0","id":1,"result":"0x1a2b3c"}
    ```

#### 3. 快速启动全节点

以 Linux 为例：

```bash
wget https://github.com/juchain/juchain/releases/download/vX.Y.Z/juchain-linux-amd64.tar.gz
tar -xzvf juchain-linux-amd64.tar.gz
cd juchain
./juchain --network mainnet
```

* `--network mainnet` 表示主网，测试网可用 `--network testnet`。

#### 4. 节点同步时间

首次全量同步时间取决于网络带宽和硬盘性能，通常需要数小时到一天不等。建议使用 SSD 硬盘以提升同步速度。

#### 5. 查看节点同步状态

启动节点后，终端会持续输出区块高度等信息。你也可以通过 RPC API 查询同步状态：

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' https://testnet-rpc.juchain.org
```

#### 6. 开放节点 RPC 服务

启动节点时添加参数：

```bash
./juchain --rpc --rpcaddr 0.0.0.0 --rpcport 8545
```

* `--rpc` 启用 RPC 服务
* `--rpcaddr` 设置监听地址
* `--rpcport` 设置端口

#### 7. 节点常见问题排查

* **端口被占用**：请检查 8545、30303 等端口是否被其他程序占用。
* **同步卡住**：可尝试重启节点或检查网络连接。
* **数据损坏**：可尝试删除数据目录后重新同步（注意备份密钥）。

***

### JSON-RPC API 调用

#### 1. 支持的 API

JuChain 兼容以太坊 JSON-RPC API，常用接口包括 `eth_blockNumber`、`eth_getBalance`、`eth_sendRawTransaction` 等。

#### 2. 通过 curl 调用 RPC

**查询区块高度：**

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' https://rpc.juchain.org
```

**查询账户余额：**

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xYourAddress","latest"],"id":1}' https://rpc.juchain.org
```

#### 3. Web3.js/ethers.js 连接示例

**Web3.js：**

```js
const Web3 = require('web3');
const web3 = new Web3('https://rpc.juchain.org');
web3.eth.getBlockNumber().then(console.log);
```

**ethers.js：**

```js
import { ethers } from "ethers";
const provider = new ethers.providers.JsonRpcProvider("https://rpc.juchain.org");
provider.getBlockNumber().then(console.log);
```

#### 4. WebSocket 支持

主网 WebSocket 地址：`wss://ws.juchain.org`\
测试网 WebSocket 地址：`ws://testnet-ws.juchain.org`

#### 5. 监听链上事件

以 ethers.js 为例：

```js
const provider = new ethers.providers.WebSocketProvider("wss://ws.juchain.org");
provider.on("block", (blockNumber) => {
  console.log("New block:", blockNumber);
});
```

#### 6. API 调用常见错误及排查

* **返回 403/401**：请检查 RPC 节点是否对外开放，或是否有访问权限限制。
* **超时/无响应**：检查网络连接，或节点是否同步完成。
* **数据格式错误**：请确保请求体为标准 JSON-RPC 格式。

***

### JuChain 网络信息

* 主网 RPC: `https://rpc.juchain.org`
* 主网 WebSocket: `wss://ws.juchain.org`
* 主网 Chain ID: `210000`
* 测试网 RPC: `https://testnet-rpc.juchain.org`
* 测试网 WebSocket: `ws://testnet-ws.juchain.org`
* 测试网 Chain ID: `202599`

***

如需更详细的节点搭建脚本、API 示例或遇到具体报错，欢迎参考开发者指南或联系官方社区支持。
