# Cross-Chain Bridge

JuChain’s cross-chain bridge is a decentralized service that allows developers to transfer assets between different testnets, such as the JuChain Testnet, BSC Testnet (Chapel), and ETH Testnet (Holesky). It operates through on-chain smart contracts (primarily the `BridgeBank` contract) and off-chain components (Relayers and Signers) that work together to manage locking, unlocking, minting, and burning operations, ensuring the security and efficiency of cross-chain asset transfers.

#### Architecture Overview

JuChain’s cross-chain bridge adopts a typical "lock/burn & mint/unlock" model, combined with an off-chain validation mechanism. Below are the core components and their interactions:

* **BridgeBank Contract**: The core smart contract deployed on each supported chain (JuChain, BSC, ETH).
  * **On the Source Chain**: Responsible for receiving and locking (`lock`) users’ original assets or burning (`burnBridgeTokens`) assets bridged back, and triggering corresponding events (`LogLock`, `LogBtcTokenBurn`).
  * **On the Target Chain**: Responsible for minting (`mintBridgeTokens`) bridged assets or unlocking (`unlock`) returned original assets based on validated information.
* **Relayers**: Off-chain services that monitor events from the `BridgeBank` contracts on each chain.
  * **Monitoring**: Continuously listen for `LogLock` events on the source chain (for ETH/BSC -> JuChain) or `LogBtcTokenBurn` events on JuChain (for JuChain -> ETH/BSC).
  * **Submission**: Upon detecting relevant events, collect event data and submit it to Signers for validation.
  * **Execution**: After receiving valid signatures/authorizations from Signers, call the appropriate methods (`mintBridgeTokens` or `unlock`) on the target chain’s `BridgeBank` contract to complete the cross-chain operation. The diagram shows multiple Relayers (Relayer\_0, Relayer\_1, Relayer\_2), suggesting redundancy or parallel processing mechanisms.
* **Signers (Validators)**: Off-chain services responsible for verifying the authenticity and validity of cross-chain events.
  * **Validation**: Receive event data from Relayers and independently verify whether the event genuinely occurred and is valid on the source chain.
  * **Authorization**: Upon successful validation, generate signatures or other forms of authorization, allowing Relayers to execute operations on the target chain. The diagram includes `signer0` and `signer1` (possibly multiple `signer1` instances or representing a multi-signature group), indicating that the validation process may involve multiple parties or vary by chain/process. The index information at `signer0` (`Index: 0:for juchain, 1:eth, 2:bsc`) suggests that validation nodes internally distinguish between different chains.
* **Users**: The end-users initiating cross-chain transfers, interacting with the source chain’s `BridgeBank` contract to start the process.
* **Admin**: A role potentially responsible for providing initial liquidity, maintaining contracts, or performing other administrative tasks (e.g., depositing 20 into BSC as shown in the diagram).

This architecture ensures the security of cross-chain operations, as actions on the target chain (minting/unlocking) require confirmation from off-chain validators.

***

**Contract Addresses and Token Information**

Below are the detailed addresses of the cross-chain bridge-related contracts and tokens across different networks. Developers must ensure they use the correct network:

