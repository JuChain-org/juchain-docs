# Contract Verification on JuChain using the Hardhat framework

### 1. Environment Setup

Ensure your Hardhat development environment is ready, especially the configurations related to verification.

#### 1.1 Core Plugin: `@nomicfoundation/hardhat-verify`

This plugin handles interactions with the block explorer's API. Add it to your project's development dependencies using npm or yarn:

```bash
npm install --save-dev @nomicfoundation/hardhat-verify
# or
yarn add --dev @nomicfoundation/hardhat-verify
```

#### 1.2 `hardhat.config.js` Configuration Details

Contract verification configuration is centralized in the `hardhat.config.js` (or `.ts`) file.

**a) Enable the Plugin:**

Require the plugin at the top of your configuration file:

```javascript
require("@nomicfoundation/hardhat-verify");
```

**b) Network Definitions (`networks`):**

Define your target deployment networks, ensuring they include the `url` (RPC address), `chainId`, and `accounts` used for deployment.

```javascript
// .env file example: PRIVATE_KEY=0x...
// Use dotenv to load environment variables like private keys
require("dotenv").config();

// ... other require statements ...

networks: {
  JuChainTestnet: {
    url: "https://testnet-rpc.juchain.org",
    chainId: 202599,
    // Strongly recommended to load private keys from environment variables, avoid hardcoding
    accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
  },
  JuChain: {
    url: "https://rpc.juchain.org",
    chainId: 210000,
    accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
  },
  // ... other networks like mainnet, sepolia, etc.
},
```

* **Key Point:** The private key in the `accounts` array must belong to the account that performed the contract deployment.

**c) Block Explorer Verification Configuration (`etherscan`):**

This section is for connecting to the block explorer's verification service. **Even if targeting Juscan, the configuration must be placed under the `etherscan` field.**

```javascript
etherscan: {
  // Currently, Juscan does not require an API Key, so this value is sufficient.
  // If Juscan's policy changes in the future and requires an API Key, please consult their official documentation.
  apiKey: {
    JuChainTestnet: "JUSCAN_NO_API_KEY_REQUIRED",
    JuChain: "JUSCAN_NO_API_KEY_REQUIRED",
  },
  // Custom configuration for blockchains compatible with the Etherscan API
  customChains: [
    {
      network: "JuChainTestnet",
      chainId: 202599,
      urls: {
        apiURL: "https://testnet.juscan.io/api",
        browserURL: "https://testnet.juscan.io",
      },
    },
    {
      network: "JuChain", // Ensure this network name matches the key defined in 'networks'
      chainId: 210000,
      urls: {
        // Please refer to the official Juscan documentation to confirm the latest mainnet API URL
        apiURL: "https://juscan.io/api",
        browserURL: "https://juscan.io",
      },
    },
    // ... other custom chains
  ],
},
```

* **`customChains`**: Configure networks not natively supported by Etherscan. The `network` field must strictly match the key name in `networks`. `apiURL` is the target address for verification requests.

**d) Sourcify (Optional):**

Configure Sourcify verification support as needed.

```javascript
sourcify: {
  enabled: false, // Set to true if you need to use it
},
```

***

### 2. Contract Deployment (Prerequisite)

Verification requires the contract to be successfully deployed to the target network. **The following is an example scenario, please replace with your actual contract and parameters:**

* **Example `ContractA` (No dependencies):**
  * Deployment Address (Example): `0x60322a8476918646E297E0267F3444872cF5bA09`
  * Constructor Arguments (Example): `100`, `"0x...00a"` (assuming an address or bytes32)
* **Example `ContractB` (Depends on `ContractA` address):**
  * Deployment Address (Example): `0x2C8B08E38fd8BaaED93F91929659b9aF2B65345E`
  * Constructor Arguments (Example): `0x60322a8476918646E297E0267F3444872cF5bA09` (ContractA address), `50`, `"0x...00b"`

You should accurately recorded the actual deployment address and the constructor arguments used via your Hardhat deployment script.

***

### 3. Executing the Verification Command

Use the `npx hardhat verify` command to initiate verification.

#### 3.1 Verifying Example `ContractA`

```bash
# Command structure: npx hardhat verify --network <network_name> <contract_address> [constructor_arguments...]
# Ensure you use your actual deployment address and constructor arguments
npx hardhat verify --network JuChainTestnet 0x60322a8476918646E297E0267F3444872cF5bA09 100 "0x000000000000000000000000000000000000000a"
```

