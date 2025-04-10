# 跨链桥

JuChain 的跨链桥是一个去中心化的服务，允许开发者在不同测试网之间转移资产，如 JuChain 测试网、BSC 测试网（Chapel）和 ETH 测试网（Holesky）。它通过链上智能合约（主要是 `BridgeBank` 合约）和链下组件（Relayers 和 Signers）协同工作，管理锁定、解锁、铸造和销毁操作，确保资产跨链转移的安全性和效率。

#### 架构概览

JuChain 跨链桥采用典型的"锁定/销毁 & 铸造/解锁"模型，并结合了链下验证机制。以下是核心组件及其交互：

* **BridgeBank 合约**:部署在各个支持的链（JuChain, BSC, ETH）上的核心智能合约。
  * 在**源链**: 负责接收和锁定（`lock`）用户的原始资产，或销毁（`burnBridgeTokens`）已桥接回来的资产，并触发相应的事件（`LogLock`, `LogBtcTokenBurn`）。
  * 在**目标链**: 负责根据验证后的信息铸造（`mintBridgeTokens`）桥接资产，或解锁（`unlock`）返回的原始资产。
* **Relayers (中继器)**: 链下服务，负责监控各个链上 `BridgeBank` 合约的事件。
  * **监听**: 持续监听源链上的 `LogLock` 事件（用于 ETH/BSC -> JuChain）或 JuChain 上的 `LogBtcTokenBurn` 事件（用于 JuChain -> ETH/BSC）。
  * **提交**: 检测到相关事件后，收集事件数据并将其提交给 Signers 进行验证。
  * **执行**: 收到 Signers 的有效签名/授权后，调用目标链 `BridgeBank` 合约的相应方法（`mintBridgeTokens` 或 `unlock`）来完成跨链操作。图示中有多个 Relayer (Relayer\_0, Relayer\_1, Relayer\_2)，可能意味着存在冗余或并行处理机制。
* **Signers (签名者/验证者)**: 链下服务，负责验证跨链事件的真实性和有效性。
  * **验证**: 接收来自 Relayer 的事件数据，并独立验证该事件在源链上是否真实发生且有效。
  * **授权**: 验证通过后，生成签名或其他形式的授权，允许 Relayer 在目标链上执行操作。图示中存在 `signer0` 和 `signer1`（甚至可能是多个 `signer1` 实例或代表一个多签群体），表明验证过程可能涉及多方或根据特定流程/链进行区分。`signer0` 处的索引信息（`Index: 0:for juchain, 1:eth, 2:bse`）表明验证节点内部需要区分不同的链。
* **用户 (用户)**: 发起跨链转移的最终使用者，与源链的 `BridgeBank` 合约交互以启动流程 。
* **管理员 (管理员)**: 可能代表进行初始流动性提供、合约维护或其他管理操作的角色（如图中向 BSC 存入 20）。

这个架构确保了跨链操作的安全性，因为目标链上的操作（铸造/解锁）必须经过链下验证者的确认。

***

**合约地址与代币信息**

以下是跨链桥相关合约和代币在不同网络上的详细地址，开发者需确保在正确网络中使用：

