# 网络信息

## 网络信息

### JuChain 主网

| 名称 | 值 |
| --- | --- |
| 网络名称 | JuChain Mainnet |
| 描述 | JuChain 公共主网 |
| RPC 端点 | 待定 |
| 链 ID | 待定 |
| 货币符号 | ETH |
| 区块浏览器 | http://explorer.juchain.org/ |

### JuChain 测试网

| 名称 | 值 |
| --- | --- |
| 网络名称 | JuChain Testnet |
| 描述 | JuChain 公共测试网 |
| RPC 端点 | https://testnet-rpc.juchain.org |
| 链 ID | 66633666 |
| 货币符号 | JU |
| 区块浏览器 | http://explorer-testnet.juchain.org/ |

### 重要信息

L1（以太坊）、L2（OP Mainnet）和 L3（JuChain）的智能合约部署信息可以在[合约页面](TODO:link)找到。

### 节点服务

对于生产系统，我们建议：

1. 使用我们节点合作伙伴提供的服务
2. 运行您自己的 JuChain 节点
3. 使用我们的高可用性 RPC 服务（待定详情）

### 网络架构

作为基于 OP Mainnet 的 Layer 3 解决方案，JuChain 采用以下架构：

1. 数据可用性层：以太坊 L1
2. 共识层：OP Mainnet
3. 执行层：JuChain

这种架构设计带来以下优势：

* 继承以太坊的安全性
* 利用 OP Mainnet 的高性能
* 实现更低的交易成本
* 保持完全的 EVM 兼容性

### 跨链桥接

JuChain 支持以下网络之间的资产转移：

1. 以太坊主网
2. OP Mainnet
3. 其他主要 L2 网络（待定具体列表）

桥接服务由以下机制保护：

* 标准化消息协议
* 多重签名验证者网络
* 自动安全监控
* 紧急暂停机制

### 网络升级

JuChain 遵循透明的网络升级流程：

1. 提案期（2周）
2. 投票期（1周）
3. 准备期（1周）
4. 执行期（计划升级）

所有重要的网络参数调整和协议升级都将遵循此流程。 