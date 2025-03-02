# Network Information



#### JuChain Mainnet

| Name            | Value                        |
| --------------- | ---------------------------- |
| Network Name    | JuChain Mainnet              |
| Description     | JuChain Public Mainnet       |
| RPC Endpoint    | TBD                          |
| Chain ID        | TBD                          |
| Currency Symbol | JU                           |
| Block Explorer  | http://explorer.juchain.org/ |

#### JuChain Testnet

| Name            | Value                                |
| --------------- | ------------------------------------ |
| Network Name    | JuChain Testnet                      |
| Description     | JuChain Public Testnet               |
| RPC Endpoint    | https://testnet-rpc.juchain.org      |
|                 | ws://testnet-ws.juchain.org          |
| Chain ID        | 66633666                             |
| Currency Symbol | JU                                   |
| Block Explorer  | http://explorer-testnet.juchain.org/ |

#### Important Information

Smart contract deployment information for L1 (Ethereum), L2 (OP Mainnet), and L3 (JuChain) can be found on the [Contracts page](TODO:link).

#### Node Services

For production systems, we recommend:

1. Using services provided by our node partners
2. Running your own JuChain node
3. Using our high-availability RPC service (TODO: details pending)

#### Network Architecture

As a Layer 3 solution based on OP Mainnet, JuChain adopts the following architecture:

1. Data Availability Layer: Ethereum L1
2. Consensus Layer: OP Mainnet
3. Execution Layer: JuChain

This architectural design brings the following advantages:

* Inherits Ethereum's security
* Leverages OP Mainnet's high performance
* Achieves lower transaction costs
* Maintains full EVM compatibility

#### Cross-chain Bridging

JuChain supports asset transfers between the following networks:

1. Ethereum Mainnet
2. OP Mainnet
3. Other major L2 networks (TODO: specific list)

Bridge services are secured by the following mechanisms:

* Standardized messaging protocol
* Multi-signature validator network
* Automated security monitoring
* Emergency pause mechanism

#### Network Upgrades

JuChain follows a transparent network upgrade process:

1. Proposal Period (2 weeks)
2. Voting Period (1 week)
3. Preparation Period (1 week)
4. Execution Period (scheduled upgrade)

All important network parameter adjustments and protocol upgrades will follow this process.
