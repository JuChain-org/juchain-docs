# 快速开始



{% hint style="warning" %}
JuChain 当前仅支持 Solidity 版本 <= 0.8.8 进行编译，后续版本支持将根据官方通知逐步开放。
{% endhint %}

## 配置网络

### 网络信息

#### JuChain 主网

| 名称     | 值                                                      |
| ------ | ------------------------------------------------------ |
| 网络名称   | JuChain Mainnet                                        |
| 描述     | JuChain 公共主网                                           |
| RPC 端点 | <p>https://rpc.juchain.org<br>wss://ws.juchain.org</p> |
| 链 ID   | 210000                                                 |
| 货币符号   | JU                                                     |
| 区块浏览器  | https://juscan.io                                      |

#### JuChain 测试网

| 名称     | 值                                                                         |
| ------ | ------------------------------------------------------------------------- |
| 网络名称   | JuChain Testnet                                                           |
| 描述     | JuChain 公共测试网                                                             |
| RPC 端点 | <p>https://testnet-rpc.juchain.org </p><p>ws://testnet-ws.juchain.org</p> |
| 链 ID   | 202599                                                                    |
| 货币符号   | JU                                                                        |
| 区块浏览器  | https://testnet.juscan.io                                                 |

## 部署

### **Hardhat​** &#x20;

\
修改您的 Hardhat 配置文件 `hardhat.config.ts` 以指向 Juchain Testnet 公共 RPC。

```
const config: HardhatUserConfig = {
  ...
  networks: {
    jucoin: {
      url: "https://testnet-rpc.juchain.org" || "",
      accounts:
        process.env.PRIVATE_KEY !== undefined ? [process.env.PRIVATE_KEY] : [],
    },
  },
};
```

### **Foundry​**&#x20;

\
要使用 Juchain Testnet Public RPC 进行部署，请运行：

```
forge create ... --rpc-url=https://testnet-rpc.juchain.org --legacy
```

### **Remix Web IDE​** &#x20;

编译合约后，使用 Remix 部署的最简单方法是[设置 Metamask](https://docs.bitlayer.org/user-guide/setup#metamask) ，然后选择 **Juchain Testnet** 网络。

在“部署和运行交易”选项卡中，使用“环境”下拉菜单并选择“注入提供程序 - MetaMask”。\
连接你的钱包并选择 Juchain 测试网。你的账户应该会在 Remix 中自动选中，点击“部署”。

### **Truffle​** &#x20;

警告\
Truffle Suite 即将**停用** 。有关持续支持、迁移选项和常见问题解答，请访问 [Consensys 博客 ](https://consensys.io/blog/consensys-announces-the-sunset-of-truffle-and-ganache-and-new-hardhat?utm_source=github\&utm_medium=referral\&utm_campaign=2023_Sep_truffle-sunset-2023_announcement_)。

\
假设你已经设置了 Truffle 环境，请转到 Truffle[ 配置文件 ](https://trufflesuite.com/docs/truffle/reference/configuration/)， `truffle.js` 。确保已安装 HDWalletProvider： `npm install @truffle/hdwallet-provider@1.4.0`

```
const HDWalletProvider = require("@truffle/hdwallet-provider")
...
module.exports = {
  networks: {
    juchain: {
      provider: () =>
        new HDWalletProvider(process.env.PRIVATE_KEY, "https://testnet-rpc.juchain.org"),
      network_id: '202599',
    },
  }
}
```

### **Brownie​**&#x20;

要添加 Juchain 测试网，请运行以下命令：

```
brownie networks add Juchain host=https://testnet-rpc.juchain.org chainid=202599 
```

要将其设置为默认网络，请在项目配置文件中添加以下内容：

```
networks:
  default: Juchain


```

<pre><code><strong>添加 Juchain 测试网的另一种方法是创建一个 yaml 文件并运行命令来添加它。
</strong></code></pre>

\
这是名为 `network-config.yaml` 的 yaml 文件的示例

```
live:
- name: Juchain
 networks:
 - chainid: 202599
   explorer: https://testnet.juscan.io
   host: https://testnet-rpc.juchain.org
   id: juchain
   name: Juchain Testnet
   要将 Juchain 测试网添加到网络列表，请运行以下命令：
```

```
brownie networks import ./network-config.yaml
```

要在 Juchain 上部署，请运行以下命令。本例中， `token.py` 是部署智能合约的脚本。请将其替换为您的脚本名称：

```
brownie run token.py --network Juchain
```

### **ethers.js​**&#x20;

在 `ethers` 脚本中设置 Juchain 测试网提供商：

```
import { ethers } from "ethers"

const provider = new ethers.providers.JsonRpcProvider("https://testnet-rpc.juchain.org")
```

### **scaffold-eth​**

要使用 Scaffold-eth 进行部署，您需要将 Hardhat 和 React 设置指向 Juchain Testnet。

### **配置 Hardhat​**&#x20;

在 `packages/hardhat/hardhat.config.js` 文件中，您将添加网络并选择它作为默认网络。

```
...
//
// Select the network you want to deploy to here:
//
const defaultNetwork = "Juchain";
...
module.exports = {
...
	networks: {
...
          Juchain: {
            url: "https://testnet-rpc.juchain.org",
            accounts: {
              mnemonic: mnemonic(),
            },
          },
	}
...
}
```

确保也为部署钱包充值！运行 `yarn generate` 创建钱包，并创建 `yarn account` 来检查资金。充值完成后，运行 `yarn deploy --network Juchain` 即可在 Juchain 测试网上进行部署。

