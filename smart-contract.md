# Smart Contract

{% hint style="warning" %}
JuChain currently supports Solidity versions <= 0.8.8 for compilation. Support for later versions will be gradually introduced based on official announcements.
{% endhint %}

JuChain is fully compatible with the Ethereum Virtual Machine (EVM), enabling developers to seamlessly deploy and execute smart contracts written in Solidity. This document provides an introduction to the essentials of smart contract development on JuChain.

***

#### Contract Development

**Development Environment**

To develop smart contracts on JuChain, you’ll need:

1. **Code Editor**: Tools like VSCode or Remix.
2. **Solidity Compiler**: For compiling your Solidity code.
3. **Web3 Development Framework**: Options include Hardhat or Truffle.
4. **JuChain RPC Endpoint**: To connect to the JuChain network (e.g., `https://testnet-rpc.juchain.org` for testnet).

***

**Contract Deployment**

You can deploy smart contracts to JuChain using:

1. **Remix IDE**: Connect to the JuChain network via the “Injected Web3” provider (e.g., MetaMask configured with JuChain RPC).
2. **Hardhat or Truffle**: Use deployment scripts tailored to JuChain’s network settings.
3. **Web3 Library**: Interact directly with the network using libraries like Web3.js or ethers.js.

**Example: Deploying with Hardhat**

```javascript
const hre = require("hardhat");

async function main() {
  const [deployer] = await hre.ethers.getSigners();
  console.log("Deploying from:", deployer.address);

  const MyContract = await hre.ethers.getContractFactory("MyContract");
  const contract = await MyContract.deploy();
  await contract.deployed();

  console.log("Contract deployed to:", contract.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Configure Hardhat with JuChain’s network:

```javascript
// hardhat.config.js
module.exports = {
  solidity: "0.8.8",
  networks: {
    juchain_testnet: {
      url: "https://testnet-rpc.juchain.org",
      chainId: 202599,
      accounts: ["your_private_key"]
    }
  }
};
```

***

#### Best Practices

When developing smart contracts for JuChain:

1. **Thorough Testing**: Test contracts extensively using frameworks like Hardhat or Truffle to catch bugs early.
2. **Security Practices**: Follow guidelines like OpenZeppelin’s security recommendations to prevent vulnerabilities (e.g., reentrancy, overflow).
3. **Gas Optimization**: Minimize gas costs by optimizing loops, storage usage, and function calls.
4. **Stable Solidity Version**: Use the latest stable Solidity version supported by JuChain (currently <= 0.8.8).
5. **Access Control**: Implement role-based access (e.g., OpenZeppelin’s `Ownable` or `AccessControl`) to restrict sensitive functions.

**Example: Simple Contract with Access Control**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.8;

import "@openzeppelin/contracts/access/Ownable.sol";

contract MyContract is Ownable {
    uint256 public value;

    function setValue(uint256 _value) external onlyOwner {
        value = _value;
    }
}
```

***

#### Contract Verification

After deployment, verify your contract’s source code on JuChain’s block explorer (e.g., `https://explorer-testnet.juchain.org`) for transparency and security. Verification ensures users can audit the deployed bytecode against the source.

**Steps for Verification:**

1. **Flatten Contract**: If using multiple files or imports, flatten your code into a single file (e.g., using `hardhat flatten`).
2. **Submit on Explorer**: Provide the contract address, flattened source code, compiler version (e.g., 0.8.8), and constructor arguments (if any).
3. **Confirm**: Once verified, the explorer will display the contract’s source and ABI publicly.

**Example: Hardhat Verification**

```javascript
// Run this after deployment
hre.run("verify:verify", {
  address: "deployed_contract_address",
  constructorArguments: [],
});
```

Requires the Hardhat Etherscan plugin and an API key (if applicable).
