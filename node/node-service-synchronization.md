# Node Service Synchronization

### 1. Recommended Server Configuration

To ensure stable operation and efficient block synchronization, we recommend the following server specs:

| Component | Recommended Configuration         |
| --------- | --------------------------------- |
| CPU       | 8 cores (8c)                      |
| Memory    | 16 GB (16g)                       |
| Storage   | 500 GB SSD                        |
| Network   | ≥100 Mbps (Public IP recommended) |

> ⚠️ If block data grows rapidly or if your app makes heavy use of logs, consider increasing storage capacity.

***

### 2. Node Initialization Process

#### Step 1: Initialize the Genesis Block (Only once)

```bash
./bin/geth --datadir data init config/genesis.json
```

> ⚠️ This step should only be executed once. **Do not reinitialize** after the node is running, as it will **erase all data** and require full resynchronization.

***

#### Step 2: Start the Node Service

```bash
bash start.sh
```

***

#### Step 3: Stop the Node Service

```bash
bash stop.sh
```

***

{% file src="../.gitbook/assets/juchain-sync-node.tgz" %}