| 网络                      | 合约/代币                 | 地址                                                                                                                              | 说明/链接                        |
| ----------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| **--- 主网 Mainnet ---**  |                       |                                                                                                                                 |                              |
| ETH Mainnet             | USDT (桥接前)            | [`0xf127CcE9849BC3333B883F168efBf9773f49fD98`](https://etherscan.io/address/0xf127CcE9849BC3333B883F168efBf9773f49fD98)         | ETH 主网上的 USDT 合约             |
| BSC Mainnet             | USDT (桥接前)            | [`0x77051143118f4Dbe64cEE8ABEbd87A4B9b095402`](https://bscscan.com/address/0x77051143118f4Dbe64cEE8ABEbd87A4B9b095402)          | BSC 主网上的 USDT 合约             |
| JuChain Mainnet         | BridgeBank            | [`0x0B14AEc91b1020Bd03440A452C18B60C4D98fd0D`](https://juscan.io/address/0x0B14AEc91b1020Bd03440A452C18B60C4D98fd0D)            | JuChain 主网核心跨链合约             |
| JuChain Mainnet         | USDT (桥接后)            |                                                                                                                                 | JuChain 主网上代表 USDT 的桥接代币     |
| JuChain Mainnet         | BNB (桥接后)             | [`0x151b6F646Ac02Ed9877884ed9637A84f2FD8FaA6`](https://juscan.io/token/0x151b6F646Ac02Ed9877884ed9637A84f2FD8FaA6)              | JuChain 主网上代表 BNB 的桥接代币      |
| JuChain Mainnet         | ETH (桥接后)             | [`0x80077F108Dd73B709C43A1a13F0EEf25e48f7D0e`](https://juscan.io/token/0x80077F108Dd73B709C43A1a13F0EEf25e48f7D0e)              |                              |
| ETH Mainnet             | 签名地址 (Signer Address) | `0xc739962C7805a46BEd5bDADB4Df033e9B9aC1ff2`                                                                                    | 用于验证 ETH -> JuChain 跨链交易     |
| BSC Mainnet             | 签名地址 (Signer Address) | `0xc3F59038F2fceDec5f41f46aBb130ca4446556E1`                                                                                    | 用于验证 BSC -> JuChain 跨链交易     |
| JuChain Mainnet         | 签名地址 (Signer Address) | `0xA62b1782af4AfFd74CEcFC5E0BA96E1b31eb371C`                                                                                    | 用于验证 JuChain -> ETH/BSC 跨链交易 |
| **--- 测试网 Testnet ---** |                       |                                                                                                                                 |                              |
| JuChain Testnet         | BridgeBank            | [`0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE`](https://testnet.juscan.io/address/0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE)    | JuChain 测试网核心跨链合约            |
| JuChain Testnet         | USDT (桥接后)            | [`0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D`](https://testnet.juscan.io/token/0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D)      | JuChain 测试网上代表 USDT 的桥接代币    |
| JuChain Testnet         | tBNB (桥接后)            | [`0x2598d2e226Ce13288E314569569838bBc6Ff9402`](https://testnet.juscan.io/token/0x2598d2e226Ce13288E314569569838bBc6Ff9402)      | JuChain 测试网上代表 tBNB 的桥接代币    |
| JuChain Testnet         | tETH (桥接后)            | [`0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672`](https://testnet.juscan.io/token/0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672)      | JuChain 测试网上代表 tETH 的桥接代币    |
| BSC Testnet (Chapel)    | BridgeBank            | [`0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6`](https://testnet.bscscan.com/address/0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6)  | BSC 测试网核心跨链合约                |
| BSC Testnet (Chapel)    | USDT (桥接前)            | [`0xcD1093897a5dB4a9aF153772B35AAA066ab969f3`](https://testnet.bscscan.com/address/0xcD1093897a5dB4a9aF153772B35AAA066ab969f3)  | BSC 测试网上的 USDT 合约            |
| ETH Testnet (Holesky)   | BridgeBank            | [`0x264960f4bf655c14a74DE1A7fC5AA68E71f71924`](https://holesky.etherscan.io/address/0x264960f4bf655c14a74DE1A7fC5AA68E71f71924) | ETH Holesky 测试网核心跨链合约        |
| ETH Testnet (Holesky)   | USDT (桥接前)            | [`0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add`](https://holesky.etherscan.io/address/0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add) | ETH Holesky 测试网上的 USDT 合约    |

**BridgeBank 合约功能**

BridgeBank 合约是跨链桥的核心，提供了以下关键功能，开发者可通过 ABI 与合约交互：

* **`lock`** (ETH/BSC -> JuChain): 在源链（ETH/BSC）上锁定代币，并发起跨链转移。**此操作会触发 `LogLock` 事件，被 Relayers 监听。**
  * 参数：`_recipient`（JuChain 上的接收地址）、`_token`（源链代币地址）、`_amount`（转移金额）。
* **`burnBridgeTokens`** (JuChain -> ETH/BSC): 在 JuChain 上销毁桥接代币，并发起返回源链的跨链转移。**此操作会触发 `LogBtcTokenBurn` 事件，被 Relayers 监听。**
  * 参数：`_chainID`（目标链 ID）、`_receiver`（源链上的接收地址）、`_bridgeTokenAddress`（JuChain 上的桥接代币地址）、`_amount`（金额）。
* **`unlock`** (JuChain -> ETH/BSC completion): 在目标链（ETH/BSC）上解锁原始代币。**此操作由 Relayer 在收到 Signer 授权后调用。**
  * 参数：`_recipient`（接收地址）、`_token`（代币地址）、`_name`（代币名称）、`_amount`（金额）。(ETH/BSC ABI 中还包含 `_claimID`)
* **`mintBridgeTokens`** (ETH/BSC -> JuChain completion): 在目标链（JuChain）上铸造桥接代币。**此操作由 Relayer 在收到 Signer 授权后调用。**
  * 参数：`_intendedRecipient`（接收地址）、`_bridgeTokenAddress`（桥接代币地址）、`_amount`（金额）。

**ABI 数据**

以下是测试网使用的 BridgeBank 合约 ABI，开发者可用于与合约交互：

**JuChain BridgeBank ABI**

```json
[
  {"inputs":[{"internalType":"address","name":"_operatorAddress","type":"address"},{"internalType":"address","name":"_oracleAddress","type":"address"},{"internalType":"address","name":"_btcBridgeAddress","type":"address"},{"internalType":"address","name":"_tokenDeployer","type":"address"},{"internalType":"address payable","name":"_feeReceiver","type":"address"},{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"stateMutability":"nonpayable","type":"constructor"},
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
  {"inputs":[],"name":"btcBridge","outputs":[{"internalType":"contract Bridge","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"uint256","name":"_chainID","type":"uint256"},{"internalType":"address","name":"_receiver","type":"address"},{"internalType":"address","name":"_bridgeTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"burnBridgeTokens","outputs":[],"stateMutability":"payable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"name":"configLockedTokenOfflineSave","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address payable","name":"_offlineSave","type":"address"}],"name":"configOfflineSaveAccount","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"configplatformCoin","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"},{"internalType":"string","name":"_symbol","type":"string"},{"internalType":"uint8","name":"decimals","type":"uint8"}],"name":"createNewBridgeToken","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_bridgetoken","type":"address"},{"internalType":"uint256","name":"_toChainID","type":"uint256"}],"name":"enableBridgeToken2Withdraw","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"feeReceiver","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
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
  {"inputs":[{"internalType":"address payable","name":"_intendedRecipient","type":"address"},{"internalType":"address","name":"_bridgeTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"mintBridgeTokens","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"offlineSave","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"offlineSaveCfgs","outputs":[{"internalType":"address","name":"token","type":"address"},{"internalType":"string","name":"name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"operator","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"oracle","outputs":[{"internalType":"contract Oracle","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"platformCoin","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"name":"setBridgeServiceFee","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address payable","name":"_feeReceiver","type":"address"}],"name":"setFeeReceiver","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_tokenDeployer","type":"address"}],"name":"setTokenDeployer","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"token2WithdrawCfg","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"token2address","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"tokenAddrAllow2name","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"tokenAllow2Lock","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"tokenDeployer","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address payable","name":"_recipient","type":"address"},{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"unlock","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"stateMutability":"payable","type":"receive"}
]
```

**ETH/BSC BridgeBank ABI**

```json
[
  {"inputs":[{"internalType":"address","name":"_operatorAddress","type":"address"},{"internalType":"address","name":"_oracleAddress","type":"address"},{"internalType":"address","name":"_bridgeAddress","type":"address"},{"internalType":"address","name":"_tokenDeployer","type":"address"},{"internalType":"address payable","name":"_feeReceiver","type":"address"},{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"stateMutability":"nonpayable","type":"constructor"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"address","name":"_beneficiary","type":"address"}],"name":"LogBridgeTokenMint","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"address","name":"_ownerFrom","type":"address"},{"indexed":false,"internalType":"address","name":"_receiver","type":"address"}],"name":"LogBtcTokenBurn","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_from","type":"address"},{"indexed":false,"internalType":"address","name":"_to","type":"address"},{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_value","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"_nonce","type":"uint256"}],"name":"LogLock","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"string","name":"_symbol","type":"string"},{"indexed":false,"internalType":"uint8","name":"decimals","type":"uint8"}],"name":"LogNewBridgeToken","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"bytes32","name":"_claimID","type":"bytes32"},{"indexed":false,"internalType":"address","name":"_tokenAddress","type":"address"},{"indexed":false,"internalType":"uint256","name":"_amount","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"index","type":"uint256"}],"name":"LogNotEnoughBalanceForBurn","type":"event"},
  {"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_to","type":"address"},{"indexed":false,"internalType":"address","name":"_token","type":"address"},{"indexed":false,"internalType":"string","name":"_name","type":"string"},{"indexed":false,"internalType":"uint256","name":"_value","type":"uint256"}],"name":"LogUnlock","type":"event"},
  {"stateMutability":"payable","type":"fallback"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"}],"name":"addToken2LockList","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"bridge","outputs":[{"internalType":"contract Bridge","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"bridgeServiceFee","outputs":[{"internalType":"uint256","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"bridgeTokenCount","outputs":[{"internalType":"uint256","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"bridgeTokenCreated","outputs":[{"internalType":"bool","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"bridgeTokenWhitelist","outputs":[{"internalType":"bool","name":""}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_Receiver","type":"address"},{"internalType":"address","name":"_btcTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"burnBridgeTokens","outputs":[],"stateMutability":"payable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"name":"configLockedTokenOfflineSave","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address payable","name":"_offlineSave","type":"address"}],"name":"configOfflineSaveAccount","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"configplatformCoin","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"},{"internalType":"string","name":"_symbol","type":"string"},{"internalType":"uint8","name":"decimals","type":"uint8"}],"name":"createNewBridgeToken","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"feeReceiver","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"string","name":"_name","type":"string"}],"name":"getLockedTokenAddress","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"getProcClaimIndex","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"getSeqOnLackClaim","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"getStuckClaims","outputs":[{"components":[{"internalType":"bytes32","name":"_claimID","type":"bytes32"},{"internalType":"uint256","name":"_amount","type":"uint256"},{"internalType":"address","name":"_tokenAddress","type":"address"},{"internalType":"address payable","name":"_ethereumReceiver","type":"address"}],"internalType":"struct BridgeBank.WaitFounds[]","name":"","type":"tuple[]"}],"stateMutability":"view","type":"function"},
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
  {"inputs":[{"internalType":"address payable","name":"_intendedRecipient","type":"address"},{"internalType":"address","name":"_bridgeTokenAddress","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"mintBridgeTokens","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[],"name":"offlineSave","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"offlineSaveCfgs","outputs":[{"internalType":"address","name":"token","type":"address"},{"internalType":"string","name":"name","type":"string"},{"internalType":"uint256","name":"_threshold","type":"uint256"},{"internalType":"uint8","name":"_percents","type":"uint8"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"operator","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"oracle","outputs":[{"internalType":"contract Oracle","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"platformCoin","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"_token","type":"address"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"preCheckLockedFounds","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"uint256","name":"_bridgeServiceFee","type":"uint256"}],"name":"setBridgeServiceFee","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address payable","name":"_feeReceiver","type":"address"}],"name":"setFeeReceiver","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"address","name":"_tokenDeployer","type":"address"}],"name":"setTokenDeployer","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"token2address","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"tokenAddrAllow2name","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"name":"tokenAllow2Lock","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[],"name":"tokenDeployer","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},
  {"inputs":[{"internalType":"bytes32","name":"_claimID","type":"bytes32"},{"internalType":"address payable","name":"_recipient","type":"address"},{"internalType":"address","name":"_token","type":"address"},{"internalType":"string","name":"_name","type":"string"},{"internalType":"uint256","name":"_amount","type":"uint256"}],"name":"unlock","outputs":[],"stateMutability":"nonpayable","type":"function"},
  {"stateMutability":"payable","type":"receive"}
]
```

**跨链流程**

JuChain 跨链桥支持以下跨链流程：

**从 JuChain 到 BSC/ETH 的跨链流程**

1. **销毁资产 (用户操作 - JuChain)**:
   * 在 JuChain 测试网上，用户调用 `BridgeBank` 合约的 `burnBridgeTokens` 函数。
   * 参数包括：目标链 ID（BSC Chapel 为 97，ETH Holesky 为 17000）、接收地址（在目标链上）、代币地址（JuChain 上的桥接代币）和金额。
   * 函数会销毁 JuChain 上的桥接代币，并触发 `LogBtcTokenBurn` 事件。
2. **跨链验证 (链下操作 - Relayer & Signer)**:
   * **Relayers** 监听并检测到 JuChain 上的 `LogBtcTokenBurn` 事件。
   * Relayers 将事件信息提交给 **Signers**。
   * Signers 验证该销毁事件的有效性（确认交易已在 JuChain 上成功执行），并生成授权（例如，签名）。
3. **目标链解锁 (链下驱动，链上执行 - BSC/ETH)**:
   * 获得授权后，**Relayer** 调用目标链（BSC 或 ETH）`BridgeBank` 合约的 `unlock` 函数，附带授权信息。
   * 目标链 `BridgeBank` 合约验证授权的有效性，然后将等量的原始代币解锁/转移到用户指定的接收地址。

**从 BSC/ETH 到 JuChain 的跨链流程**

1. **锁定资产 (用户操作 - BSC/ETH)**:
   * 在 BSC 或 ETH 测试网上，用户调用 `BridgeBank` 合约的 `lock` 函数。
   * 参数包括：接收地址（在 JuChain 上）、代币地址（源链上的原始代币）和金额。
   * 函数会锁定用户的代币，并触发 `LogLock` 事件。
2. **跨链验证 (链下操作 - Relayer & Signer)**:
   * **Relayers** 监听并检测到源链上的 `LogLock` 事件。
   * Relayers 将事件信息提交给 **Signers**。
   * Signers 验证该锁定事件的有效性（确认交易已在源链上成功执行），并生成授权。
3. **JuChain 铸造 (链下驱动，链上执行 - JuChain)**:
   * 获得授权后，**Relayer** 调用 JuChain 测试网 `BridgeBank` 合约的 `mintBridgeTokens` 函数，附带授权信息。
   * JuChain `BridgeBank` 合约验证授权的有效性，然后铸造等量的桥接代币到用户指定的接收地址。

**代码示例**

以下是使用 Web3.js 与跨链桥交互的代码示例：

**从 JuChain 到 BSC 的跨链转移**

```javascript
const { Web3 } = require('web3');

// 连接到JuChain测试网
const web3 = new Web3('https://testnet-rpc.juchain.org');

// BridgeBank ABI和合约地址
const bridgeBankABI = [/* JuChain BridgeBank ABI from above */]; // 使用完整的JuChain BridgeBank ABI
const bridgeBankAddress = '0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE';

// 创建合约实例
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// 账户设置
const privateKey = '你的私钥'; // !!请勿在生产代码中硬编码私钥
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// ERC20代币ABI（用于检查余额和授权）
const erc20ABI = [
  {"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}
];

// 跨链参数
const bscChainID = 97; // BSC测试网Chapel Chain ID
const receiverAddress = '0x接收地址'; // BSC上的接收地址
const tokenAddress = '0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D'; // JuChain上的USDT地址
const amount = web3.utils.toWei('10', 'ether'); // 转移10个代币 (假设USDT是18位小数)

// 执行跨链转移
async function crossChainTransfer() {
  try {
    // 1. 检查账户余额 (用于支付Gas和可能的Service Fee)
    const balance = await web3.eth.getBalance(account.address);
    console.log('账户JU余额:', web3.utils.fromWei(balance, 'ether'), 'JU');

    // 2. 检查代币余额
    const tokenContract = new web3.eth.Contract(erc20ABI, tokenAddress);
    const tokenBalance = await tokenContract.methods.balanceOf(account.address).call();
    console.log('代币余额:', web3.utils.fromWei(tokenBalance, 'ether')); // 确保使用正确的小数位数
    if (BigInt(tokenBalance) < BigInt(amount)) {
      console.error('错误: 代币余额不足');
      return;
    }

    // 3. 授权代币使用权限 (仅在从 JuChain 转出桥接代币时需要)
    // 注意：burnBridgeTokens 通常不需要approve，因为它直接销毁用户拥有的代币。
    // approve 主要用于 lock 操作，将代币转移给合约。
    // 此处保留 Approve 逻辑以供 lock 参考，但在 burn 流程中可能不需要。
    // 如果你的 burnBridgeTokens 实现确实需要先转移代币，请取消注释此部分。
    /*
    const allowance = await tokenContract.methods.allowance(account.address, bridgeBankAddress).call();
    if (BigInt(allowance) < BigInt(amount)) {
      console.log('授权额度不足，正在授权...');
      // 授权一个足够大的数量
      const maxApproval = '115792089237316195423570985008687907853269984665640564039457584007913129639935'; // 2^256 - 1
      const approveTx = await tokenContract.methods.approve(bridgeBankAddress, maxApproval).send({
        from: account.address,
        gas: 200000 // 估算或设置合适的Gas
      });
      console.log('授权成功, Tx Hash:', approveTx.transactionHash);
      // 等待授权交易确认
      await web3.eth.getTransactionReceipt(approveTx.transactionHash);
    }
    */

    // 4. 获取跨链服务费
    let fee;
    try {
      fee = await bridgeBank.methods.bridgeServiceFee().call();
      console.log('服务费:', web3.utils.fromWei(fee.toString(), 'ether'), 'JU'); // 确保fee是字符串
    } catch (error) {
      console.error('获取服务费失败，可能合约不支持或RPC问题，使用默认值:', error.message);
      fee = web3.utils.toWei('0.01', 'ether'); // 默认服务费0.01 JU
    }

    // 将BigInt或数字转换为字符串，以避免类型混合问题
    const feeString = fee.toString();
    const amountString = amount.toString();

    // 5. 估算Gas
    let estimatedGas;
    try {
      estimatedGas = await bridgeBank.methods.burnBridgeTokens(
        bscChainID,
        receiverAddress,
        tokenAddress,
        amountString
      ).estimateGas({ from: account.address, value: feeString });
      console.log('预估Gas:', estimatedGas.toString());
    } catch (error) {
      console.error('Gas估算失败，请检查参数或网络状态，使用默认值:', error.message);
      estimatedGas = 500000n; // 使用 BigInt 作为默认 Gas 限制
    }

    // 将BigInt转换为数字或字符串，用于发送交易
    const gasLimit = BigInt(estimatedGas) + (BigInt(estimatedGas) / 2n); // 增加50%的Gas限制

    // 6. 调用burnBridgeTokens函数
    console.log(`准备发送交易: burnBridgeTokens(${bscChainID}, ${receiverAddress}, ${tokenAddress}, ${amountString}) with value: ${feeString}`);
    const tx = await bridgeBank.methods.burnBridgeTokens(
      bscChainID,
      receiverAddress,
      tokenAddress,
      amountString
    ).send({
      from: account.address,
      value: feeString,
      gas: gasLimit.toString(), // send 需要字符串或数字
      // EIP-1559 费用参数 (如果网络支持)
      // maxPriorityFeePerGas: web3.utils.toWei('2', 'gwei'), // 小费
      // maxFeePerGas: web3.utils.toWei('50', 'gwei') // 总费用上限
      // 或者 Legacy Gas Price:
      // gasPrice: web3.utils.toWei('10', 'gwei')
    });

    console.log('跨链转移交易哈希:', tx.transactionHash);
    console.log('交易已发送至 JuChain，请等待 Relayer 和 Signer 处理并在 BSC 上完成...');

  } catch (error) {
    console.error('跨链转移失败:', error);
    if (error.receipt) {
      console.error("交易失败回执:", error.receipt);
    }
  }
}

crossChainTransfer();
```

**从 BSC 到 JuChain 的跨链转移**

```javascript
const { Web3 } = require('web3');

// 连接到BSC测试网 (使用公共或你自己的节点)
const bscRpcUrl = 'https://data-seed-prebsc-1-s1.binance.org:8545/'; // 示例公共节点
// const bscRpcUrl = 'https://rpc.ankr.com/bsc_testnet_chapel/YOUR_ANKR_KEY'; // Ankr 示例
const web3 = new Web3(bscRpcUrl);

// BridgeBank ABI和合约地址 (BSC上的)
const bridgeBankABI = [/* ETH/BSC BridgeBank ABI from above */]; // 使用完整的BSC BridgeBank ABI
const bridgeBankAddress = '0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6';

// 创建合约实例
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// 账户设置
const privateKey = '你的私钥'; // !!请勿在生产代码中硬编码私钥
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// ERC20代币ABI (通用)
const erc20ABI = [
  {"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}
];

// 跨链参数
const receiverAddress = '0x接收地址'; // JuChain上的接收地址
const tokenAddress = '0xcD1093897a5dB4a9aF153772B35AAA066ab969f3'; // BSC测试网上的USDT地址
const amount = web3.utils.toWei('10', 'ether'); // 转移10个代币 (假设USDT是18位小数)

// 执行跨链转移
async function crossChainTransfer() {
  try {
    // 1. 检查BSC账户余额 (用于支付Gas和可能的Service Fee)
    const balance = await web3.eth.getBalance(account.address);
    console.log('BSC账户余额:', web3.utils.fromWei(balance, 'ether'), 'tBNB');

    // 2. 检查代币余额
    const tokenContract = new web3.eth.Contract(erc20ABI, tokenAddress);
    const tokenBalance = await tokenContract.methods.balanceOf(account.address).call();
    console.log('BSC代币余额:', web3.utils.fromWei(tokenBalance, 'ether')); // 确保使用正确的小数位数
    if (BigInt(tokenBalance) < BigInt(amount)) {
      console.error('错误: 代币余额不足');
      return;
    }

    // 3. 授权代币使用权限 (lock操作需要授权)
    const allowance = await tokenContract.methods.allowance(account.address, bridgeBankAddress).call();
    console.log('当前授权额度:', web3.utils.fromWei(allowance, 'ether'));
    if (BigInt(allowance) < BigInt(amount)) {
      console.log('授权额度不足，正在进行授权...');

      // 授权要转移的数量，或者一个更大的数量，例如 100 个
      // const approvalAmount = amount; // 仅授权需要的数量
      const approvalAmount = web3.utils.toWei('100', 'ether'); // 授权 100 个
      // const maxApproval = '115792089237316195423570985008687907853269984665640564039457584007913129639935'; // 最大授权

      const approveGas = await tokenContract.methods.approve(bridgeBankAddress, approvalAmount).estimateGas({ from: account.address });
      const approveTx = await tokenContract.methods.approve(bridgeBankAddress, approvalAmount).send({
        from: account.address,
        gas: (BigInt(approveGas) + BigInt(approveGas) / 2n).toString(), // 增加 50% Gas 缓冲
        // gasPrice: web3.utils.toWei('10', 'gwei') // BSC 测试网 Gas Price
      });
      console.log('授权成功, Tx Hash:', approveTx.transactionHash);
      // 等待授权交易确认可能更稳妥
      await web3.eth.getTransactionReceipt(approveTx.transactionHash);
      console.log('授权已确认');
    } else {
      console.log('授权额度充足');
    }


    // 4. 获取跨链服务费
    let fee;
    try {
      fee = await bridgeBank.methods.bridgeServiceFee().call();
      console.log('服务费:', web3.utils.fromWei(fee.toString(), 'ether'), 'tBNB'); // 确保 fee 是字符串
    } catch (error) {
      console.error('获取服务费失败，使用默认值:', error.message);
      fee = web3.utils.toWei('0.001', 'ether'); // 示例默认服务费0.001 tBNB
    }

    const feeString = fee.toString();
    const amountString = amount.toString();


    // 5. 估算Gas
    let estimatedGas;
    try {
       estimatedGas = await bridgeBank.methods.lock(
        receiverAddress,
        tokenAddress,
        amountString
      ).estimateGas({ from: account.address, value: feeString });
      console.log('预估Gas:', estimatedGas.toString());
    } catch (error) {
      console.error('Gas估算失败，请检查参数、授权或网络状态，使用默认值:', error.message);
      estimatedGas = 500000n; // 使用 BigInt 作为默认 Gas 限制
    }

    const gasLimit = BigInt(estimatedGas) + (BigInt(estimatedGas) / 2n); // 增加50%的Gas限制


    // 6. 调用lock函数进行跨链
    console.log(`准备发送交易: lock(${receiverAddress}, ${tokenAddress}, ${amountString}) with value: ${feeString}`);
    const tx = await bridgeBank.methods.lock(
      receiverAddress,
      tokenAddress,
      amountString
    ).send({
      from: account.address,
      value: feeString,
      gas: gasLimit.toString(),
      // gasPrice: web3.utils.toWei('10', 'gwei') // BSC 测试网 Gas Price
    });

    console.log('跨链转移交易哈希:', tx.transactionHash);
    console.log('交易已发送至 BSC，请等待 Relayer 和 Signer 处理并在 JuChain 上完成铸币...');

  } catch (error) {
    console.error('跨链转移失败:', error);
     if (error.receipt) {
      console.error("交易失败回执:", error.receipt);
    }
  }
}

crossChainTransfer();
```

**注意事项**

1. **测试网限制**：
   * 这些合约和代币仅在测试网上可用，不适用于主网环境。
   * 测试网可能会定期重置，请不要在测试网上存储有价值的资产。
2. **服务费**：
   * 跨链操作需要支付服务费，可通过 `bridgeServiceFee` 函数查询。
   * 服务费以源链的原生代币（JU、tBNB 或 Holesky ETH）支付，并在发起 `lock` 或 `burnBridgeTokens` 交易时作为 `value` 发送。
3. **授权要求**：
   * 在调用 `lock` 函数锁定 ERC20 代币前，用户需要先通过代币合约的 `approve` 函数授权 `BridgeBank` 合约使用相应数量的代币。
   * `burnBridgeTokens` 通常不需要 `approve`，因为它直接操作用户账户中的桥接代币。
4. **跨链延迟**：
   * 跨链操作并非即时完成，因为需要等待 Relayers 检测事件、Signers 验证以及 Relayers 在目标链执行操作。
   * 延迟时间取决于网络拥堵状况以及链下组件的处理速度。请耐心等待，并通过区块浏览器查询源链和目标链的交易状态。
5. **代币兼容性**：
   * 只有列入白名单的代币才能进行跨链操作。
   * 目前支持 USDT、tBNB 和 tETH (以其桥接形式存在于 JuChain，或原始形式存在于 ETH/BSC)。
6. **错误处理**：
   * 如果跨链操作失败，请检查：
     * 参数是否正确（接收地址、代币地址、金额、目标链 ID）。
     * 源链交易是否成功（Gas 不足、授权不足等）。
     * 是否有足够的代币余额和服务费（原生代币余额）。
   * 检查控制台输出的错误信息和交易回执。
7. **链下组件依赖**:
   * 桥的正常运行依赖于链下的 Relayers 和 Signers 组件。虽然这些组件通常被设计为去中心化和有弹性的（例如通过多个节点、多签或共识机制），但它们的可用性、正确性和处理速度对跨链转账的及时完成至关重要。

***

#### 跨链手续费

跨链操作需要支付服务费，该费用用于覆盖目标链的 Gas 成本以及平台运营成本。费用在发起 `lock` (从 ETH/BSC 到 JuChain) 或 `burnBridgeTokens` (从 JuChain 到 ETH/BSC) 交易时，以**源链的原生代币**形式支付。

**手续费构成:**

手续费通常包含两部分：

1. **网络 Gas 费**: 用于支付目标链上的交易执行成本（例如解锁或铸造操作）。这部分费用会根据目标链的实时 Gas 价格波动。
2. **平台服务费**: 由 JuChain 平台收取，用于维护跨链桥服务。

**主网手续费参考 (可能会根据网络状况和代币价格调整):**

| 跨链方向     | 币种   | 手续费详情 (示例)                                                                                                                            |
| -------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------- |
| BSC > JU | USDT | <p>收取 <strong>0.0002 BNB</strong> (网络费) + <strong>0.0002 JU</strong> (平台费)<br><em>(示例价值估算: 网络费 ≈ 0.1 USD, 平台费 ≈ 0.0004 USD)</em></p>  |
| BSC > JU | BNB  | <p>收取 <strong>0.0002 BNB</strong> (网络费) + <strong>0.0002 JU</strong> (平台费)<br><em>(示例价值估算: 网络费 ≈ 0.1 USD, 平台费 ≈ 0.0004 USD)</em></p>  |
| ETH > JU | USDT | <p>收取 <strong>0.00005 ETH</strong> (网络费) + <strong>0.0002 JU</strong> (平台费)<br><em>(示例价值估算: 网络费 ≈ 0.1 USD, 平台费 ≈ 0.0004 USD)</em></p> |
| ETH > JU | ETH  | <p>收取 <strong>0.00005 ETH</strong> (网络费) + <strong>0.0002 JU</strong> (平台费)<br><em>(示例价值估算: 网络费 ≈ 0.1 USD, 平台费 ≈ 0.0004 USD)</em></p> |
| JU > BSC | USDT | <p>收取 <strong>1 JU</strong> (平台费) + <strong>0.002 BNB</strong> (预估网络费)<br><em>(示例价值估算: 总计 ≈ 1 USD)</em></p>                           |
| JU > BSC | BNB  | <p>收取 <strong>1 JU</strong> (平台费) + <strong>0.002 BNB</strong> (预估网络费)<br><em>(示例价值估算: 总计 ≈ 1 USD)</em></p>                           |
| JU > ETH | USDT | <p>收取 <strong>1 JU</strong> (平台费) + <strong>0.00002 ETH</strong> (预估网络费)<br><em>(示例价值估算: 总计 ≈ 0.036 USD)</em></p>                     |
| JU > ETH | ETH  | <p>收取 <strong>1 JU</strong> (平台费) + <strong>0.00002 ETH</strong> (预估网络费)<br><em>(示例价值估算: 总计 ≈ 0.036 USD)</em></p>                     |

_注意: 上述价值估算是基于特定时间点的价格 (例如 BNB=590 USD, ETH=1800 USD, JU=2 USD)，实际费用和价值会波动。平台消耗部分指的是平台为处理交易所支付的成本。_

**手续费接收地址 (Fee Receiver Address):**

`0x8C0641240B418e0349dC52abd3F5cEcc4D4C748A`

{% hint style="info" %}
_建议：_ 在执行跨链操作前，可以通过调用源链 `BridgeBank` 合约的 `bridgeServiceFee()` 方法查询当前的预估服务费（通常仅包含平台服务费部分，不含目标链 Gas 费）。实际支付的总费用需用户根据当前网络状况和目标链 Gas 预估。
{% endhint %}

