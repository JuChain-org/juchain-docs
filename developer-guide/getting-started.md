# Getting Started

## Getting Started Guide

{% hint style="warning" %}
JuChain currently supports Solidity versions <= 0.8.8 for compilation. Support for later versions will be rolled out based on official announcements.
{% endhint %}

Welcome to your JuChain developer journey! This guide will help you quickly get started with the JuChain platform—from setting up your development environment to deploying your first smart contract and connecting to the JuChain Testnet. Designed for developers new to JuChain, this section offers clear steps and practical examples to ensure you can start building right away.

***

#### Prerequisites

Before diving in, ensure you have the following knowledge and tools:

* **Basic Blockchain Knowledge**: Familiarity with smart contracts, blockchain transactions, and the Ethereum Virtual Machine (EVM).
* **Development Tools**:
  * **Node.js** (v16 or higher recommended): For running JavaScript and managing dependencies.
  * **npm** or **yarn**: Package managers for installing project dependencies.
  * **Code Editor**: Visual Studio Code or a similar tool is recommended.
* **Wallet**: Install MetaMask or another EVM-compatible wallet to manage accounts and testnet tokens.
* **Terminal Skills**: Basic command-line proficiency.

If you’re already familiar with Ethereum development, JuChain’s full EVM compatibility will make your transition seamless.

***

#### Step 1: Set Up Your Development Environment

**Install Essential Tools**

