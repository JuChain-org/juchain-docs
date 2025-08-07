# Node Service Synchronization

### 1. Initialize the Genesis Block

> Note: This operation only needs to be performed **once**. Do **not** reinitialize after the service is running, otherwise all data will be **overwritten**, and you will need to resynchronize blocks from the beginning.

Run the following command to initialize the genesis block:

```shell
./bin/geth --datadir data init config/genesis.json
```

### 2. Start the Node Service

Run the following command to start the node service:

```shell
bash start.sh
```

### 3. Stop the Node Service

Run the following command to stop the node service:

```shell
bash stop.sh
```

### 4. Important Notes

> To synchronize blocks from seed nodes, the IP address of your self-hosted node must be added to the **whitelist** in advance. Otherwise, synchronization will fail.\
> Please contact the relevant personnel to apply for whitelisting beforehand.

{% file src=".gitbook/assets/juchain-sync-node.tgz" %}
