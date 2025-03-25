# Cross-Chain Bridge

## Cross-Chain Bridge

JuChain’s cross-chain bridge is a decentralized service that enables developers to transfer assets between different testnets, such as the JuChain Testnet, BSC Testnet (Chapel), and ETH Testnet (Holesky). It leverages smart contracts—primarily the `BridgeBank` contract—to handle locking, unlocking, minting, and burning operations, ensuring secure and efficient asset transfers across chains.

***

### Testnet Setup

To use the cross-chain bridge, developers need to connect to the following testnets:

* **JuChain Testnet**:
  * RPC: `https://testnet-rpc.juchain.org`
  * Explorer: `https://explorer-testnet.juchain.org`
* **BSC Testnet (Chapel)**:
  * RPC: `https://rpc.ankr.com/bsc_testnet_chapel/b6aba93f145bbc37b4acc1cda2e7c0b38fff5a09fb21a55c1fb65883d35789bd`
  * Note: May require an API key or authentication.
* **ETH Testnet (Holesky)**:
  * RPC: `https://rpc.ankr.com/eth_holesky/b6aba93f145bbc37b4acc1cda2e7c0b38fff5a09fb21a55c1fb65883d35789bd`
  * Explorer: `https://holesky.etherscan.io`

When configuring your wallet or dev tools, use the RPC endpoints above to ensure you’re connected to the right networks.

***

### Contract Addresses

Here are the contract addresses for the cross-chain bridge and related tokens:

#### JuChain Testnet