1. **Install Node.js and npm**
   * Download and install the latest Node.js version (includes npm) from [nodejs.org](https://nodejs.org/).
   *   Verify installation:

       ```bash
       node -v
       npm -v
       ```
   * Optionally, install yarn: `npm install -g yarn`.
2. **Install Truffle or Hardhat**\
   JuChain supports popular Ethereum frameworks. Here, we’ll use Hardhat:
   *   Install Hardhat globally:

       ```bash
       npm install -g hardhat
       ```
   *   Create a new Hardhat project:

       ```bash
       npx hardhat
       ```

       Select “Create a basic sample project” and follow the prompts to initialize.
3. **Install MetaMask**
   * Download and install the MetaMask browser extension from [metamask.io](https://metamask.io/).
   * Create or import a wallet account for testnet interactions.

**Configure the JuChain Network**

JuChain offers both mainnet and testnet environments. Below is the testnet configuration:

* **Network Name**: JuChain Testnet
* **RPC URL**: `https://testnet-rpc.juchain.org`
* **Chain ID**: 202599
* **Currency Symbol**: JU
* **Block Explorer URL**: `https://explorer-testnet.juchain.org`

Add the JuChain Testnet to MetaMask:

1. Open MetaMask and click the “Network” dropdown.
2. Select “Add Network” > “Add a network manually”.
3. Enter the details above and save.

***

#### Step 2: Obtain Testnet JU Tokens

You’ll need JU tokens to deploy and test contracts on the JuChain Testnet. Get them via the official faucet:

1. Visit the JuChain Testnet Faucet: [https://faucet-testnet.juchain.org](https://faucet-testnet.juchain.org).
2. Enter your MetaMask wallet address.
3. Click “Claim” to receive test JU (e.g., 10 JU daily).
4. Check your MetaMask balance on the JuChain Testnet to confirm.

***

#### Step 3: Write Your First Smart Contract

Here’s a simple contract example to store and retrieve data on JuChain.

**Create the Smart Contract**

1. In your Hardhat project, navigate to the `contracts` folder.
2.  Create a file named `SimpleStorage.sol` with this code:

    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract SimpleStorage {
        uint256 private storedValue;

        // Store a value
        function set(uint256 _value) public {
            storedValue = _value;
        }

        // Retrieve the stored value
        function get() public view returns (uint256) {
            return storedValue;
        }
    }
    ```

**Configure Hardhat**

1.  Open `hardhat.config.js` and add the JuChain Testnet configuration:

    ```javascript
    require("@nomicfoundation/hardhat-toolbox");

    module.exports = {
      solidity: "0.8.0",
      networks: {
        juchain_testnet: {
          url: "https://testnet-rpc.juchain.org",
          chainId: 202599,
          accounts: ["YOUR_PRIVATE_KEY"] // Replace with your private key (keep secure)
        }
      }
    };
    ```

    * Replace `YOUR_PRIVATE_KEY` with your MetaMask private key (exported from “Account Details”—never share publicly).
2.  Install required Hardhat plugins:

    ```bash
    npm install --save-dev @nomicfoundation/hardhat-toolbox
    ```

**Compile the Smart Contract**

Compile the contract by running:

```bash
npx hardhat compile
```

Upon success, compiled outputs will appear in the `artifacts` folder.

***

#### Step 4: Deploy to JuChain Testnet

**Write a Deployment Script**

1.  In the `scripts` folder, create `deploy.js`:

    ```javascript
    const hre = require("hardhat");

    async function main() {
      const SimpleStorage = await hre.ethers.getContractFactory("SimpleStorage");
      const simpleStorage = await SimpleStorage.deploy();

      await simpleStorage.deployed();
      console.log("SimpleStorage deployed to:", simpleStorage.address);
    }

    main()
      .then(() => process.exit(0))
      .catch((error) => {
        console.error(error);
        process.exit(1);
      });
    ```
2.  Deploy the contract to the JuChain Testnet:

    ```bash
    npx hardhat run scripts/deploy.js --network juchain_testnet
    ```

    *   On success, the terminal will display the contract address, e.g.:

        ```
        SimpleStorage deployed to: 0x1234...abcd
        ```
3. Verify deployment by entering the contract address into the JuChain Testnet Explorer: `https://explorer-testnet.juchain.org`.

***

#### Step 5: Interact with Your Smart Contract

**Test Contract Functions**

1.  Open the Hardhat console:

    ```bash
    npx hardhat console --network juchain_testnet
    ```
2.  Interact with the contract:

    ```javascript
    const SimpleStorage = await ethers.getContractFactory("SimpleStorage");
    const simpleStorage = await SimpleStorage.attach("0x1234...abcd"); // Replace with your contract address
    await simpleStorage.set(42);
    (await simpleStorage.get()).toString();
    ```

    * Expected output: `"42"`, confirming the contract works.

**Frontend Interaction with ethers.js (Optional)**

To interact via a webpage:

1.  Initialize a frontend project:

    ```bash
    npm init -y
    npm install ethers
    ```
2.  Create `index.html` and `app.js`:

    ```javascript
    import { ethers } from "ethers";

    const provider = new ethers.providers.Web3Provider(window.ethereum);
    await provider.send("eth_requestAccounts", []);
    const signer = provider.getSigner();
    const contractAddress = "0x1234...abcd"; // Replace with your contract address
    const abi = [ /* Copy ABI from artifacts/SimpleStorage.json */ ];
    const contract = new ethers.Contract(contractAddress, abi, signer);

    // Set value
    await contract.set(100);
    // Get value
    const value = await contract.get();
    console.log("Stored value:", value.toString());
    ```

***

#### Step 6: Connect to JuChain Testnet Nodes

JuChain provides free public RPC nodes, so developers can build and test without running their own nodes.

**Use Public RPC**

* **Testnet RPC**: `https://testnet-rpc.juchain.org`
* Use this URL directly in Hardhat or other tools to connect to the network.

**Run Your Own Node**

* Not yet available.

***

#### Next Steps

Congratulations on deploying your first smart contract to the JuChain Testnet! Here’s what you can do next:

* Explore the Developer Guide for deeper insights into APIs and traffic finance integration.
* Check out Tutorials and Examples to build more complex dApps.
* Join the JuChain developer community on Discord or Forums for support.

For issues, consult the FAQ or contact the technical support team.

***
