# JSON-RPC

\
This document introduces the common problems and operation methods of JuChain node construction and JSON-RPC API calls, which is suitable for developers to quickly integrate and troubleshoot.

***

#### &#x20;<a href="#jie-dian-da-jian" id="jie-dian-da-jian"></a>

**1. Supported operating systems**

JuChain nodes support mainstream Linux distributions (such as Ubuntu, CentOS), macOS, and Windows (WSL is recommended).

&#x20;**2. Get the node program**

* **Endpoint** : `https://testnet-rpc.juchain.org`
*   **Example** (query block height):

    ```
    curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' https://testnet-rpc.juchain.org
    ```
*   **response** :

    ```
    {"jsonrpc":"2.0","id":1,"result":"0x1a2b3c"}
    ```

**3. Quickly start a full node**

Take Linux as an example:

```
wget https://github.com/juchain/juchain/releases/download/vX.Y.Z/juchain-linux-amd64.tar.gz
tar -xzvf juchain-linux-amd64.tar.gz
cd juchain
./juchain --network mainnet
```

* `--network mainnet` indicates the main network, and the test network can be used `--network testnet` .

**4. Node synchronization time**

The time for the first full synchronization depends on the network bandwidth and hard disk performance, and usually takes from several hours to a day. It is recommended to use an SSD hard disk to increase the synchronization speed.

**5. Check the node synchronization status**

After starting the node, the terminal will continue to output information such as block height. You can also query the synchronization status through the RPC API:

```
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' https://testnet-rpc.juchain.org
```

**6. Open Node RPC Service**

&#x20;Add parameters when starting the node:

```
./juchain --rpc --rpcaddr 0.0.0.0 --rpcport 8545
```

* `--rpc` Enable RPC service
* `--rpcaddr` Set the listening address
* `--rpcport` set port

&#x20;**7. Troubleshooting common node issues**

* **The port is occupied** : Please check whether ports 8545, 30303, etc. are occupied by other programs.
* **Synchronization stuck** : Try restarting the node or checking the network connection.
* **Data corruption** : Try deleting the data directory and resynchronizing (note the backup key).

***

#### &#x20;JSON-RPC API calls <a href="#jsonrpc-api-diao-yong" id="jsonrpc-api-diao-yong"></a>

**1. Supported APIs**

JuChain is compatible with Ethereum JSON-RPC API. Common interfaces include `eth_blockNumber` , `eth_getBalance` , `eth_sendRawTransaction` , etc.

\
**2. Call RPC via curl**

**Query block height:**

```
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' https://rpc.juchain.org
```

**Check account balance:**

```
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xYourAddress","latest"],"id":1}' https://rpc.juchain.org
```

**3. Web3.js/ethers.js connection example**

**Web3.js：**

```
const Web3 = require('web3');
const web3 = new Web3('https://rpc.juchain.org');
web3.eth.getBlockNumber().then(console.log);
```

**ethers.js：**

```
import { ethers } from "ethers";
const provider = new ethers.providers.JsonRpcProvider("https://rpc.juchain.org");
provider.getBlockNumber().then(console.log);
```

**4. WebSocket support**

Mainnet WebSocket address: `wss://ws.juchain.org` Testnet WebSocket address: `ws://testnet-ws.juchain.org`

**5. Listen to on-chain events**

&#x20;Take ethers.js as an example:

```
const provider = new ethers.providers.WebSocketProvider("wss://ws.juchain.org");
provider.on("block", (blockNumber) => {
  console.log("New block:", blockNumber);
});
```

**6. Common API call errors and troubleshooting**

* **Return 403/401** : Please check whether the RPC node is open to the public or whether there are any access restrictions.
* **Timeout/No response** : Check the network connection or whether the node has completed synchronization.
* **Data format error** : Please ensure that the request body is in the standard JSON-RPC format.

***

#### JuChain Network Information <a href="#juchain-wang-luo-xin-xi" id="juchain-wang-luo-xin-xi"></a>

* Mainnet RPC: `https://rpc.juchain.org`
* Mainnet  WebSocket: `wss://ws.juchain.org`
* Mainnet Chain ID: `210000`
* Testnet RPC: `https://testnet-rpc.juchain.org`
* Testnet WebSocket: `ws://testnet-ws.juchain.org`
* Testnet Chain ID: `202599`

***

If you need more detailed node building scripts, API examples, or encounter specific errors, please refer to the developer guide or contact the official community support.
