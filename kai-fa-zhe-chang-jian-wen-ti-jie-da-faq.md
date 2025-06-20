# 开发者常见问题解答（FAQ）

### 1. JuChain 主网和测试网的 RPC 地址是多少？

* 主网 RPC: https://rpc.juchain.org
* 主网 WebSocket: wss://ws.juchain.org
* 测试网 RPC: https://testnet-rpc.juchain.org
* 测试网 WebSocket: ws://testnet-ws.juchain.org

### 2. JuChain 的 Chain ID 是多少？

* 主网 Chain ID: 210000
* 测试网 Chain ID: 202599

### 3. 如何在 Hardhat 中配置 JuChain 网络？

```
networks: {
  jucoin: {
    url: "https://testnet-rpc.juchain.org",
    accounts: [PRIVATE_KEY]
  }
}
```

将 PRIVATE\_KEY 替换为你的钱包私钥。

### 4. 如何在 Truffle 中配置 JuChain 网络？

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

### 5. 合约部署时报 “insufficient funds” 怎么办？

请确保你的钱包地址在 JuChain 测试网有足够的 JU 代币。可以通过 [测试网水龙头](ecosystem/ce-shi-wang-shui-long-tou.md) 领取。

### 6. 如何在 Remix 上部署合约到 JuChain？

* 在 MetaMask 添加 JuChain 测试网。
* 选择 “Injected Provider - MetaMask” 环境。
* 连接钱包后即可部署。

### 7. 如何在 Brownie 中添加 JuChain 网络？

`brownie networks add Juchain host=https://testnet-rpc.juchain.org chainid=202599`

### 8. 为什么合约验证失败？

* 请确保上传的源码与部署时完全一致。
* 检查编译器版本、优化参数等设置是否一致。
* 参考合约源码验证实践。

### 9. JuChain 支持哪些 Solidity 版本？

JuChain 当前仅支持 Solidity 版本 <= 0.8.8 进行编译，后续版本支持将根据官方通知逐步开放。

### 10. 如何获取更多技术支持？

* 访问 JuChain 社区与支持



\
