# Tutorials and Examples: Building a Simple DeFi Lending dApp



This tutorial will walk you through developing a simple decentralized finance (DeFi) lending dApp on **JuChain**. Users can deposit JU (JuChain’s native token) as collateral, borrow a portion of JU based on that collateral, and retrieve their collateral after repaying the loan. We’ll use Solidity to write the smart contract, leverage JuChain’s EVM compatibility for deployment, and create a frontend for user interaction.

This example highlights JuChain’s **near-instant transaction confirmations** and **ultra-low costs**, making it ideal for DeFi use cases.

***

#### Tutorial: Simple DeFi Lending dApp

**Prerequisites**

**Development Environment**

* **Node.js**: v16 or higher
* **Truffle Suite**: Install via `npm install -g truffle`
* **MetaMask**: Browser extension, configured for JuChain Testnet
* **Knowledge**: Familiarity with Solidity and Web3.js

**JuChain Testnet**

* **Testnet RPC**: `https://testnet-rpc.juchain.org`
* **Network ID**: `202599` (assumed)
* **Test JU**: Obtainable via the JuChain Testnet faucet

***

**Step 1: Set Up the Project**

1.  **Create and Initialize the Project**:

    ```bash
    mkdir JuChainLending
    cd JuChainLending
    truffle init
    ```
2.  **Install Dependencies**:

    ```bash
    npm install @openzeppelin/contracts web3
    ```

***

**Step 2: Write the Lending Smart Contract**