| Network               | Contract/Token     | Address                                                                                                                         | Description/Link                                 |
| --------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **--- Mainnet ---**   |                    |                                                                                                                                 |                                                  |
| ETH Mainnet           | USDT (pre-bridge)  | [`0xf127CcE9849BC3333B883F168efBf9773f49fD98`](https://etherscan.io/address/0xf127CcE9849BC3333B883F168efBf9773f49fD98)         | USDT contract on ETH Mainnet                     |
| BSC Mainnet           | USDT (pre-bridge)  | [`0x77051143118f4Dbe64cEE8ABEbd87A4B9b095402`](https://bscscan.com/address/0x77051143118f4Dbe64cEE8ABEbd87A4B9b095402)          | USDT contract on BSC Mainnet                     |
| JuChain Mainnet       | BridgeBank         | [`0x0B14AEc91b1020Bd03440A452C18B60C4D98fd0D`](https://juscan.io/address/0x0B14AEc91b1020Bd03440A452C18B60C4D98fd0D)            | Core cross-chain contract on JuChain Mainnet     |
| JuChain Mainnet       | USDT (post-bridge) | [`0x80077F108Dd73B709C43A1a13F0EEf25e48f7D0e`](https://juscan.io/token/0x80077F108Dd73B709C43A1a13F0EEf25e48f7D0e)              | Bridged USDT token on JuChain Mainnet            |
| JuChain Mainnet       | BNB (post-bridge)  | [`0x151b6F646Ac02Ed9877884ed9637A84f2FD8FaA6`](https://juscan.io/token/0x151b6F646Ac02Ed9877884ed9637A84f2FD8FaA6)              | Bridged BNB token on JuChain Mainnet             |
| JuChain Mainnet       | ETH (post-bridge)  | `---`                                                                                                                           | (Not available)                                  |
| ETH Mainnet           | Signer Address     | `0xc739962C7805a46BEd5bDADB4Df033e9B9aC1ff2`                                                                                    | Used to validate ETH -> JuChain transactions     |
| BSC Mainnet           | Signer Address     | `0xc3F59038F2fceDec5f41f46aBb130ca4446556E1`                                                                                    | Used to validate BSC -> JuChain transactions     |
| JuChain Mainnet       | Signer Address     | `0xA62b1782af4AfFd74CEcFC5E0BA96E1b31eb371C`                                                                                    | Used to validate JuChain -> ETH/BSC transactions |
| **--- Testnet ---**   |                    |                                                                                                                                 |                                                  |
| JuChain Testnet       | BridgeBank         | [`0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE`](https://testnet.juscan.io/address/0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE)    | Core cross-chain contract on JuChain Testnet     |
| JuChain Testnet       | USDT (post-bridge) | [`0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D`](https://testnet.juscan.io/token/0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D)      | Bridged USDT token on JuChain Testnet            |
| JuChain Testnet       | tBNB (post-bridge) | [`0x2598d2e226Ce13288E314569569838bBc6Ff9402`](https://testnet.juscan.io/token/0x2598d2e226Ce13288E314569569838bBc6Ff9402)      | Bridged tBNB token on JuChain Testnet            |
| JuChain Testnet       | tETH (post-bridge) | [`0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672`](https://testnet.juscan.io/token/0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672)      | Bridged tETH token on JuChain Testnet            |
| BSC Testnet (Chapel)  | BridgeBank         | [`0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6`](https://testnet.bscscan.com/address/0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6)  | Core cross-chain contract on BSC Testnet         |
| BSC Testnet (Chapel)  | USDT (pre-bridge)  | [`0xcD1093897a5dB4a9aF153772B35AAA066ab969f3`](https://testnet.bscscan.com/address/0xcD1093897a5dB4a9aF153772B35AAA066ab969f3)  | USDT contract on BSC Testnet                     |
| ETH Testnet (Holesky) | BridgeBank         | [`0x264960f4bf655c14a74DE1A7fC5AA68E71f71924`](https://holesky.etherscan.io/address/0x264960f4bf655c14a74DE1A7fC5AA68E71f71924) | Core cross-chain contract on ETH Holesky Testnet |
| ETH Testnet (Holesky) | USDT (pre-bridge)  | [`0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add`](https://holesky.etherscan.io/address/0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add) | USDT contract on ETH Holesky Testnet             |

**BridgeBank Contract Functions**

The `BridgeBank` contract is the core of the cross-chain bridge, providing the following key functions that developers can interact with via ABI:

* **`lock`** (ETH/BSC -> JuChain): Locks tokens on the source chain (ETH/BSC) and initiates a cross-chain transfer. **This triggers the `LogLock` event, monitored by Relayers.**
  * Parameters: `_recipient` (recipient address on JuChain), `_token` (source chain token address), `_amount` (transfer amount).
* **`burnBridgeTokens`** (JuChain -> ETH/BSC): Burns bridged tokens on JuChain and initiates a return transfer to the source chain. **This triggers the `LogBtcTokenBurn` event, monitored by Relayers.**
  * Parameters: `_chainID` (target chain ID), `_receiver` (recipient address on the source chain), `_bridgeTokenAddress` (bridged token address on JuChain), `_amount` (amount).
* **`unlock`** (JuChain -> ETH/BSC completion): Unlocks original tokens on the target chain (ETH/BSC). **Called by a Relayer after receiving Signer authorization.**
  * Parameters: `_recipient` (recipient address), `_token` (token address), `_name` (token name), `_amount` (amount). (ETH/BSC ABI also includes `_claimID`).
* **`mintBridgeTokens`** (ETH/BSC -> JuChain completion): Mints bridged tokens on the target chain (JuChain). **Called by a Relayer after receiving Signer authorization.**
  * Parameters: `_intendedRecipient` (recipient address), `_bridgeTokenAddress` (bridged token address), `_amount` (amount).

**ABI Data**

Below is the ABI for the `BridgeBank` contracts used on the testnets, which developers can use to interact with the contracts:

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

**Cross-Chain Process**

JuChain’s cross-chain bridge supports the following workflows:

**From JuChain to BSC/ETH**

1. **Burn Assets (User Action - JuChain)**:
   * On the JuChain Testnet, the user calls the `burnBridgeTokens` function of the `BridgeBank` contract.
   * Parameters include: target chain ID (97 for BSC Chapel, 17000 for ETH Holesky), recipient address (on the target chain), token address (bridged token on JuChain), and amount.
   * The function burns the bridged tokens on JuChain and triggers the `LogBtcTokenBurn` event.
2. **Cross-Chain Validation (Off-Chain - Relayer & Signer)**:
   * **Relayers** monitor and detect the `LogBtcTokenBurn` event on JuChain.
   * Relayers submit the event data to **Signers**.
   * Signers verify the validity of the burn event (confirming the transaction was successfully executed on JuChain) and generate authorization (e.g., a signature).
3. **Target Chain Unlock (Off-Chain Driven, On-Chain Execution - BSC/ETH)**:
   * After receiving authorization, the **Relayer** calls the `unlock` function on the target chain’s (BSC or ETH) `BridgeBank` contract, attaching the authorization data.
   * The target chain’s `BridgeBank` contract verifies the authorization and unlocks/transfers the equivalent amount of original tokens to the user-specified recipient address.

**From BSC/ETH to JuChain**

1. **Lock Assets (User Action - BSC/ETH)**:
   * On the BSC or ETH Testnet, the user calls the `lock` function of the `BridgeBank` contract.
   * Parameters include: recipient address (on JuChain), token address (original token on the source chain), and amount.
   * The function locks the user’s tokens and triggers the `LogLock` event.
2. **Cross-Chain Validation (Off-Chain - Relayer & Signer)**:
   * **Relayers** monitor and detect the `LogLock` event on the source chain.
   * Relayers submit the event data to **Signers**.
   * Signers verify the validity of the lock event (confirming the transaction was successfully executed on the source chain) and generate authorization.
3. **JuChain Minting (Off-Chain Driven, On-Chain Execution - JuChain)**:
   * After receiving authorization, the **Relayer** calls the `mintBridgeTokens` function on the JuChain Testnet’s `BridgeBank` contract, attaching the authorization data.
   * The JuChain `BridgeBank` contract verifies the authorization and mints the equivalent amount of bridged tokens to the user-specified recipient address.

**Code Examples**

Below are code examples using Web3.js to interact with the cross-chain bridge:

**Cross-Chain Transfer from JuChain to BSC**

```javascript
const { Web3 } = require('web3');

// Connect to JuChain Testnet
const web3 = new Web3('https://testnet-rpc.juchain.org');

// BridgeBank ABI and contract address
const bridgeBankABI = [/* JuChain BridgeBank ABI from above */]; // Use the full JuChain BridgeBank ABI
const bridgeBankAddress = '0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE';

// Create contract instance
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// Account setup
const privateKey = 'your_private_key'; // !!Do not hardcode private keys in production code
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// ERC20 token ABI (for balance checks and approvals)
const erc20ABI = [
  {"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}
];

// Cross-chain parameters
const bscChainID = 97; // BSC Testnet Chapel Chain ID
const receiverAddress = '0xrecipient_address'; // Recipient address on BSC
const tokenAddress = '0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D'; // USDT address on JuChain
const amount = web3.utils.toWei('10', 'ether'); // Transfer 10 tokens (assuming 18 decimals)

// Perform cross-chain transfer
async function crossChainTransfer() {
  try {
    // 1. Check account balance (for Gas and possible Service Fee)
    const balance = await web3.eth.getBalance(account.address);
    console.log('Account JU Balance:', web3.utils.fromWei(balance, 'ether'), 'JU');

    // 2. Check token balance
    const tokenContract = new web3.eth.Contract(erc20ABI, tokenAddress);
    const tokenBalance = await tokenContract.methods.balanceOf(account.address).call();
    console.log('Token Balance:', web3.utils.fromWei(tokenBalance, 'ether')); // Ensure correct decimals
    if (BigInt(tokenBalance) < BigInt(amount)) {
      console.error('Error: Insufficient token balance');
      return;
    }

    // 3. Approve token usage (only needed for lock operations, not burn in most cases)
    // Note: burnBridgeTokens typically doesn’t require approve since it burns user-owned tokens directly.
    // Uncomment this section if your burnBridgeTokens requires token transfer first.
    /*
    const allowance = await tokenContract.methods.allowance(account.address, bridgeBankAddress).call();
    if (BigInt(allowance) < BigInt(amount)) {
      console.log('Insufficient allowance, approving...');
      // Approve a sufficiently large amount
      const maxApproval = '115792089237316195423570985008687907853269984665640564039457584007913129639935'; // 2^256 - 1
      const approveTx = await tokenContract.methods.approve(bridgeBankAddress, maxApproval).send({
        from: account.address,
        gas: 200000 // Estimate or set appropriate Gas
      });
      console.log('Approval successful, Tx Hash:', approveTx.transactionHash);
      // Wait for approval transaction confirmation
      await web3.eth.getTransactionReceipt(approveTx.transactionHash);
    }
    */

    // 4. Get cross-chain service fee
    let fee;
    try {
      fee = await bridgeBank.methods.bridgeServiceFee().call();
      console.log('Service Fee:', web3.utils.fromWei(fee.toString(), 'ether'), 'JU'); // Ensure fee is a string
    } catch (error) {
      console.error('Failed to fetch service fee, possibly due to contract or RPC issue, using default:', error.message);
      fee = web3.utils.toWei('0.01', 'ether'); // Default fee 0.01 JU
    }

    // Convert BigInt or number to string to avoid type mismatch
    const feeString = fee.toString();
    const amountString = amount.toString();

    // 5. Estimate Gas
    let estimatedGas;
    try {
      estimatedGas = await bridgeBank.methods.burnBridgeTokens(
        bscChainID,
        receiverAddress,
        tokenAddress,
        amountString
      ).estimateGas({ from: account.address, value: feeString });
      console.log('Estimated Gas:', estimatedGas.toString());
    } catch (error) {
      console.error('Gas estimation failed, check parameters or network status, using default:', error.message);
      estimatedGas = 500000n; // Use BigInt as default Gas limit
    }

    // Convert BigInt to number or string for transaction
    const gasLimit = BigInt(estimatedGas) + (BigInt(estimatedGas) / 2n); // Add 50% Gas buffer

    // 6. Call burnBridgeTokens function
    console.log(`Preparing transaction: burnBridgeTokens(${bscChainID}, ${receiverAddress}, ${tokenAddress}, ${amountString}) with value: ${feeString}`);
    const tx = await bridgeBank.methods.burnBridgeTokens(
      bscChainID,
      receiverAddress,
      tokenAddress,
      amountString
    ).send({
      from: account.address,
      value: feeString,
      gas: gasLimit.toString(), // Send requires string or number
      // EIP-1559 fee parameters (if supported by network)
      // maxPriorityFeePerGas: web3.utils.toWei('2', 'gwei'), // Tip
      // maxFeePerGas: web3.utils.toWei('50', 'gwei') // Max fee cap
      // Or Legacy Gas Price:
      // gasPrice: web3.utils.toWei('10', 'gwei')
    });

    console.log('Cross-chain transfer transaction hash:', tx.transactionHash);
    console.log('Transaction sent to JuChain, please wait for Relayer and Signer to process and complete on BSC...');

  } catch (error) {
    console.error('Cross-chain transfer failed:', error);
    if (error.receipt) {
      console.error("Transaction failure receipt:", error.receipt);
    }
  }
}

crossChainTransfer();
```

**Cross-Chain Transfer from BSC to JuChain**

```javascript
const { Web3 } = require('web3');

// Connect to BSC Testnet (use a public or your own node)
const bscRpcUrl = 'https://data-seed-prebsc-1-s1.binance.org:8545/'; // Example public node
// const bscRpcUrl = 'https://rpc.ankr.com/bsc_testnet_chapel/YOUR_ANKR_KEY'; // Ankr example
const web3 = new Web3(bscRpcUrl);

// BridgeBank ABI and contract address (on BSC)
const bridgeBankABI = [/* ETH/BSC BridgeBank ABI from above */]; // Use the full BSC BridgeBank ABI
const bridgeBankAddress = '0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6';

// Create contract instance
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// Account setup
const privateKey = 'your_private_key'; // !!Do not hardcode private keys in production code
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// ERC20 token ABI (generic)
const erc20ABI = [
  {"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}
];

// Cross-chain parameters
const receiverAddress = '0xrecipient_address'; // Recipient address on JuChain
const tokenAddress = '0xcD1093897a5dB4a9aF153772B35AAA066ab969f3'; // USDT address on BSC Testnet
const amount = web3.utils.toWei('10', 'ether'); // Transfer 10 tokens (assuming 18 decimals)

// Perform cross-chain transfer
async function crossChainTransfer() {
  try {
    // 1. Check BSC account balance (for Gas and possible Service Fee)
    const balance = await web3.eth.getBalance(account.address);
    console.log('BSC Account Balance:', web3.utils.fromWei(balance, 'ether'), 'tBNB');

    // 2. Check token balance
    const tokenContract = new web3.eth.Contract(erc20ABI, tokenAddress);
    const tokenBalance = await tokenContract.methods.balanceOf(account.address).call();
    console.log('BSC Token Balance:', web3.utils.fromWei(tokenBalance, 'ether')); // Ensure correct decimals
    if (BigInt(tokenBalance) < BigInt(amount)) {
      console.error('Error: Insufficient token balance');
      return;
    }

    // 3. Approve token usage (required for lock operation)
    const allowance = await tokenContract.methods.allowance(account.address, bridgeBankAddress).call();
    console.log('Current Allowance:', web3.utils.fromWei(allowance, 'ether'));
    if (BigInt(allowance) < BigInt(amount)) {
      console.log('Insufficient allowance, approving...');

      // Approve the amount to transfer, or a larger amount, e.g., 100 tokens
      // const approvalAmount = amount; // Approve only the required amount
      const approvalAmount = web3.utils.toWei('100', 'ether'); // Approve 100 tokens
      // const maxApproval = '115792089237316195423570985008687907853269984665640564039457584007913129639935'; // Max approval

      const approveGas = await tokenContract.methods.approve(bridgeBankAddress, approvalAmount).estimateGas({ from: account.address });
      const approveTx = await tokenContract.methods.approve(bridgeBankAddress, approvalAmount).send({
        from: account.address,
        gas: (BigInt(approveGas) + BigInt(approveGas) / 2n).toString(), // Add 50% Gas buffer
        // gasPrice: web3.utils.toWei('10', 'gwei') // BSC Testnet Gas Price
      });
      console.log('Approval successful, Tx Hash:', approveTx.transactionHash);
      // Waiting for approval confirmation might be safer
      await web3.eth.getTransactionReceipt(approveTx.transactionHash);
      console.log('Approval confirmed');
    } else {
      console.log('Sufficient allowance');
    }

    // 4. Get cross-chain service fee
    let fee;
    try {
      fee = await bridgeBank.methods.bridgeServiceFee().call();
      console.log('Service Fee:', web3.utils.fromWei(fee.toString(), 'ether'), 'tBNB'); // Ensure fee is a string
    } catch (error) {
      console.error('Failed to fetch service fee, using default:', error.message);
      fee = web3.utils.toWei('0.001', 'ether'); // Example default fee 0.001 tBNB
    }

    const feeString = fee.toString();
    const amountString = amount.toString();

    // 5. Estimate Gas
    let estimatedGas;
    try {
      estimatedGas = await bridgeBank.methods.lock(
        receiverAddress,
        tokenAddress,
        amountString
      ).estimateGas({ from: account.address, value: feeString });
      console.log('Estimated Gas:', estimatedGas.toString());
    } catch (error) {
      console.error('Gas estimation failed, check parameters, approval, or network status, using default:', error.message);
      estimatedGas = 500000n; // Use BigInt as default Gas limit
    }

    const gasLimit = BigInt(estimatedGas) + (BigInt(estimatedGas) / 2n); // Add 50% Gas buffer

    // 6. Call lock function for cross-chain transfer
    console.log(`Preparing transaction: lock(${receiverAddress}, ${tokenAddress}, ${amountString}) with value: ${feeString}`);
    const tx = await bridgeBank.methods.lock(
      receiverAddress,
      tokenAddress,
      amountString
    ).send({
      from: account.address,
      value: feeString,
      gas: gasLimit.toString(),
      // gasPrice: web3.utils.toWei('10', 'gwei') // BSC Testnet Gas Price
    });

    console.log('Cross-chain transfer transaction hash:', tx.transactionHash);
    console.log('Transaction sent to BSC, please wait for Relayer and Signer to process and complete minting on JuChain...');

  } catch (error) {
    console.error('Cross-chain transfer failed:', error);
    if (error.receipt) {
      console.error("Transaction failure receipt:", error.receipt);
    }
  }
}

crossChainTransfer();
```

**Notes**

1. **Testnet Limitations**:
   * These contracts and tokens are only available on testnets and are not suitable for mainnet environments.
   * Testnets may be reset periodically; do not store valuable assets on them.
2. **Service Fees**:
   * Cross-chain operations require a service fee, which can be queried via the `bridgeServiceFee` function.
   * Fees are paid in the source chain’s native token (JU, tBNB, or Holesky ETH) and sent as the `value` when initiating `lock` or `burnBridgeTokens` transactions.
3. **Approval Requirements**:
   * Before calling the `lock` function to lock ERC20 tokens, users must first call the token contract’s `approve` function to authorize the `BridgeBank` contract to use the required amount.
   * `burnBridgeTokens` typically does not require `approve`, as it directly operates on the bridged tokens in the user’s account.
4. **Cross-Chain Delay**:
   * Cross-chain operations are not instantaneous, as they depend on Relayers detecting events, Signers validating them, and Relayers executing actions on the target chain.
   * Delays depend on network congestion and the processing speed of off-chain components. Be patient and check transaction statuses on block explorers for both source and target chains.
5. **Token Compatibility**:
   * Only whitelisted tokens can be bridged.
   * Currently supported tokens include USDT, tBNB, and tETH (in bridged form on JuChain or original form on ETH/BSC).
6. **Error Handling**:
   * If a cross-chain operation fails, check:
     * Whether parameters are correct (recipient address, token address, amount, target chain ID).
     * Whether the source chain transaction succeeded (insufficient Gas, insufficient approval, etc.).
     * Whether there’s enough token balance and service fee (native token balance).
   * Review console output for error messages and transaction receipts.
7. **Off-Chain Component Dependency**:
   * The bridge’s proper functioning relies on off-chain Relayers and Signers. While these components are typically designed to be decentralized and resilient (e.g., through multiple nodes, multi-signature, or consensus mechanisms), their availability, correctness, and processing speed are critical to the timely completion of cross-chain transfers.

***

#### Cross-Chain Fees

Cross-chain operations require a service fee to cover the Gas costs on the target chain and platform operational costs. The fee is paid in the **source chain’s native token** when initiating a `lock` (ETH/BSC to JuChain) or `burnBridgeTokens` (JuChain to ETH/BSC) transaction.

**Fee Structure:**

Fees typically consist of two parts:

1. **Network Gas Fee**: Covers the transaction execution cost on the target chain (e.g., unlocking or minting). This varies based on the target chain’s real-time Gas price.
2. **Platform Service Fee**: Charged by the JuChain platform to maintain the bridge service.

**Mainnet Fee Reference (subject to change based on network conditions and token prices):**

| Direction | Token | Fee Details (Example)                                                                                                                                                         |
| --------- | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| BSC > JU  | USDT  | <p>Charges <strong>0.0002 BNB</strong> (network fee) + <strong>0.0002 JU</strong> (platform fee)<br><em>(Estimated value: network ≈ 0.1 USD, platform ≈ 0.0004 USD)</em></p>  |
| BSC > JU  | BNB   | <p>Charges <strong>0.0002 BNB</strong> (network fee) + <strong>0.0002 JU</strong> (platform fee)<br><em>(Estimated value: network ≈ 0.1 USD, platform ≈ 0.0004 USD)</em></p>  |
| ETH > JU  | USDT  | <p>Charges <strong>0.00005 ETH</strong> (network fee) + <strong>0.0002 JU</strong> (platform fee)<br><em>(Estimated value: network ≈ 0.1 USD, platform ≈ 0.0004 USD)</em></p> |
| ETH > JU  | ETH   | <p>Charges <strong>0.00005 ETH</strong> (network fee) + <strong>0.0002 JU</strong> (platform fee)<br><em>(Estimated value: network ≈ 0.1 USD, platform ≈ 0.0004 USD)</em></p> |
| JU > BSC  | USDT  | <p>Charges <strong>1 JU</strong> (platform fee) + <strong>0.002 BNB</strong> (estimated network fee)<br><em>(Estimated total ≈ 1 USD)</em></p>                                |
| JU > BSC  | BNB   | <p>Charges <strong>1 JU</strong> (platform fee) + <strong>0.002 BNB</strong> (estimated network fee)<br><em>(Estimated total ≈ 1 USD)</em></p>                                |
| JU > ETH  | USDT  | <p>Charges <strong>1 JU</strong> (platform fee) + <strong>0.00002 ETH</strong> (estimated network fee)<br><em>(Estimated total ≈ 0.036 USD)</em></p>                          |
| JU > ETH  | ETH   | <p>Charges <strong>1 JU</strong> (platform fee) + <strong>0.00002 ETH</strong> (estimated network fee)<br><em>(Estimated total ≈ 0.036 USD)</em></p>                          |

_Note: The above value estimates are based on prices at a specific time (e.g., BNB=590 USD, ETH=1800 USD, JU=2 USD). Actual fees and values will fluctuate. The platform consumption portion refers to the cost incurred by the platform to process the transaction._

**Fee Receiver Address:**

`0x8C0641240B418e0349dC52abd3F5cEcc4D4C748A`

_Recommendation:_ Before performing a cross-chain operation, you can query the current estimated service fee (typically only the platform fee, excluding target chain Gas) by calling the `bridgeServiceFee()` method on the source chain’s `BridgeBank` contract. The total fee paid should be estimated by the user based on current network conditions and target chain Gas prices.

***
