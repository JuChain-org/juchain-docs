# Technical Reference

#### Overview

The "Technical Reference" section provides developers with detailed technical insights into JuChain, including API specifications, smart contract ABIs, consensus mechanism parameters, and other critical configuration details. This chapter serves as a technical handbook for developers, aiding in interactions with the JuChain Testnet or Mainnet, debugging applications, and optimizing development workflows. JuChain’s design emphasizes high performance, low cost, and EVM compatibility, with all reference information built around these principles.

***

#### 1. JSON-RPC API Reference

JuChain offers a standard JSON-RPC API for blockchain interaction. Below are commonly used methods and their parameters, applicable to the Testnet (`https://testnet-rpc.juchain.org`) and Mainnet (to be provided upon launch).

**1.1 Configuration**

* **Endpoint**: `https://testnet-rpc.juchain.org`
* **Protocol**: HTTP/HTTPS or WebSocket (`wss://testnet-rpc.juchain.org`)

**1.2 Common Methods**

**eth\_blockNumber**

* **Description**: Returns the current block height.
*   **Request**:

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_blockNumber",
      "params": [],
      "id": 1
    }
    ```
*   **Response**:

    ```json
    {
      "jsonrpc": "2.0",
      "id": 1,
      "result": "0x1a2b3c" // Hexadecimal block height
    }
    ```
*   **Example (cURL)**:

    ```bash
    curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' https://testnet-rpc.juchain.org
    ```

**eth\_getBalance**

* **Description**: Queries the JU balance of a specified address.
* **Parameters**:
  * `address`: 20-byte address (e.g., `"0x1234567890abcdef1234567890abcdef12345678"`)
  * `block`: Block tag (e.g., `"latest"`)
*   **Request**:

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_getBalance",
      "params": ["0x1234567890abcdef1234567890abcdef12345678", "latest"],
      "id": 2
    }
    ```
*   **Response**:

    ```json
    {
      "jsonrpc": "2.0",
      "id": 2,
      "result": "0x16345785d8a0000" // Balance in wei
    }
    ```

**eth\_sendRawTransaction**

* **Description**: Submits a signed transaction.
* **Parameters**:
  * `rawTx`: Signed transaction data (hex string)
*   **Request**:

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_sendRawTransaction",
      "params": ["0xf86c018502540be400825208941234567890abcdef1234567890abcdef1234567888016345785d8a00008025a0..."],
      "id": 3
    }
    ```
*   **Response**:

    ```json
    {
      "jsonrpc": "2.0",
      "id": 3,
      "result": "0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890" // Transaction hash
    }
    ```

**eth\_call**

* **Description**: Executes a read-only smart contract call.
* **Parameters**:
  * `transaction`: Object with `to` (contract address) and `data` (method signature and parameters)
  * `block`: Block tag
*   **Request**:

    ```json
    {
      "jsonrpc": "2.0",
      "method": "eth_call",
      "params": [{"to": "0xContractAddress", "data": "0x6d4ce63c"}, "latest"],
      "id": 4
    }
    ```
*   **Response**:

    ```json
    {
      "jsonrpc": "2.0",
      "id": 4,
      "result": "0x000000000000000000000000000000000000000000000000000000000000002a" // Return value (e.g., 42)
    }
    ```

**1.3 Full API List**

* Additional methods (e.g., `eth_getTransactionByHash`, `eth_getBlockByNumber`) are Ethereum-compatible. Refer to the [Ethereum JSON-RPC Documentation](https://ethereum.org/en/developers/docs/apis/json-rpc/).

***

#### 2. Smart Contract ABI

Below is an example ABI for JuChain’s core contracts, facilitating interaction with JU tokens and other functionalities.

**2.1 JU Token ABI**

* **Contract Address**: TBD (to be updated upon Mainnet launch).
* **Standard**: ERC-20 (if applicable; JU as a native token may not require this in all contexts).
*   **Core Methods**:

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

**Note**: JU is JuChain’s native token, so some interactions (e.g., transfers) may use native ETH-like calls (`msg.value`) rather than ERC-20 methods unless wrapped (e.g., WJU).

***

#### 3. JPoSA Parameters

JPoSA (JuChain Proof of Stake Authorization) is JuChain’s consensus mechanism. Key parameters are listed below:

**3.1 Consensus Parameters**

* **Block Time**: 1 second
* **Transaction Finality**: 2-3 seconds (2-3 blocks)
* **Max Core Validators**: 21
* **Minimum Staking Requirement**: 10,000 JU (adjustable on Testnet)
* **Validation Cycle**: Default 7,200 blocks (\~6 hours), adjustable between 3,600-14,400 blocks

**3.2 Fault Tolerance**

* **Byzantine Fault Tolerance**: Tolerates up to 1/3 validator failures or malicious behavior (max 7 core validators).
* **Penalty Mechanism**: Missing 100 consecutive blocks results in a 5% JU stake deduction.

**3.3 Rewards**

* **Block Reward**: 2 JU per block (Testnet), dynamically adjusted
* **Delegation Reward Split**: 70:30 between validators and delegators

***

#### 4. Network Configuration

**4.1 Testnet**

* **RPC Endpoint**: `https://testnet-rpc.juchain.org`
* **WebSocket**: `wss://testnet-ws.juchain.org`
* **Network ID**: 202599
* **Currency Symbol**: JU
* **Block Explorer**: `https://explorer-testnet.juchain.org`

**4.2 Gas Parameters**

* **Gas Price**: Minimum 1 gwei (10⁹ wei), recommended 1-5 gwei
* **Gas Limits**:
  * Per Transaction: Default 5,500,000
  * Per Block: Default 30,000,000
* **Unit Conversion**: 1 JU = 10¹⁸ wei

**4.3 Node Requirements**

* **Full Node**:
  * CPU: 4 cores
  * Memory: 8 GB
  * Storage: 500 GB SSD
* **Validator Node**: Increase memory to 16 GB, requires 24/7 uptime

***

#### 5. Error Codes

Common errors and their resolutions:

| **Error Code** | **Description**      | **Resolution**                            |
| -------------- | -------------------- | ----------------------------------------- |
| -32000         | Insufficient funds   | Check account balance and acquire JU      |
| -32603         | Out of gas           | Increase transaction gas limit            |
| 429            | Too Many Requests    | Wait 1 minute or check rate limits        |
| 0x             | Transaction reverted | Verify contract logic or input parameters |

***

#### 6. Tools and SDKs

**6.1 Truffle Configuration**

*   **Example** (`truffle-config.js`):

    ```javascript
    const HDWalletProvider = require("@truffle/hdwallet-provider");
    module.exports = {
      networks: {
        juchain_testnet: {
          provider: () => new HDWalletProvider("mnemonic", "https://testnet-rpc.juchain.org"),
          network_id: 202599,
          gas: 5500000,
          gasPrice: 1000000000,
        },
      },
    };
    ```

***

#### 7. Additional References

* **EVM Compatibility**: JuChain supports all Ethereum opcodes. See the [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf).
* **Block Structure**:
  * Header: Version, previous hash, timestamp, Merkle root
  * Body: Transaction list
* **State Database**: Based on Merkle Patricia Trie (MPT). Refer to [Ethereum Trie Documentation](https://ethereum.org/en/developers/docs/data-structures-and-encoding/).

***

#### Summary

This technical reference equips developers with detailed information for interacting with JuChain, covering API calls, contract ABIs, consensus parameters, and network configurations. Paired with the \[Developer Guide] and \[Tutorials and Examples], you can efficiently build, deploy, and troubleshoot dApps. For further assistance, visit the \[Community and Support] page.
