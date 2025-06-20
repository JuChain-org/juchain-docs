# Quick Start

### Network Configuration <a href="#network-configuration" id="network-configuration"></a>

### Network Information

**JuChain Mainnet**

| Name            | Value                                                  |
| --------------- | ------------------------------------------------------ |
| Network Name    | JuChain Mainnet                                        |
| Description     | JuChain Public Mainnet                                 |
| RPC Endpoint    | <p>https://rpc.juchain.org<br>wss://ws.juchain.org</p> |
| Chain ID        | 210000                                                 |
| Currency Symbol | JU                                                     |
| Block Explorer  | https://juscan.io                                      |

**JuChain Testnet**

| Name            | Value                                                                    |
| --------------- | ------------------------------------------------------------------------ |
| Network Name    | JuChain Testnet                                                          |
| Description     | JuChain Public Testnet                                                   |
| RPC Endpoint    | <p>https://testnet-rpc.juchain.org</p><p>ws://testnet-ws.juchain.org</p> |
| Chain ID        | 202599                                                                   |
| Currency Symbol | JU                                                                       |
| Block Explorer  | https://testnet.juscan.io                                                |



**Hardhat​**

Modify your Hardhat config file `hardhat.config.ts` to point at the Juchain Testnet public RPC.

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

**Foundry​**

To deploy using the Juchain Testnet Public RPC, run:

```
forge create ... --rpc-url=https://testnet-rpc.juchain.org --legacy
```

**Remix Web IDE​**

After compiling your contracts, the easiest way to deploy using Remix is by [setting up Metamask](https://docs.bitlayer.org/user-guide/setup#metamask), then selecting the **Juchain Testnet** network.

In the “Deploy and Run Transactions” tab, use the “Environment” drop-down and select “Injected Provider - MetaMask.”

Connect your wallet and select the Juchain Testnet. Your account should be selected automatically in Remix, and you can click “Deploy.”

**Truffle​**

caution

The Truffle Suite is being **sunset**. For information on ongoing support, migration options and FAQs, visit the [Consensys blog](https://consensys.io/blog/consensys-announces-the-sunset-of-truffle-and-ganache-and-new-hardhat?utm_source=github\&utm_medium=referral\&utm_campaign=2023_Sep_truffle-sunset-2023_announcement_).

Assuming you already have a Truffle environment setup, go to the Truffle [configuration file](https://trufflesuite.com/docs/truffle/reference/configuration/), `truffle.js`. Make sure to have installed HDWalletProvider: `npm install @truffle/hdwallet-provider@1.4.0`

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

**Brownie​**

To add the Juchain Testnet, run the following command:

```
brownie networks add Juchain host=https://testnet-rpc.juchain.org chainid=202599 
```

To set this as your default network, add the following in your project config file:

```
networks:
  default: Juchain
```

Another way to add the Juchain Testnet is to create a `yaml` file and run a command to add it.

This is an example of a yaml file called `network-config.yaml`

```
live:
- name: Juchain
 networks:
 - chainid: 202599
   explorer: https://testnet.juscan.io
   host: https://testnet-rpc.juchain.org
   id: juchain
   name: Juchain Testnet
```

To add the Juchain Testnet to the network list, run the following command:

```
brownie networks import ./network-config.yaml
```

To deploy on Juchain, run the following command. In this example, `token.py` is the script to deploy the smart contract. Replace this with the name of your script:

```
brownie run token.py --network Juchain
```

**ethers.js​**

Setting up a Juchain Testnet provider in an `ethers` script:

```
import { ethers } from "ethers"

const provider = new ethers.providers.JsonRpcProvider("https://testnet-rpc.juchain.org")
```

**scaffold-eth​**

To deploy using Scaffold-eth, you’ll need to point both your Hardhat and React settings at the Juchain Testnet.

**Configure Hardhat​**

In the `packages/hardhat/hardhat.config.js` file, you’ll add the network and select it as the default network.

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

Be sure to fund the deployment wallet as well! Run `yarn generate` to create the wallet and `yarn account` to check its funds. Once funded, run `yarn deploy --network Juchain` to deploy on the Juchain testnet.