* Arguments must be provided in the exact order and type as during deployment.

#### 3.2 Verifying Example `ContractB`

```bash
# Ensure you use your actual deployment address and constructor arguments
npx hardhat verify --network JuChainTestnet 0x2C8B08E38fd8BaaED93F91929659b9aF2B65345E "0x60322a8476918646E297E0267F3444872cF5bA09" 50 "0x000000000000000000000000000000000000000b"
```

* Address arguments are passed as strings.

#### 3.3 Checking the Result

Upon successful verification, the command line will print a link to the block explorer. Visit this link to confirm that a verification mark (like a green checkmark) appears next to the contract code, and the matching Solidity source code is displayed under the "Contract" / "Code" tab.

***

### 4. Common Issues and Troubleshooting

Verification failures are often due to the following reasons. Please check them one by one:

#### 4.1 Constructor Arguments Mismatch

* **Check**: This is the most common issue. Carefully compare the arguments provided in the `verify` command with those actually used in your deployment script or transaction record. Ensure the type, order, and values match exactly.
* **Complex Arguments**: For complex arguments (like structs or arrays), consider using the `--constructor-args` option to specify a JS/TS file that exports the argument list. E.g., `arguments.js`: `module.exports = [arg1, arg2, ...];`, then run `npx hardhat verify --constructor-args arguments.js ...`.

#### 4.2 Network Configuration Error

* Check if the network name specified with `--network` (e.g., `JuChainTestnet`) exists in both `networks` and `etherscan.customChains` in `hardhat.config.js`, and that the `chainId` is correct.
* Confirm that the `apiURL` in `customChains` is the valid verification endpoint provided by the block explorer.

#### 4.3 Incorrect Contract Address or Name

* Verify that the provided contract address is correct and was deployed on the specified `--network` (`JuChainTestnet`).
* If a Solidity file contains multiple contracts, Hardhat usually infers the correct one. If inference fails or you need to specify explicitly, use the `--contract` flag: `npx hardhat verify --contract contracts/MyFile.sol:MySpecificContract ...`.

#### 4.4 Compiler or Optimizer Settings Mismatch

* Confirm that the `solidity` version and optimizer settings (`optimizer: { enabled: true/false, runs: ... }`) in `hardhat.config.js` are exactly the same as those used during deployment. Hardhat uses the settings from the config file by default during verification.

#### 4.5 Proxy Contracts

* **Verify Separately**: Usually, you need to verify the implementation contract (Logic Contract) first, and then handle the proxy contract separately.
* **Link Proxy**: For transparent or UUPS proxies, after verifying the implementation contract, you might need to manually link the proxy address to the implementation address on the block explorer's interface. Check the specific procedures for your proxy pattern and the block explorer. The `@openzeppelin/hardhat-upgrades` plugin provides helper functions for verifying OpenZeppelin proxies.

#### 4.6 Verification Service Delay or Temporary Outage

* Sometimes, even if the submission is successful, the block explorer needs a few minutes to process and update the status. Refresh the page later.
* Check the block explorer's status page or community forums to confirm if the verification service is operating normally.

***

### 5. Manual Verification (Optional)

Besides using the Hardhat plugin, most block explorers (including Juscan) also support manual source code verification directly through their website interface. If you encounter difficulties with Hardhat verification or prefer a graphical approach, you can try this method.

The general steps are as follows:

1. Visit your deployed contract's address page on Juscan (or another explorer).
2. Look for an option or button like "Verify & Publish Contract Source Code".
3. Select the correct Solidity **compiler version** and **optimizer settings** (must exactly match those used during deployment).
4. Paste your contract **source code** into the provided text box. If your contract imports other files, you might need to:
   * "Flatten" all code into a single file before pasting.
   * Alternatively, if the explorer supports it, upload the main contract file and all dependent library/interface files separately.
5. If your contract has **constructor arguments**, enter them in the format required by the explorer.
6. Submit the verification request and wait for the explorer to process it.

Manual verification can be helpful for understanding the details of the process, but it's less efficient than automated Hardhat verification for iterative development.

***

Following these steps and paying attention to detail can significantly increase the success rate of contract verification. Contract verification is an important practice for ensuring the transparency and trustworthiness of the smart contract ecosystem.