Create `LendingPool.sol` in the `contracts/` folder:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract LendingPool is ReentrancyGuard {
    mapping(address => uint256) public collateral; // User's deposited JU collateral
    mapping(address => uint256) public borrowed;   // User's borrowed JU amount
    uint256 public constant LTV = 50;              // Loan-to-Value ratio, 50%
    uint256 public constant INTEREST_RATE = 5;     // 5% interest rate (simplified per repayment)

    event Deposited(address indexed user, uint256 amount);
    event Borrowed(address indexed user, uint256 amount);
    event Repaid(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    constructor() {
        // No parameters needed; JU is the native token
    }

    // Deposit JU as collateral
    function depositCollateral() external payable nonReentrant {
        require(msg.value > 0, "Amount must be greater than 0");
        collateral[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    // Borrow JU based on 50% LTV of collateral
    function borrow(uint256 amount) external nonReentrant {
        require(amount > 0, "Amount must be greater than 0");
        uint256 maxBorrow = (collateral[msg.sender] * LTV) / 100;
        require(borrowed[msg.sender] + amount <= maxBorrow, "Exceeds max borrowable amount");
        borrowed[msg.sender] += amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Borrowed(msg.sender, amount);
    }

    // Repay loan (including interest)
    function repay(uint256 amount) external payable nonReentrant {
        require(amount > 0, "Amount must be greater than 0");
        require(borrowed[msg.sender] >= amount, "Repay exceeds borrowed amount");
        uint256 interest = (amount * INTEREST_RATE) / 100;
        uint256 totalRepay = amount + interest;
        require(msg.value >= totalRepay, "Insufficient payment");
        borrowed[msg.sender] -= amount;
        emit Repaid(msg.sender, amount);

        // Refund excess JU
        if (msg.value > totalRepay) {
            (bool success, ) = msg.sender.call{value: msg.value - totalRepay}("");
            require(success, "Refund failed");
        }
    }

    // Withdraw collateral (after repaying all loans)
    function withdrawCollateral(uint256 amount) external nonReentrant {
        require(amount > 0, "Amount must be greater than 0");
        require(collateral[msg.sender] >= amount, "Insufficient collateral");
        require(borrowed[msg.sender] == 0, "Must repay all loans first");
        collateral[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }

    // View user status
    function getUserStatus(address user) external view returns (uint256, uint256, uint256) {
        uint256 maxBorrow = (collateral[user] * LTV) / 100;
        return (collateral[user], borrowed[user], maxBorrow);
    }
}
```

**Functionality Explained**:

* **`depositCollateral()`**: Users send JU (native token) as collateral.
* **`borrow(uint256 amount)`**: Borrow up to 50% of collateral’s value in JU.
* **`repay(uint256 amount)`**: Repay the loan with 5% interest; excess JU is refunded.
* **`withdrawCollateral(uint256 amount)`**: Retrieve collateral after full repayment.
* **`getUserStatus(address user)`**: Returns collateral, borrowed amount, and max borrowable JU.
* **Security**: Uses `ReentrancyGuard` to prevent reentrancy attacks.

**Note**: JU is JuChain’s native token, handled via `msg.value` and `call`, not an ERC20 token.

***

**Step 3: Configure Truffle**

Edit `truffle-config.js` to connect to the JuChain Testnet:

```javascript
const HDWalletProvider = require("@truffle/hdwallet-provider");
const mnemonic = "your MetaMask mnemonic"; // Replace with your mnemonic

module.exports = {
  networks: {
    juchain_testnet: {
      provider: () => new HDWalletProvider(mnemonic, "https://testnet-rpc.juchain.org"),
      network_id: 202599,
      gas: 5500000,
      gasPrice: 1000000000, // 1 gwei
    },
  },
  compilers: {
    solc: {
      version: "0.8.0",
    },
  },
};
```

***

**Step 4: Deploy the Contract**

1.  **Create a Deployment Script**:\
    In `migrations/`, add `2_deploy_lending.js`:

    ```javascript
    const LendingPool = artifacts.require("LendingPool");

    module.exports = function (deployer) {
      deployer.deploy(LendingPool);
    };
    ```
2.  **Deploy to JuChain Testnet**:

    ```bash
    truffle migrate --network juchain_testnet
    ```

    * Note the deployed `LendingPool` contract address (e.g., `0x1234...5678`).

***

**Step 5: Test the Contract**

Validate functionality using the Truffle console:

1.  **Enter the Console**:

    ```bash
    truffle console --network juchain_testnet
    ```
2.  **Test Commands**:

    ```javascript
    let lending = await LendingPool.deployed();
    let accounts = await web3.eth.getAccounts();

    // Deposit 100 JU as collateral
    await lending.depositCollateral({ from: accounts[0], value: web3.utils.toWei("100") });
    console.log((await lending.collateral(accounts[0])).toString()); // "100000000000000000000"

    // Borrow 50 JU
    await lending.borrow(web3.utils.toWei("50"), { from: accounts[0] });
    console.log((await lending.borrowed(accounts[0])).toString()); // "50000000000000000000"

    // Check user status
    let status = await lending.getUserStatus(accounts[0]);
    console.log(status.map(x => web3.utils.fromWei(x))); // [100, 50, 50]

    // Repay 50 JU (with 5% interest, total 52.5 JU)
    await lending.repay(web3.utils.toWei("50"), { from: accounts[0], value: web3.utils.toWei("52.5") });
    console.log((await lending.borrowed(accounts[0])).toString()); // "0"

    // Withdraw 100 JU
    await lending.withdrawCollateral(web3.utils.toWei("100"), { from: accounts[0] });
    console.log((await lending.collateral(accounts[0])).toString()); // "0"
    ```

***

**Step 6: Create the Frontend Interface**

Create `index.html` in the project root:

```html
<!DOCTYPE html>
<html>
<head>
  <title>JuChain Lending dApp</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
    button { padding: 10px 20px; margin: 5px; }
    input { padding: 5px; margin: 5px; }
  </style>
</head>
<body>
  <h1>JuChain Lending Pool</h1>
  <p>Collateral: <span id="collateral">0</span> JU</p>
  <p>Borrowed: <span id="borrowed">0</span> JU</p>
  <p>Max Borrowable: <span id="maxBorrow">0</span> JU</p>
  <input id="amount" type="number" placeholder="Amount in JU" />
  <br />
  <button onclick="deposit()">Deposit Collateral</button>
  <button onclick="borrow()">Borrow JU</button>
  <button onclick="repay()">Repay Loan</button>
  <button onclick="withdraw()">Withdraw Collateral</button>

  <script src="https://cdn.jsdelivr.net/npm/web3@1.5.3/dist/web3.min.js"></script>
  <script>
    const contractAddress = "your LendingPool contract address"; // Replace with actual address
    const abi = [ /* Replace with full ABI from build/contracts/LendingPool.json */ ];
    let web3, lending;

    async function init() {
      if (window.ethereum) {
        web3 = new Web3(window.ethereum);
        await window.ethereum.enable();
        lending = new web3.eth.Contract(abi, contractAddress);
        updateStatus();
      } else {
        alert("Please install MetaMask!");
      }
    }

    async function updateStatus() {
      const accounts = await web3.eth.getAccounts();
      const status = await lending.methods.getUserStatus(accounts[0]).call();
      document.getElementById("collateral").innerText = web3.utils.fromWei(status[0]);
      document.getElementById("borrowed").innerText = web3.utils.fromWei(status[1]);
      document.getElementById("maxBorrow").innerText = web3.utils.fromWei(status[2]);
    }

    async function deposit() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      await lending.methods.depositCollateral().send({ from: accounts[0], value: amount });
      updateStatus();
    }

    async function borrow() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      await lending.methods.borrow(amount).send({ from: accounts[0] });
      updateStatus();
    }

    async function repay() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      const interest = (BigInt(amount) * 5n) / 100n;
      const total = (BigInt(amount) + interest).toString();
      await lending.methods.repay(amount).send({ from: accounts[0], value: total });
      updateStatus();
    }

    async function withdraw() {
      const accounts = await web3.eth.getAccounts();
      const amount = web3.utils.toWei(document.getElementById("amount").value);
      await lending.methods.withdrawCollateral(amount).send({ from: accounts[0] });
      updateStatus();
    }

    init();
  </script>
</body>
</html>
```

**Configuration Notes**:

* **ABI**: Copy the full ABI from `build/contracts/LendingPool.json` and replace the `abi` variable.
* **Contract Address**: Update `contractAddress` with the address from Step 4.
* **Run**: Serve `index.html` with a local server (e.g., `npx http-server`) and connect via MetaMask on the JuChain Testnet.

***

**Step 7: Run and Test**

1. Switch MetaMask to the JuChain Testnet and ensure sufficient test JU is available.
2. Open `index.html` in a browser and test:
   * **Deposit Collateral**: Enter 100 JU and click "Deposit Collateral".
   * **Borrow JU**: Enter 50 JU and click "Borrow JU".
   * **Repay Loan**: Enter 50 JU and click "Repay Loan" (includes 5% interest, total 52.5 JU).
   * **Withdraw Collateral**: Enter 100 JU and click "Withdraw Collateral".
3. JuChain’s near-instant confirmations ensure transactions complete in seconds, with fees typically below 0.001 JU.

***

**Outcome**

You’ve built and deployed a simple DeFi lending dApp:

* Users can deposit JU as collateral.
* Borrow up to 50% of their collateral’s value in JU.
* Repay loans with 5% interest to reclaim collateral.
* The frontend displays collateral, borrowed amounts, and max borrowable JU in real-time.

This dApp demonstrates JuChain’s high-performance infrastructure, perfect for scaling into more complex DeFi applications (e.g., multi-asset support or dynamic rates).

***

**Notes**

1. **Security**:
   * For production, add liquidation mechanisms (to handle undercollateralization) and conduct a code audit.
   * The current contract doesn’t address edge cases (e.g., insufficient JU balance).
2. **Testnet**:
   * Ensure your account has enough test JU for interactions.
3. **Scalability**:
   * Enhance with time-based interest calculations or additional asset support as needed.

***
