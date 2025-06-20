# Developer FAQ

\
**1. What are the RPC addresses of JuChain mainnet and testnet?**

* Mainnet RPC: https://rpc.juchain.org
* Mainnet WebSocket: wss://ws.juchain.org
* Testnet RPC: https://testnet-rpc.juchain.org
* Testnet WebSocket: ws://testnet-ws.juchain.org

#### 2. What is JuChain’s Chain ID? <a href="#id-2.-juchain-de-chain-id-shi-duo-shao" id="id-2.-juchain-de-chain-id-shi-duo-shao"></a>

* Mainnet Chain ID: 210000
* Testnet Chain ID: 202599

#### 3. How to configure JuChain network in Hardhat? <a href="#id-3.-ru-he-zai-hardhat-zhong-pei-zhi-juchain-wang-luo" id="id-3.-ru-he-zai-hardhat-zhong-pei-zhi-juchain-wang-luo"></a>

```
networks: {
  jucoin: {
    url: "https://testnet-rpc.juchain.org",
    accounts: [PRIVATE_KEY]
  }
}
```

&#x20;Replace PRIVATE\_KEY with your wallet private key.

#### 4. How to configure JuChain network in Truffle? <a href="#id-4.-ru-he-zai-truffle-zhong-pei-zhi-juchain-wang-luo" id="id-4.-ru-he-zai-truffle-zhong-pei-zhi-juchain-wang-luo"></a>

```
const HDWalletProvider = require("@truffle/hdwallet-provider");
module.exports = {
  networks: {
    juchain: {
      provider: () => new HDWalletProvider(PRIVATE_KEY, "https://testnet-rpc.juchain.org"),
      network_id: '202599',
    },
  }
}
```

#### 5. What should I do if “insufficient funds” is reported during contract deployment? <a href="#id-5.-he-yue-bu-shu-shi-bao-insufficient-funds-zen-me-ban" id="id-5.-he-yue-bu-shu-shi-bao-insufficient-funds-zen-me-ban"></a>

Please make sure your wallet address has enough JU tokens on the JuChain testnet. You can claim them through [the testnet faucet](https://juchain.gitbook.io/juchain-docs/zh/ecosystem/ce-shi-wang-shui-long-tou) .

#### 6. How to deploy a contract on Remix to JuChain? <a href="#id-6.-ru-he-zai-remix-shang-bu-shu-he-yue-dao-juchain" id="id-6.-ru-he-zai-remix-shang-bu-shu-he-yue-dao-juchain"></a>

* Add JuChain testnet in MetaMask.
* Select the “Injected Provider - MetaMask” environment.
* Once the wallet is connected, it can be deployed.

#### 7. How to add JuChain network in Brownie? <a href="#id-7.-ru-he-zai-brownie-zhong-tian-jia-juchain-wang-luo" id="id-7.-ru-he-zai-brownie-zhong-tian-jia-juchain-wang-luo"></a>

`brownie networks add Juchain host=https://testnet-rpc.juchain.org chainid=202599`

#### 8. Why did the contract verification fail? <a href="#id-8.-wei-shen-me-he-yue-yan-zheng-shi-bai" id="id-8.-wei-shen-me-he-yue-yan-zheng-shi-bai"></a>

* Please ensure that the source code you upload is exactly the same as that you deployed.
* Check whether the compiler version, optimization parameters and other settings are consistent.
* &#x20;Refer to the contract source code verification practice.

#### 9. What Solidity versions does JuChain support? <a href="#id-9.-juchain-zhi-chi-na-xie-solidity-ban-ben" id="id-9.-juchain-zhi-chi-na-xie-solidity-ban-ben"></a>

JuChain is compatible with EVM and it is recommended to use Solidity 0.8.x and above.

#### &#x20;10. How can I get more technical support? <a href="#id-10.-ru-he-huo-qu-geng-duo-ji-shu-zhi-chi" id="id-10.-ru-he-huo-qu-geng-duo-ji-shu-zhi-chi"></a>

* Visit JuChain Community and Support
* Join the official Telegram and Discord.
