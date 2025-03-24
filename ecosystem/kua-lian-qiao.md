# 跨链桥

JuChain 的跨链桥是一个去中心化的服务，允许开发者在不同测试网之间转移资产，如 JuChain 测试网、BSC 测试网（Chapel）和 ETH 测试网（Holesky）。它通过智能合约（主要是 BridgeBank 合约）管理锁定、解锁、铸造和销毁操作，确保资产跨链转移的安全性和效率。

***

#### 测试网设置

要使用跨链桥，开发者需要连接到以下测试网：

* **JuChain 测试网**：
  * RPC：`https://testnet-rpc.juchain.org`
  * 浏览器：`https://explorer-testnet.juchain.org`
* **BSC 测试网（Chapel）**：
  * RPC：`https://rpc.ankr.com/bsc_testnet_chapel/b6aba93f145bbc37b4acc1cda2e7c0b38fff5a09fb21a55c1fb65883d35789bd`
  * 注意：可能需要 API 密钥或认证。
* **ETH 测试网（Holesky）**：
  * RPC：`https://rpc.ankr.com/eth_holesky/b6aba93f145bbc37b4acc1cda2e7c0b38fff5a09fb21a55c1fb65883d35789bd`
  * 浏览器：`https://holesky.etherscan.io`

配置钱包或开发工具时，请使用上述 RPC 端点，确保连接到正确网络。

***

#### 合约地址

以下是跨链桥和相关代币的合约地址：

**JuChain 测试网**