* **BridgeBank Contract**: `0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE`
* **Tokens**:
  * USDT: `0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D`\
    [View Details](https://explorer-testnet.juchain.org/token/0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D)
  * tBNB: `0x2598d2e226Ce13288E314569569838bBc6Ff9402`\
    [View Details](https://explorer-testnet.juchain.org/token/0x2598d2e226Ce13288E314569569838bBc6Ff9402)
  * tETH: `0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672`\
    [View Details](https://explorer-testnet.juchain.org/token/0x1a4911109be74dc5C9CC8e4AfC3d8D7Fd06CA672)

#### BSC Testnet (Chapel)

* **BridgeBank Contract**: `0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6`
* **Tokens**:
  * USDT: `0xcD1093897a5dB4a9aF153772B35AAA066ab969f3`

#### ETH Testnet (Holesky)

* **BridgeBank Contract**: `0x264960f4bf655c14a74DE1A7fC5AA68E71f71924`\
  [View Details](https://holesky.etherscan.io/address/0x264960f4bf655c14a74DE1A7fC5AA68E71f71924)
* **Tokens**:
  * USDT: `0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add`\
    [View Details](https://holesky.etherscan.io/address/0xc7062D0A7553fabbf0b9B5DF9E9648Cffd2B9add)

***

### BridgeBank Contract Functions

The `BridgeBank` contract is the heart of the cross-chain bridge, offering these key functions for developers to interact with via its ABI:

* **`lock`**: Locks tokens on the source chain for cross-chain transfer.
  * Params: `_recipient` (target chain recipient), `_token` (token address), `_amount` (transfer amount).
  * Example: Lock USDT on JuChain for transfer to BSC.
* **`unlock`**: Unlocks tokens on the target chain.
  * Params: `_recipient` (recipient address), `_token` (token address), `_name` (token name), `_amount` (amount).
* **`mintBridgeTokens`**: Mints bridge tokens on the target chain.
  * Params: `_intendedRecipient` (recipient address), `_bridgeTokenAddress` (bridge token address), `_amount` (amount).
* **`burnBridgeTokens`**: Burns bridge tokens on the source chain to complete the transfer.
  * Params: `_chainID` (target chain ID, JuChain-specific), `_receiver` (recipient address), `_bridgeTokenAddress` (bridge token address), `_amount` (amount).

***

### ABI Data

Below are the ABIs for the `BridgeBank` contracts used on the testnets:

#### JuChain BridgeBank ABI

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

#### ETH/BSC BridgeBank ABI

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

***

### Cross-Chain Process

The JuChain cross-chain bridge supports these workflows:

#### From JuChain to BSC/ETH

1. **Lock Assets**:
   * On JuChain Testnet, users call `burnBridgeTokens` on the `BridgeBank` contract.
   * Params: Target chain ID (BSC: 97, ETH Holesky: 17000), recipient address, token address, amount.
   * This burns tokens on JuChain and emits a `LogBtcTokenBurn` event.
2. **Cross-Chain Validation**:
   * The bridge oracle listens for `LogBtcTokenBurn` events.
   * After verifying the transaction, it triggers minting on the target chain.
3. **Target Chain Minting**:
   * On BSC or ETH Testnet, the oracle calls `mintBridgeTokens` on the `BridgeBank` contract.
   * Equivalent tokens are minted for the recipient.

#### From BSC/ETH to JuChain

1. **Lock Assets**:
   * On BSC or ETH Testnet, users call `lock` on the `BridgeBank` contract.
   * Params: Recipient address, token address, amount.
   * This locks tokens and emits a `LogLock` event.
2. **Cross-Chain Validation**:
   * The bridge oracle listens for `LogLock` events.
   * After verification, it triggers unlocking on JuChain.
3. **JuChain Unlocking**:
   * On JuChain Testnet, the oracle calls `unlock` on the `BridgeBank` contract.
   * Equivalent tokens are unlocked for the recipient.

***

### Code Examples

Here are Web3.js examples for interacting with the bridge:

#### JuChain to BSC Transfer

```javascript
const { Web3 } = require('web3');

// Connect to JuChain Testnet
const web3 = new Web3('https://testnet-rpc.juchain.org');

// BridgeBank ABI and address
const bridgeBankABI = [...]; // Use full JuChain BridgeBank ABI
const bridgeBankAddress = '0x3516949D3c530E4FB65Fa2a02ef808e5587ebaBE';

// Contract instance
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// Account setup
const privateKey = 'your_private_key';
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// ERC20 ABI for balance and approval
const erc20ABI = [
  {"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}
];

// Transfer parameters
const bscChainID = 97; // BSC Testnet Chain ID
const receiverAddress = '0x_recipient_address'; // BSC recipient
const tokenAddress = '0x16E0499Cb600ef4F4FbEca756E90D658D9a74E4D'; // USDT on JuChain
const amount = web3.utils.toWei('10', 'ether'); // 10 tokens

async function crossChainTransfer() {
  try {
    // Check account balance
    const balance = await web3.eth.getBalance(account.address);
    console.log('Balance:', web3.utils.fromWei(balance, 'ether'), 'JU');

    // Check token balance
    const tokenContract = new web3.eth.Contract(erc20ABI, tokenAddress);
    const tokenBalance = await tokenContract.methods.balanceOf(account.address).call();
    if (BigInt(tokenBalance) < BigInt(amount)) {
      throw new Error('Insufficient token balance');
    }

    // Approve token usage
    const allowance = await tokenContract.methods.allowance(account.address, bridgeBankAddress).call();
    if (BigInt(allowance) < BigInt(amount)) {
      console.log('Insufficient allowance, approving...');
      const maxApproval = '115792089237316195423570985008687907853269984665640564039457584007913129639935'; // Max uint256
      await tokenContract.methods.approve(bridgeBankAddress, maxApproval).send({
        from: account.address,
        gas: 200000
      });
      console.log('Approval successful');
    }

    // Get bridge service fee
    let fee = await bridgeBank.methods.bridgeServiceFee().call().catch(() => web3.utils.toWei('0.01', 'ether')); // Fallback to 0.01 JU
    console.log('Service Fee:', web3.utils.fromWei(fee, 'ether'), 'JU');
    fee = fee.toString(); // Convert BigInt to string

    // Estimate Gas
    let estimatedGas = await bridgeBank.methods.burnBridgeTokens(bscChainID, receiverAddress, tokenAddress, amount)
      .estimateGas({ from: account.address, value: fee }).catch(() => 500000); // Fallback to 500k
    estimatedGas = Number(estimatedGas);

    // Execute burnBridgeTokens
    const tx = await bridgeBank.methods.burnBridgeTokens(bscChainID, receiverAddress, tokenAddress, amount).send({
      from: account.address,
      value: fee,
      gas: Math.floor(estimatedGas * 1.5), // 50% buffer
      maxPriorityFeePerGas: web3.utils.toWei('10', 'gwei'),
      maxFeePerGas: web3.utils.toWei('100', 'gwei')
    });

    console.log('Cross-chain Tx Hash:', tx.transactionHash);
  } catch (error) {
    console.error('Cross-chain transfer failed:', error);
  }
}

crossChainTransfer();
```

#### BSC to JuChain Transfer

```javascript
const { Web3 } = require('web3');

// Connect to BSC Testnet
const web3 = new Web3('https://data-seed-prebsc-1-s1.binance.org:8545');

// BridgeBank ABI and address
const bridgeBankABI = [...]; // Use full BSC BridgeBank ABI
const bridgeBankAddress = '0x30DBF30Eb71ddb49d526AFdb832C7Ba4D85953f6';

// Contract instance
const bridgeBank = new web3.eth.Contract(bridgeBankABI, bridgeBankAddress);

// Account setup
const privateKey = 'your_private_key';
const account = web3.eth.accounts.privateKeyToAccount(privateKey);
web3.eth.accounts.wallet.add(account);

// ERC20 ABI
const erc20ABI = [
  {"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
  {"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
  {"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}
];

// Transfer parameters
const receiverAddress = '0x_recipient_address'; // JuChain recipient
const tokenAddress = '0xcD1093897a5dB4a9aF153772B35AAA066ab969f3'; // USDT on BSC
const amount = web3.utils.toWei('10', 'ether'); // 10 tokens

async function crossChainTransfer() {
  try {
    // Check BSC balance
    const balance = await web3.eth.getBalance(account.address);
    console.log('BSC Balance:', web3.utils.fromWei(balance, 'ether'), 'BNB');

    // Check token balance
    const tokenContract = new web3.eth.Contract(erc20ABI, tokenAddress);
    const tokenBalance = await tokenContract.methods.balanceOf(account.address).call();
    if (BigInt(tokenBalance) < BigInt(amount)) {
      throw new Error('Insufficient token balance');
    }

    // Approve token usage
    const allowance = await tokenContract.methods.allowance(account.address, bridgeBankAddress).call();
    if (BigInt(allowance) < BigInt(amount)) {
      console.log('Insufficient allowance, approving...');
      const approvalAmount = web3.utils.toWei('100', 'ether');
      await tokenContract.methods.approve(bridgeBankAddress, approvalAmount).send({
        from: account.address,
        gas: 200000,
        gasPrice: web3.utils.toWei('10', 'gwei')
      });
      console.log('Approval successful');
    }

    // Get bridge service fee
    let fee = await bridgeBank.methods.bridgeServiceFee().call().catch(() => web3.utils.toWei('0.01', 'ether')); // Fallback to 0.01 BNB
    console.log('Service Fee:', web3.utils.fromWei(fee, 'ether'), 'BNB');
    fee = fee.toString();

    // Estimate Gas
    let estimatedGas = await bridgeBank.methods.lock(receiverAddress, tokenAddress, amount)
      .estimateGas({ from: account.address, value: fee }).catch(() => 500000); // Fallback to 500k
    estimatedGas = Number(estimatedGas);

    // Execute lock
    const tx = await bridgeBank.methods.lock(receiverAddress, tokenAddress, amount).send({
      from: account.address,
      value: fee,
      gas: Math.floor(estimatedGas * 1.5), // 50% buffer
      gasPrice: web3.utils.toWei('10', 'gwei')
    });

    console.log('Cross-chain Tx Hash:', tx.transactionHash);
    console.log('Transfer submitted, awaiting bridge processing to JuChain...');
  } catch (error) {
    console.error('Cross-chain transfer failed:', error);
  }
}

crossChainTransfer();
```

***

### Notes

1. **Testnet Limitations**:
   * These contracts and tokens are testnet-only and not for mainnet use.
   * Testnets may reset periodically—don’t store valuable assets here.
2. **Service Fees**:
   * Cross-chain actions require a fee, checkable via `bridgeServiceFee`.
   * Fees are paid in native tokens (JU, BNB, or ETH).
3. **Approval Requirement**:
   * Before locking ERC20 tokens, approve the `BridgeBank` contract using the token’s `approve` function.
4. **Cross-Chain Delay**:
   * Transfers take time based on oracle processing speed—be patient and check status via block explorers.
5. **Token Compatibility**:
   * Only whitelisted tokens (currently USDT, tBNB, tETH) are supported for bridging.
6. **Error Handling**:
   * If a transfer fails, verify parameters, token balance, and fee sufficiency.
