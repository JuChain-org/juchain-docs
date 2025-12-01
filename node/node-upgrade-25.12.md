# Node Upgrade \[25.12]

### 1. Upgrade Overview

This is a **major version upgrade**. The new data storage format is **not compatible** with previous releases. A **complete blockchain re-sync** is required.\
To ensure stability and minimize service downtime, it is recommended to **deploy a brand-new node**, synchronize it to full state, then **stop the legacy node and migrate services to the upgraded one**.

### 2. Hardware & System Requirements

| Component        | Recommended Spec                    |
| ---------------- | ----------------------------------- |
| CPU              | 8 cores or above                    |
| RAM              | 16GB or above                       |
| Storage          | 1.5TB SSD or above                  |
| Network          | 100Mbps bandwidth or above          |
| Public IP        | Static dedicated public IP required |
| Operating System | Ubuntu 24.04 or above               |

### 3. Pre-Upgrade Preparation

* Prepare and provision the server with required OS.
* Configure a **static dedicated public IP**.
* Contact your support representative to **add the node IP to the whitelist**.

### 4. Upgrade Steps

1. Extract the upgrade package.
2.  Enter the `juchain` directory and initialize the genesis block:

    ```bash
    ./bin/geth --datadir data init config/genesis.json
    ```
3.  Start the node:

    ```bash
    bash start.sh
    ```
4. Check logs under the `logs/` folder to verify normal operation.
5. Wait until the node is **fully synchronized**.
6. Provide external services after full sync.

### ⚠️ Critical Notes

1. ⚠ Do **not activate services on the upgraded node before 2025-12-05 12:00:00 (UTC+8)** — early migration may cause **unstable behavior**.
2. ⚠ Do **not remove or decommission the legacy node before 2025-12-10 12:00:00**, until mainnet fork success is officially confirmed. This ensures safe rollback if needed.

### 5. Rollback Procedure

If the JuChain mainnet fork upgrade **fails**, immediately:

1. Stop the newly deployed syncing node.
2. Continue serving externally using the legacy node.
3. Reach out to your support personnel if assistance is required.

### 6. Upgrade Package

{% file src="../.gitbook/assets/juchain-sync-node-v1.13.15.tgz" %}

> ✅ Verify file integrity using SHA256 or other checksum tools before deployment.

### 7. Support Contact

* For IP whitelisting, sync status verification, or technical assistance, please contact your designated support representative.