* **BridgeBank 合约**：`0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE`
* **代币**：
  * USDT：`0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D`\
    [查看详情](https://explorer-testnet.juchain.org/token/0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D)
  * tBNB：`0x2598d2e226Ce13288E314569569838bBc6Ff9402`\
    [查看详情](https://explorer-testnet.juchain.org/token/0x2598d2e226Ce13288E314569569838bBc6Ff9402)
  * tETH：`0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672`\
    [查看详情](https://explorer-testnet.juchain.org/token/0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672)

**BSC 测试网（Chapel）**

* **BridgeBank 合约**：`0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6`
* **代币**：
  * USDT：`0xcD1093897a5dB4a9aF153772B35AAA066ab969f3`

**ETH 测试网（Holesky）**

* **BridgeBank 合约**：`0x264960f4bf655c14a74DE1A7fC5AA68E71f71924`\
  [查看详情](https://holesky.etherscan.io/address/0x264960f4bf655c14a74DE1A7fC5AA68E71f71924)
* **代币**：
  * USDT：`0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add`\
    [查看详情](https://holesky.etherscan.io/address/0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add)

***

***

#### 详细报告

**背景与目的**

JuChain 的跨链桥旨在为开发者提供一个测试环境，方便在不同区块链网络之间转移资产，特别是在开发和测试阶段。跨链桥通过智能合约实现资产的锁定、解锁、铸造和销毁，确保跨链操作的安全性和一致性。本文档详细记录了 JuChain 测试网及其与 BSC 测试网（Chapel）和 ETH 测试网（Holesky）的跨链桥信息，供开发者参考。

**测试网设置**

为了使用跨链桥，开发者需要配置开发环境以连接到以下测试网：

* **JuChain 测试网**：
  * RPC 端点：`https://testnet-rpc.juchain.org`，这是官方推荐的连接方式。
  * 区块浏览器：`https://explorer-testnet.juchain.org`，用于查询交易和合约状态。
  * 配置建议：开发者可通过 MetaMask 或 Hardhat/Truffle 设置网络，链 ID 需为 8888（假设，需确认）。
* **BSC 测试网（Chapel）**：
  * RPC 端点：`https://rpc.ankr.com/bsc_testnet_chapel/b6aba93f145bbc37b4acc1cda2e7c0b38fff5a09fb21a55c1fb65883d35789bd`，由 Ankr 提供。
  * 注意：该 RPC 可能需要 API 密钥或认证，开发者需参考 Ankr 文档（[Ankr 文档](https://www.ankr.com/docs/)）配置。
  * 区块浏览器：未提供具体链接，建议使用 BSC 官方测试网浏览器。
* **ETH 测试网（Holesky）**：
  * RPC 端点：`https://rpc.ankr.com/eth_holesky/b6aba93f145bbc37b4acc1cda2e7c0b38fff5a09fb21a55c1fb65883d35789bd`，同样由 Ankr 提供。
  * 区块浏览器：`https://holesky.etherscan.io`，可查询交易和合约详情。
  * 配置建议：确保钱包或工具支持 Holesky 网络，链 ID 为 17000（需确认）。

**合约地址与代币信息**

以下是跨链桥相关合约和代币的详细地址，开发者需确保在正确网络中使用：

| **网络**           | **合约/代币**                                                                                                | **地址**                                       | **链接**                                                                                        |
| ---------------- | -------------------------------------------------------------------------------------------------------- | -------------------------------------------- | --------------------------------------------------------------------------------------------- |
| JuChain 测试网      | BridgeBank [合约](https://explorer-testnet.juchain.org/address/0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE) | `0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE` | -                                                                                             |
| JuChain 测试网      | [USDT](https://explorer-testnet.juchain.org/token/0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D)            | `0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D` | [查看详情](https://explorer-testnet.juchain.org/token/0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D) |
| JuChain 测试网      | [tBNB](https://explorer-testnet.juchain.org/token/0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D)            | `0x2598d2e226Ce13288E314569569838bBc6Ff9402` | [查看详情](https://explorer-testnet.juchain.org/token/0x2598d2e226Ce13288E314569569838bBc6Ff9402) |
| JuChain 测试网      | [tETH](https://explorer-testnet.juchain.org/token/0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672)            | `0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672` | [查看详情](https://explorer-testnet.juchain.org/token/0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672) |
| BSC 测试网（Chapel）  | BridgeBank 合约                                                                                            | `0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6` | -                                                                                             |
| BSC 测试网（Chapel）  | USDT                                                                                                     | `0xcD1093897a5dB4a9aF153772B35AAA066ab969f3` | -                                                                                             |
| ETH 测试网（Holesky） | BridgeBank 合约                                                                                            | `0x264960f4bf655c14a74DE1A7fC5AA68E71f71924` | [查看详情](https://holesky.etherscan.io/address/0x264960f4bf655c14a74DE1A7fC5AA68E71f71924)       |
| ETH 测试网（Holesky） | USDT                                                                                                     | `0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add` | [查看详情](https://holesky.etherscan.io/address/0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add)       |

**BridgeBank 合约功能**

BridgeBank 合约是跨链桥的核心，提供了以下关键功能，开发者可通过 ABI 与合约交互：

* **lock**：在源链上锁定代币，准备跨链转移。
  * 参数：`_recipient`（目标链接收地址）、`_token`（代币地址）、`_amount`（转移金额）。
  * 示例：从 JuChain 测试网锁定 USDT 到 BSC。
* **unlock**：在目标链上解锁代币。
  * 参数：`_recipient`（接收地址）、`_token`（代币地址）、`_name`（代币名称）、`_amount`（金额）。
* **mintBridgeTokens**：在目标链上铸造桥接代币。
  * 参数：`_intendedRecipient`（接收地址）、`_bridgeTokenAddress`（桥接代币地址）、`_amount`（金额）。
* **burnBridgeTokens**：在源链上销毁桥接代币，完成跨链转移。
  * 参数：`_chainID`（目标链 ID，JuChain BridgeBank 专用）、`_receiver`（接收地址）、`_bridgeTokenAddress`（桥接代币地址）、`_amount`（金额）。

**ABI 数据**

以下是测试网使用的 BridgeBank 合约 ABI，开发者可用于与合约交互：

**JuChain BridgeBank ABI**

```json
[
  {"inputs":[{"internalType":"address","name":"_operatorAddress","type":"address"},{"internalType":"address","name":"_oracleAddress","type":"address"},{"internalType":"address","name":"_btcBridgeAddress","type":"address"},{"internalType":"address","name":"_tokenDeployer","type":"address"},{"internalType":"addresspayable","name":"_feeReceiver","type":"address"},{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"stateMutability":"nonpayable","type":"constructor"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"address","name":"_beneficiary","type":"address"}],"name":"LogBridgeTokenMint","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"uint256","name":"_chainID","type":"uint256"},{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"address","name":"_ownerFrom","type":"address"},{"indexed":false,"internalType":"address","name":"_receiver","type":"address"}],"name":"LogBtcTokenBurn","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_from","type":"address"},{"indexed":false,"internalType":"address","name":"_to","type":"address"},{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_value","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"_nonce","type":"uint256"}],"name":"LogLock","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"string","name":"_symbol","type":"string"},{"indexed":false,"internalType":"uint8","name":"decimals","type":"uint8"}],"name":"LogNewBridgeToken","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_to","type":"address"},{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_value","type":"uint256"}],"name":"LogUnlock","type":"event"},
  {"stateMutability":"payable","type":"fallback"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"}],"name":"addToken2LockList","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"bridgeServiceFee","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"bridgeTokenCount","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"bridgeTokenCreated","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"bridgeTokenWhitelist","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"btcBridge","outputs":[{"internalType":"contractBridge","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"uint256","name":"_chainID","type":"uint256"},{"internalType":"address","name":"_receiver","type":"address"},{"internalType":"address","name":"_bridgeTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"burnBridgeTokens","outputs":[],"stateMutability":"payable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"name":"configLockedTokenOfflineSave","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"addresspayable","name":"_offlineSave","type":"address"}],"name":"configOfflineSaveAccount","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"configplatformCoin","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"},{"internalType":"string","name":"_symbol","type":"string"},{"internalType":"uint8","name":"decimals","type":"uint8"}],"name":"createNewBridgeToken","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_bridgetoken","type":"address"},{"internalType":"uint256","name":"_toChainID","type":"uint256"}],"name":"enableBridgeToken2Withdraw","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"feeReceiver","outputs":[{"internalType":"addresspayable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"getLockedTokenAddress","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"getToken2address","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"getToken2addressV2","outputs":[{"internalType":"address","name":"","type":"address"},{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_tokenAddress","type":"address"}],"name":"getTokenName","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"}],"name":"getofflineSaveCfg","outputs":[{"internalType":"uint256","name":"","type":"uint256"},{"internalType":"uint8","name":"","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"hasBridgeTokenCreated","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"hasSetPlatformCoin","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"highThreshold","outputs":[{"internalType":"uint8","name":"","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_recipient","type":"address"},{"internalType":"address","name":"_token","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"lock","outputs":[],"stateMutability":"payable","type":"function"},
  {"inputs":[],"name":"lockNonce","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"lockedFunds","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"lowThreshold","outputs":[{"internalType":"uint8","name":"","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"addresspayable","name":"_intendedRecipient","type":"address"},{"internalType":"address","name":"_bridgeTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"mintBridgeTokens","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"offlineSave","outputs":[{"internalType":"addresspayable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"offlineSaveCfgs","outputs":[{"internalType":"address","name":"token","type":"address"},{"internalType":"string","name":"name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"operator","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"oracle","outputs":[{"internalType":"contractOracle","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"platformCoin","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"name":"setBridgeServiceFee","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"addresspayable","name":"_feeReceiver","type":"address"}],"name":"setFeeReceiver","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_tokenDeployer","type":"address"}],"name":"setTokenDeployer","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"token2WithdrawCfg","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"token2address","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"tokenAddrAllow2name","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"tokenAllow2Lock","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"tokenDeployer","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"addresspayable","name":"_recipient","type":"address"},{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"unlock","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"stateMutability":"payable","type":"receive"}
]
```

**ETH/BSC BridgeBank ABI**



```json
[
  {"inputs":[{"internalType":"address","name":"_operatorAddress","type":"address"},{"internalType":"address","name":"_oracleAddress","type":"address"},{"internalType":"address","name":"_bridgeAddress","type":"address"},{"internalType":"address","name":"_tokenDeployer","type":"address"},{"internalType":"addresspayable","name":"_feeReceiver","type":"address"},{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"stateMutability":"nonpayable","type":"constructor"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"address","name":"_beneficiary","type":"address"}],"name":"LogBridgeTokenMint","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"address","name":"_ownerFrom","type":"address"},{"indexed":false,"internalType":"address","name":"_receiver","type":"address"}],"name":"LogBtcTokenBurn","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_from","type":"address"},{"indexed":false,"internalType":"address","name":"_to","type":"address"},{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_value","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"_nonce","type":"uint256"}],"name":"LogLock","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"string","name":"_symbol","type":"string"},{"indexed":false,"internalType":"uint8","name":"decimals","type":"uint8"}],"name":"LogNewBridgeToken","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"bytes32","name":"_claimID","type":"bytes32"},{"indexed":false,"internalType":"address","name":"_tokenAddress","type":"address"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"index","type":"uint256"}],"name":"LogNotEnoughBalanceForBurn","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_to","type":"address"},{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_value","type":"uint256"}],"name":"LogUnlock","type":"event"},
  {"stateMutability":"payable","type":"fallback"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"}],"name":"addToken2LockList","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"bridge","outputs":[{"internalType":"contractBridge","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"bridgeServiceFee","outputs":[{"internalType":"uint256","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"bridgeTokenCount","outputs":[{"internalType":"uint256","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"bridgeTokenCreated","outputs":[{"internalType":"bool","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"bridgeTokenWhitelist","outputs":[{"internalType":"bool","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_Receiver","type":"address"},{"internalType":"address","name":"_btcTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"burnBridgeTokens","outputs":[],"stateMutability":"payable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"name":"configLockedTokenOfflineSave","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"addresspayable","name":"_offlineSave","type":"address"}],"name":"configOfflineSaveAccount","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"configplatformCoin","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"},{"internalType":"string","name":"_symbol","type":"string"},{"internalType":"uint8","name":"decimals","type":"uint8"}],"name":"createNewBridgeToken","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"feeReceiver","outputs":[{"internalType":"addresspayable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"getLockedTokenAddress","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"getProcClaimIndex","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"getSeqOnLackClaim","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"getStuckClaims","outputs":[{"components":[{"internalType":"bytes32","name":"_claimID","type":"bytes32"},{"internalType":"uint256","name":"_amount","type":"uint256"},{"internalType":"address","name":"_tokenAddress","type":"address"},{"internalType":"addresspayable","name":"_ethereumReceiver","type":"address"}],"internalType":"structBridgeBank.WaitFounds[]","name":"","type":"tuple[]"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"getToken2address","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"getToken2addressV2","outputs":[{"internalType":"address","name":"","type":"address"},{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_tokenAddress","type":"address"}],"name":"getTokenName","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"}],"name":"getofflineSaveCfg","outputs":[{"internalType":"uint256","name":"","type":"uint256"},{"internalType":"uint8","name":"","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"hasBridgeTokenCreated","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"hasSetPlatformCoin","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"highThreshold","outputs":[{"internalType":"uint8","name":"","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_recipient","type":"address"},{"internalType":"address","name":"_token","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"lock","outputs":[],"stateMutability":"payable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_recipient","type":"address"},{"internalType":"address","name":"_token","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"lockForWaitClaims","outputs":[],"stateMutability":"payable","type":"function"},
  {"inputs":[],"name":"lockNonce","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"lockedFunds","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"lowThreshold","outputs":[{"internalType":"uint8","name":"","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"addresspayable","name":"_intendedRecipient","type":"address"},{"internalType":"address","name":"_bridgeTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"mintBridgeTokens","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"offlineSave","outputs":[{"internalType":"addresspayable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"offlineSaveCfgs","outputs":[{"internalType":"address","name":"token","type":"address"},{"internalType":"string","name":"name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"operator","outputs":[{"internalType":"addresspayable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"oracle","outputs":[{"internalType":"contractOracle","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"platformCoin","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"preCheckLockedFounds","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"name":"setBridgeServiceFee","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"addresspayable","name":"_feeReceiver","type":"address"}],"name":"setFeeReceiver","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_tokenDeployer","type":"address"}],"name":"setTokenDeployer","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"token2address","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"tokenAddrAllow2name","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"tokenAllow2Lock","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"tokenDeployer","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"_claimID","type":"bytes32"},{"internalType":"addresspayable","name":"_recipient","type":"address"},{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"unlock","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"stateMutability":"payable","type":"receive"}
]
```

### 跨链流程

JuChain 跨链桥支持以下跨链流程：

#### 从 JuChain 到 BSC/ETH 的跨链流程

1. **锁定资产**：
   * 在 JuChain 测试网上，用户调用 BridgeBank 合约的 `burnBridgeTokens` 函数
   * 参数包括：目标链 ID（BSC 为 97，ETH Holesky 为 17000）、接收地址、代币地址和金额
   * 函数会销毁 JuChain 上的代币，并触发 `LogBtcTokenBurn` 事件
2. **跨链验证**：
   * 跨链桥的预言机监听 `LogBtcTokenBurn` 事件
   * 验证交易有效性后，在目标链上执行铸造操作
3. **目标链铸造**：
   * 在 BSC 或 ETH 测试网上，预言机调用 BridgeBank 合约的 `mintBridgeTokens` 函数
   * 在接收地址铸造等量的代币

#### 从 BSC/ETH 到 JuChain 的跨链流程

1. **锁定资产**：
   * 在 BSC 或 ETH 测试网上，用户调用 BridgeBank 合约的 `lock` 函数
   * 参数包括：接收地址、代币地址和金额
   * 函数会锁定代币，并触发 `LogLock` 事件
2. **跨链验证**：
   * 跨链桥的预言机监听 `LogLock` 事件
   * 验证交易有效性后，在 JuChain 上执行解锁操作
3. **JuChain 解锁**：
   * 在 JuChain 测试网上，预言机调用 BridgeBank 合约的 `unlock` 函数
   * 在接收地址解锁等量的代币

### 代码示例

以下是使用 Web3.js 与跨链桥交互的代码示例：

#### 从 JuChain 到 BSC 的跨链转移

```javascript
// 连接到 JuChain 测试网
const Web3 = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// BridgeBank ABI 和合约地址
const bridgeBankABI = [...]; // 使用上面提供的 JuChain BridgeBank ABI
const bridgeBankAddress = '0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE';

// 创建合约实例
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// 账户设置
const privateKey = '你的私钥';
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// 跨链参数
const bscChainID = 97; // BSC 测试网 Chain ID
const receiverAddress = '0x接收地址'; // BSC 上的接收地址
const tokenAddress = '0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D'; // JuChain 上的 USDT 地址
const amount = web3.utils.toWei('10', 'ether'); // 转移 10 个代币

// 执行跨链转移
async function crossChainTransfer() {
  try {
    // 获取服务费
    const fee = await bridgeBank.methods.bridgeServiceFee().call();
    
    // 调用 burnBridgeTokens 函数
    const tx = await bridgeBank.methods.burnBridgeTokens(
      bscChainID,
      receiverAddress,
      tokenAddress,
      amount
    ).send({
      from: account.address,
      value: fee,
      gas: 500000
    });
    
    console.log('跨链转移交易哈希:', tx.transactionHash);
  } catch (error) {
    console.error('跨链转移失败:', error);
  }
}

crossChainTransfer();
```

#### 从 BSC 到 JuChain 的跨链转移

```javascript
// 连接到 BSC 测试网
const Web3 = require('web3');
const web3 = new Web3('https://rpc.ankr.com/bsc_testnet_chapel/b6aba93f145bbc37b4acc1cda2e7c0b38fff5a09fb21a55c1fb65883d35789bd');

// BridgeBank ABI 和合约地址
const bridgeBankABI = [...]; // 使用上面提供的 ETH/BSC BridgeBank ABI
const bridgeBankAddress = '0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6';

// 创建合约实例
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// 账户设置
const privateKey = '你的私钥';
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// USDT 合约设置
const usdtABI = [
  // ERC20 标准接口
  {
    "constant": false,
    "inputs": [
      {"name": "_spender", "type": "address"},
      {"name": "_value", "type": "uint256"}
    ],
    "name": "approve",
    "outputs": [{"name": "", "type": "bool"}],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  }
];
const usdtAddress = '0xcD1093897a5dB4a9aF153772B35AAA066ab969f3'; // BSC 测试网上的 USDT 地址
const usdtContract = new web3.eth.Contract(usdtABI, usdtAddress);

// 跨链参数
const receiverAddress = '0x接收地址'; // JuChain 上的接收地址
const amount = web3.utils.toWei('10', 'ether'); // 转移 10 个代币

// 执行跨链转移
async function crossChainTransfer() {
  try {
    // 获取服务费
    const fee = await bridgeBank.methods.bridgeServiceFee().call();
    
    // 首先授权 BridgeBank 合约使用 USDT
    await usdtContract.methods.approve(bridgeBankAddress, amount).send({
      from: account.address,
      gas: 200000
    });
    
    console.log('授权成功');
    
    // 调用 lock 函数
    const tx = await bridgeBank.methods.lock(
      receiverAddress,
      usdtAddress,
      amount
    ).send({
      from: account.address,
      value: fee,
      gas: 500000
    });
    
    console.log('跨链转移交易哈希:', tx.transactionHash);
  } catch (error) {
    console.error('跨链转移失败:', error);
  }
}

crossChainTransfer();
```

### 注意事项

1. **测试网限制**：
   * 这些合约和代币仅在测试网上可用，不适用于主网环境
   * 测试网可能会定期重置，请不要在测试网上存储有价值的资产
2. **服务费**：
   * 跨链操作需要支付服务费，可通过 `bridgeServiceFee` 函数查询
   * 服务费以原生代币（JU、BNB 或 ETH）支付
3. **授权要求**：
   * 在锁定 ERC20 代币前，需要先授权 BridgeBank 合约使用代币
   * 使用代币的 `approve` 函数进行授权
4. **跨链延迟**：
   * 跨链操作通常需要一定时间完成，取决于预言机的处理速度
   * 请耐心等待，并通过区块浏览器查询交易状态
5. **代币兼容性**：
   * 只有列入白名单的代币才能进行跨链操作
   * 目前支持 USDT、tBNB 和 tETH
6. **错误处理**：
   * 如果跨链操作失败，请检查参数是否正确
   * 确保有足够的代币余额和服务费
