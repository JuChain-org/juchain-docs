# Tutorials and Examples: Building and Deploying a Simple dApp

This tutorial will guide you through creating a simple decentralized application (dApp) on JuChain: a "Greeting" smart contract where users can set and read custom greetings. We'll write the contract in Solidity, deploy it to the JuChain Testnet using Truffle, and build a frontend interface to interact with it via MetaMask. The tutorial showcases JuChain’s development workflow, fast transaction confirmations, and low-cost benefits.

***

#### Prerequisites

* **Tools**:
  * Node.js (v16 or higher, download: [https://nodejs.org/](https://nodejs.org/))
  * Truffle Suite (install globally: `npm install -g truffle`)
  * MetaMask browser extension (install: [https://metamask.io/](https://metamask.io/))
* **JuChain Testnet**:
  * RPC Endpoint: `https://testnet-rpc.juchain.org`
  * Network ID: 202599
  * Test JU Tokens: Obtain via the JuChain Testnet faucet (assumed: `https://faucet-testnet.juchain.org`)
* **Knowledge**: Basic understanding of JavaScript, Solidity, and blockchain concepts.

***

#### Step 1: Set Up the Project

1.  **Create and Initialize the Project**:

    ```bash
    mkdir JuChainGreeting
    cd JuChainGreeting
    truffle init
    ```

    This creates folders like `contracts/`, `migrations/`, and `truffle-config.js`.
2.  **Install Dependencies**:

    ```bash
    npm install @openzeppelin/contracts web3 dotenv
    ```

    * `@openzeppelin/contracts`: Secure contract templates.
    * `web3`: Frontend-blockchain interaction.
    * `dotenv`: Environment variable management.
3.  **Create a `.env` File**:\
    In the project root, create `.env`:

    ```plaintext
    MNEMONIC="your MetaMask mnemonic"
    RPC_URL="https://testnet-rpc.juchain.org"
    ```

    > **Security Note**: Never commit `.env` to GitHub.

***

#### Step 2: Write the Smart Contract

In the `contracts/` folder, create `Greeting.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Greeting {
    string public greeting;

    event GreetingSet(address indexed user, string newGreeting);

    constructor() {
        greeting = "Hello, JuChain!";
    }

    function setGreeting(string memory _greeting) public {
        require(bytes(_greeting).length > 0, "Greeting cannot be empty");
        greeting = _greeting;
        emit GreetingSet(msg.sender, _greeting);
    }

    function getGreeting() public view returns (string memory) {
        return greeting;
    }
}
```

**Explanation**:

* **`greeting`**: Public variable storing the greeting.
* **`setGreeting`**: Sets a new greeting and emits an event.
* **`getGreeting`**: Retrieves the current greeting.
* **Event**: Logs each greeting change for frontend monitoring.

***

#### Step 3: Configure Truffle

Edit `truffle-config.js` to include the JuChain Testnet:

```javascript
require("dotenv").config();
const HDWalletProvider = require("@truffle/hdwallet-provider");

const { MNEMONIC, RPC_URL } = process.env;

module.exports = {
  networks: {
    juchain_testnet: {
      provider: () => new HDWalletProvider(MNEMONIC, RPC_URL),
      network_id: 202599, // JuChain Testnet ID
      gas: 5500000,     // Gas limit
      gasPrice: 1000000000, // 1 gwei
      confirmations: 2, // Wait for 2 block confirmations
      timeoutBlocks: 200, // Timeout after 200 blocks
    },
  },
  compilers: {
    solc: {
      version: "0.8.0", // Solidity version
      settings: {
        optimizer: {
          enabled: true,
          runs: 200,
        },
      },
    },
  },
};
```

**Checkpoint**:

* Ensure `MNEMONIC` and `RPC_URL` in `.env` are correctly set.
*   Test the network connection:

    ```bash
    truffle console --network juchain_testnet
    > web3.eth.getBlockNumber()
    ```

    A block number output confirms a successful connection.

***

#### Step 4: Deploy the Contract

1.  **Create a Deployment Script**:\
    In `migrations/`, create `2_deploy_greeting.js`:

    ```javascript
    const Greeting = artifacts.require("Greeting");

    module.exports = function (deployer) {
      deployer.deploy(Greeting);
    };
    ```
2.  **Deploy to JuChain Testnet**:

    ```bash
    truffle migrate --network juchain_testnet
    ```

    *   Example output:

        ```
        2_deploy_greeting.js
        ====================
        Deploying 'Greeting'
        --------------------
        > transaction hash: 0x...
        > contract address: 0x1234567890abcdef1234567890abcdef12345678
        ```
    * Note the contract address (e.g., `0x1234...5678`) for later use.
3.  **Verify Deployment**:\
    In the Truffle console:

    ```bash
    truffle console --network juchain_testnet
    > let greeting = await Greeting.at("0x1234567890abcdef1234567890abcdef12345678")
    > await greeting.getGreeting()
    ```

    Expected output: `"Hello, JuChain!"`.

***

#### Step 5: Create the Frontend Interface

1.  **Initialize the Frontend Folder**:

    ```bash
    mkdir public && cd public
    touch index.html
    ```
2.  **Write `index.html`**:\
    Add the following to `public/index.html`:

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>JuChain Greeting dApp</title>
      <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
        input { padding: 5px; margin: 10px; width: 200px; }
        button { padding: 10px 20px; margin: 5px; }
        #status { color: green; }
      </style>
    </head>
    <body>
      <h1>JuChain Greeting dApp</h1>
      <p>Current Greeting: <span id="greeting">Loading...</span></p>
      <input id="newGreeting" type="text" placeholder="Enter new greeting" />
      <br />
      <button onclick="setGreeting()">Set Greeting</button>
      <p id="status"></p>

      <script src="https://cdn.jsdelivr.net/npm/web3@1.5.3/dist/web3.min.js"></script>
      <script>
        const contractAddress = "0x1234567890abcdef1234567890abcdef12345678"; // Replace with your contract address
        const abi = [
          {
            "inputs": [],
            "stateMutability": "nonpayable",
            "type": "constructor"
          },
          {
            "anonymous": false,
            "inputs": [
              {"indexed": true, "internalType": "address", "name": "user", "type": "address"},
              {"indexed": false, "internalType": "string", "name": "newGreeting", "type": "string"}
            ],
            "name": "GreetingSet",
            "type": "event"
          },
          {
            "inputs": [],
            "name": "getGreeting",
            "outputs": [{"internalType": "string", "name": "", "type": "string"}],
            "stateMutability": "view",
            "type": "function"
          },
          {
            "inputs": [{"internalType": "string", "name": "_greeting", "type": "string"}],
            "name": "setGreeting",
            "outputs": [],
            "stateMutability": "nonpayable",
            "type": "function"
          },
          {
            "inputs": [],
            "name": "greeting",
            "outputs": [{"internalType": "string", "name": "", "type": "string"}],
            "stateMutability": "view",
            "type": "function"
          }
        ];
        let web3, contract;

        async function init() {
          if (window.ethereum) {
            web3 = new Web3(window.ethereum);
            try {
              await window.ethereum.request({ method: "eth_requestAccounts" });
              contract = new web3.eth.Contract(abi, contractAddress);
              updateGreeting();
            } catch (error) {
              console.error("User denied account access", error);
              document.getElementById("status").innerText = "Please connect MetaMask";
            }
          } else {
            alert("Please install MetaMask!");
          }
        }

        async function updateGreeting() {
          const greeting = await contract.methods.getGreeting().call();
          document.getElementById("greeting").innerText = greeting;
        }

        async function setGreeting() {
          const accounts = await web3.eth.getAccounts();
          const newGreeting = document.getElementById("newGreeting").value;
          if (!newGreeting) {
            document.getElementById("status").innerText = "Greeting cannot be empty!";
            return;
          }
          document.getElementById("status").innerText = "Setting greeting...";
          try {
            await contract.methods.setGreeting(newGreeting).send({ from: accounts[0] });
            document.getElementById("status").innerText = "Greeting updated successfully!";
            updateGreeting();
          } catch (error) {
            document.getElementById("status").innerText = "Error: " + error.message;
          }
        }

        window.onload = init;
      </script>
    </body>
    </html>
    ```
3. **Obtain the ABI**:
   * Copy the `"abi"` field from `build/contracts/Greeting.json` and replace the `abi` variable in `index.html`.
   * Update `contractAddress` with the address from Step 4.

***

#### Step 6: Run and Test

1.  **Start a Local Server**:

    ```bash
    npx http-server ./public
    ```

    * Access `http://localhost:8080` by default.
2. **Connect MetaMask**:
   * Add the JuChain Testnet to MetaMask:
     * Network Name: JuChain Testnet
     * RPC URL: `https://testnet-rpc.juchain.org`
     * Chain ID: 202599
     * Currency Symbol: JU
   * Ensure your account has test JU tokens (via the faucet).
3. **Test the dApp**:
   * Open `http://localhost:8080` in your browser.
   * Verify the initial greeting displays as `"Hello, JuChain!"`.
   * Enter a new greeting (e.g., `"Welcome to JuChain!"`) and click “Set Greeting”.
   * Confirm the status updates to “Greeting updated successfully!” and the new greeting appears.
   * Check MetaMask: Transactions should confirm in 3-6 seconds with minimal fees (\~0.001 JU).

***

#### Outcome

You’ve successfully built and deployed a simple “Greeting” dApp:

* **Smart Contract**: Deployed on the JuChain Testnet, supporting greeting updates and retrieval.
* **Frontend Interface**: Interacts with the contract via MetaMask, displaying results intuitively.
* **JuChain Advantages**:
  * **Fast Confirmation**: Transactions complete in seconds, enhancing user experience.
  * **Low Cost**: Setting a greeting incurs minimal fees, ideal for frequent interactions.
  * **EVM Compatibility**: Leverages familiar Ethereum tools (Truffle, Web3.js) for seamless development.

***

#### Notes

* **Network Status**: Ensure the JuChain Testnet RPC is operational; contact JuChain support if issues arise.
* **Gas Fees**: Testnet costs are low, but estimate mainnet expenses before production deployment.

***
