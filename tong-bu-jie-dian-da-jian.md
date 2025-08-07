# 同步节点搭建

### 1. 初始化创世区块

> 注意，本操作只需要执行一次，一旦服务运行起来之后不要再次初始化，否则数据会被覆盖，还需要重头开始同步区块。

执行下面命令初始化创世区块

```shell
./bin/geth --datadir data init config/genesis.json
```

### 2. 启动节点服务

执行下面命令启动节点服务

```shell
bash  start.sh
```

### 3. 停止节点服务

执行下面命令停止节点服务

```shell
bash  stop.sh
```

### 4. 注意事项

> 自建节点从种子节点同步区块需要提前将自建节点的IP申请加入到白名单，否则将无法同步。 请提前联系相关人员进行申请。

{% file src=".gitbook/assets/juchain-sync-node.tgz" %}
