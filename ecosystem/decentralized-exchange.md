# Decentralized Exchange

JuChain provides a decentralized exchange (DEX) based on the Uniswap V2 model, enabling token swaps and liquidity provision. This documentation guides developers on interacting with the core DEX contracts using Javascript (Web3.js/ethers.js) or Solidity.

**Core Components:**

* **Router (`JUV2Router02`)**: The primary contract for user interactions (swaps, adding/removing liquidity).
* **Factory (`JUV2Factory`)**: Creates and manages trading pairs.
* **Pair (`JUV2Pair`)**: Represents a specific token trading pair (e.g., WJU-USDT) and holds liquidity.
* **WJU (Wrapped JU)**: The wrapped version of the native JU token, conforming to ERC20 standards allowing interaction with the DEX.
* **USDT**: An example ERC20 token on the network.

### 1. Prerequisites & Network Setup

* **Network**: JuChain Testnet (or mainnet when applicable)
  * **RPC URL**: `https://testnet-rpc.juchain.org` (Verify with official sources)
  * **Chain ID**: `202599`
* **Tools**: Node.js environment with Web3.js or ethers.js library, or a Solidity development environment (Remix, Hardhat, Truffle).
* **Account**: An Externally Owned Account (EOA) with native JU (for gas) and relevant tokens (WJU, USDT).

### 2. Core Contract Addresses & Details

| Contract           | Address                                                              | Solidity Version | Notes                                                                        |
| ------------------ | -------------------------------------------------------------------- | ---------------- | ---------------------------------------------------------------------------- |
| **WJU**            | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`                         | `0.4.26`         | Wrapped JU. Use `deposit()` (payable) to wrap JU, `withdraw()` to unwrap.    |
| **USDT**           | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`                         | `0.8.8`          | Standard ERC20 token.                                                        |
| **Factory**        | `0x66682281BdfeC17fCBcae0480C77edFb0c489339`                         | `0.5.16`         | Deploys and tracks `JUV2Pair` contracts.                                     |
| **Router**         | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0`                         | `0.6.6`          | **Main interaction point for Swaps & Liquidity.** Assumes WJU is its `WETH`. |
| **Pair Init Hash** | `0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b` | N/A              | Used for calculating pair addresses (See `JUV2Library.pairFor`).             |

**Important:** The Router (`JUV2Router02`) contract uses a `WETH` address internally. For JuChain, this `WETH` address **must correspond to the WJU address** (`0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`) for the `*ETH` functions (e.g., `addLiquidityETH`, `swapExactETHForTokens`) to work correctly with Wrapped JU. We will assume this configuration is correct.

### 3. Interacting with WJU (Wrapping/Unwrapping)

Since the DEX works with ERC20 tokens, you need to wrap your native JU into WJU first.

*   **Wrap JU**: Send native JU to the WJU contract's `deposit()` function (or its fallback function).

    ```javascript
    // Web3.js Example: Wrap 1 native JU
    const { Web3 } = require('web3');
    const web3 = new Web3('https://testnet-rpc.juchain.org');
    const wjuAddress = '0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3';
    const wjuABI = [/* WJU ABI from user */]; // Use the provided WJU ABI
    const wjuContract = new web3.eth.Contract(wjuABI, wjuAddress);
    const account = '0xYourAddress';
    const privateKey = '0xYourPrivateKey'; // Handle securely!
    const amountToWrap = web3.utils.toWei('1', 'ether');

    async function wrapJU() {
      const tx = {
        to: wjuAddress,
        value: amountToWrap,
        gas: 300000, // Estimate gas appropriately
        from: account
      };
      const signedTx = await web3.eth.accounts.signTransaction(tx, privateKey);
      const receipt = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
      console.log('JU Wrapped:', receipt.transactionHash);
      // OR call deposit() explicitly:
      // const depositTx = wjuContract.methods.deposit();
      // const gas = await depositTx.estimateGas({ from: account, value: amountToWrap });
      // const signedDepositTx = await web3.eth.accounts.signTransaction({ ...tx, data: depositTx.encodeABI(), gas }, privateKey);
      // const depositReceipt = await web3.eth.sendSignedTransaction(signedDepositTx.rawTransaction);
      // console.log('JU Wrapped via deposit():', depositReceipt.transactionHash);
    }
    wrapJU();
    ```
*   **Unwrap WJU**: Call the `withdraw()` function on the WJU contract, specifying the amount of WJU to convert back to native JU.

    ```javascript
    // Web3.js Example: Unwrap 1 WJU
    async function unwrapWJU() {
      const amountToUnwrap = web3.utils.toWei('1', 'ether');
      const tx = wjuContract.methods.withdraw(amountToUnwrap);
      const gas = await tx.estimateGas({ from: account });
      const data = tx.encodeABI();

      const signedTx = await web3.eth.accounts.signTransaction({
          to: wjuAddress,
          data,
          gas,
          from: account
      }, privateKey);
      const receipt = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
      console.log('WJU Unwrapped:', receipt.transactionHash);
    }
    unwrapWJU();
    ```

### 4. Swap Functionality (Using the Router)

Swaps allow exchanging one token for another through existing liquidity pools.

**Key Router Functions for Swaps:**

* `swapExactTokensForTokens`: Swap an exact amount of input token for a minimum amount of output token.
* `swapTokensForExactTokens`: Swap a maximum amount of input token for an exact amount of output token.
* `swapExactETHForTokens`: Swap an exact amount of **WJU** (acting as WETH) for a minimum amount of output token.
* `swapTokensForExactETH`: Swap a maximum amount of input token for an exact amount of **WJU** (acting as WETH).
* `swapExactTokensForETH`: Swap an exact amount of input token for a minimum amount of **WJU** (acting as WETH).
* `swapETHForExactTokens`: Swap a maximum amount of **WJU** (acting as WETH, sent with the transaction) for an exact amount of output token.
* `*SupportingFeeOnTransferTokens`: Variants designed for tokens that charge a fee on transfer.

**Steps for Swapping:**

1. **Approve Router**: Before swapping ERC20 tokens (like USDT or even WJU when it's the _input_ token in `swapExactTokensFor*`), you must approve the Router contract to spend the token on your behalf. Call the token's `approve(routerAddress, amount)` function.
2. **Call Swap Function**: Call the desired swap function on the Router contract.

**Example 1: Swap Exact WJU for USDT (using `swapExactETHForTokens`)**

```javascript
// --- Web3.js Example ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

const routerAddress = '0x6A647E09193a130b0CccBF26A1CF442491bDeCc0';
const wjuAddress = '0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3'; // Acts as WETH
const usdtAddress = '0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02';
const routerABI = [/* Relevant Router ABI Snippets - see section 7 */];

const routerContract = new web3.eth.Contract(routerABI, routerAddress);
const account = '0xYourAddress';
const privateKey = '0xYourPrivateKey'; // Handle securely!

async function swapWJUForUSDT(wjuAmountIn, usdtAmountOutMin) {
  const path = [wjuAddress, usdtAddress]; // Path: WJU -> USDT
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10; // 10 minutes from now

  // No ERC20 approval needed for WJU when using swapExactETHForTokens
  // The WJU is sent with the transaction value

  const tx = routerContract.methods.swapExactETHForTokens(
    usdtAmountOutMin, // Minimum USDT you want to receive (slippage protection)
    path,
    account,          // Address to receive the USDT
    deadline
  );

  const gas = await tx.estimateGas({ from: account, value: wjuAmountIn });
  const data = tx.encodeABI();

  const signedTx = await web3.eth.accounts.signTransaction({
      to: routerAddress,
      data,
      gas,
      value: wjuAmountIn, // Send WJU (as ETH value) with the transaction
      from: account
  }, privateKey);

  const receipt = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
  console.log(`Swap WJU -> USDT successful: ${receipt.transactionHash}`);
}

// Example: Swap 1 WJU for at least 0.9 USDT
const oneWJU = web3.utils.toWei('1', 'ether');
const pointNineUSDT = web3.utils.toWei('0.9', 'ether'); // USDT might have different decimals! Check USDT contract. Let's assume 18 for example.
// const pointNineUSDT = '900000000000000000'; // If USDT has 18 decimals
// CHECK USDT DECIMALS: The provided USDT contract uses OZ ERC20, which defaults to 18 decimals.

swapWJUForUSDT(oneWJU, pointNineUSDT);
```

**Example 2: Swap Exact USDT for WJU (using `swapExactTokensForETH`)**

```javascript
// --- Web3.js Example ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// Addresses and ABIs as above...
const usdtABI = [/* USDT ABI from user */];
const usdtContract = new web3.eth.Contract(usdtABI, usdtAddress);

async function swapUSDTForWJU(usdtAmountIn, wjuAmountOutMin) {
  const path = [usdtAddress, wjuAddress]; // Path: USDT -> WJU
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10; // 10 minutes

  // 1. Approve Router to spend USDT
  console.log(`Approving Router to spend ${web3.utils.fromWei(usdtAmountIn)} USDT...`);
  const approveTx = usdtContract.methods.approve(routerAddress, usdtAmountIn);
  const approveGas = await approveTx.estimateGas({ from: account });
  const approveData = approveTx.encodeABI();
  const signedApproveTx = await web3.eth.accounts.signTransaction({
      to: usdtAddress, data: approveData, gas: approveGas, from: account
  }, privateKey);
  const approveReceipt = await web3.eth.sendSignedTransaction(signedApproveTx.rawTransaction);
  console.log(`Approval successful: ${approveReceipt.transactionHash}`);

  // 2. Call swap function
  console.log(`Swapping ${web3.utils.fromWei(usdtAmountIn)} USDT for WJU...`);
  const swapTx = routerContract.methods.swapExactTokensForETH(
    usdtAmountIn,     // Exact USDT amount to send
    wjuAmountOutMin,  // Minimum WJU you want to receive
    path,
    account,          // Address to receive WJU
    deadline
  );

  const swapGas = await swapTx.estimateGas({ from: account });
  const swapData = swapTx.encodeABI();

  const signedSwapTx = await web3.eth.accounts.signTransaction({
      to: routerAddress, data: swapData, gas: swapGas, from: account
  }, privateKey);

  const swapReceipt = await web3.eth.sendSignedTransaction(signedSwapTx.rawTransaction);
  console.log(`Swap USDT -> WJU successful: ${swapReceipt.transactionHash}`);
}

// Example: Swap 1 USDT for at least 0.9 WJU (assuming 18 decimals for both)
const oneUSDT = web3.utils.toWei('1', 'ether');
const pointNineWJU = web3.utils.toWei('0.9', 'ether');

swapUSDTForWJU(oneUSDT, pointNineWJU);
```

**Notes on Swaps:**

* **Slippage:** Set `amountOutMin` (for exact input swaps) or `amountInMax` (for exact output swaps) carefully to protect against price changes during transaction execution.
* **Path:** The `path` array defines the route for the swap. For direct swaps (e.g., WJU-USDT), it's `[tokenInAddress, tokenOutAddress]`. For multi-hop swaps, include intermediate tokens.
* **Deadline:** Transactions pending after the deadline will fail, preventing execution at an unfavorable old price.
* **Gas:** Gas costs vary. Use `estimateGas` before sending.
* **Liquidity:** Swaps require sufficient liquidity in the relevant pair contract. Check the pair on the explorer or via `getReserves`.

### 5. Liquidity Provision (Using the Router)

Users can provide liquidity to a pair (e.g., WJU-USDT) and earn fees from swaps occurring in that pair. They receive LP (Liquidity Provider) tokens representing their share.

**Key Router Functions for Liquidity:**

* `addLiquidity`: Add liquidity for a pair of two ERC20 tokens.
* `addLiquidityETH`: Add liquidity for a pair involving WJU (acting as WETH) and another ERC20 token. WJU is sent with the transaction value.
* `removeLiquidity`: Remove liquidity for a pair of two ERC20 tokens by burning LP tokens.
* `removeLiquidityETH`: Remove liquidity for a WJU (acting as WETH) / ERC20 pair by burning LP tokens. Receives WJU and the ERC20 token.
* `*WithPermit`: Variants allowing liquidity removal using an off-chain signature (EIP-712) instead of a prior `approve` transaction for the LP token.

**Steps for Adding Liquidity:**

1. **Approve Router**: Approve the Router to spend _both_ ERC20 tokens you are providing (unless one is WJU being provided via `addLiquidityETH`).
2. **Call Add Function**: Call `addLiquidity` or `addLiquidityETH`. You specify _desired_ amounts, but the actual amounts taken will maintain the pair's current price ratio. Set minimum amounts (`amountAMin`, `amountBMin`, `amountETHMin`) to control slippage.

**Example: Add Liquidity for WJU-USDT (using `addLiquidityETH`)**

```javascript
// --- Web3.js Example ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// Addresses, ABIs, account, privateKey as above...

async function addLiquidityWJUUSDT(wjuAmountDesired, usdtAmountDesired, wjuAmountMin, usdtAmountMin) {
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10;

  // 1. Approve Router to spend USDT
  console.log(`Approving Router to spend ${web3.utils.fromWei(usdtAmountDesired)} USDT...`);
  const approveTx = usdtContract.methods.approve(routerAddress, usdtAmountDesired); // Approve desired amount or max_uint
  const approveGas = await approveTx.estimateGas({ from: account });
  const approveData = approveTx.encodeABI();
  const signedApproveTx = await web3.eth.accounts.signTransaction({
      to: usdtAddress, data: approveData, gas: approveGas, from: account
  }, privateKey);
  const approveReceipt = await web3.eth.sendSignedTransaction(signedApproveTx.rawTransaction);
  console.log(`Approval successful: ${approveReceipt.transactionHash}`);

  // 2. Add Liquidity
  console.log(`Adding liquidity: ${web3.utils.fromWei(wjuAmountDesired)} WJU and ${web3.utils.fromWei(usdtAmountDesired)} USDT...`);
  const addLiqTx = routerContract.methods.addLiquidityETH(
    usdtAddress,        // The ERC20 token address
    usdtAmountDesired,  // Desired amount of USDT to add
    usdtAmountMin,      // Minimum amount of USDT to add (slippage)
    wjuAmountMin,       // Minimum amount of WJU to add (slippage)
    account,            // Address to receive LP tokens
    deadline
  );

  const addLiqGas = await addLiqTx.estimateGas({ from: account, value: wjuAmountDesired });
  const addLiqData = addLiqTx.encodeABI();

  const signedAddLiqTx = await web3.eth.accounts.signTransaction({
      to: routerAddress,
      data: addLiqData,
      gas: addLiqGas,
      value: wjuAmountDesired, // Send the desired WJU amount (router calculates optimal)
      from: account
  }, privateKey);

  const addLiqReceipt = await web3.eth.sendSignedTransaction(signedAddLiqTx.rawTransaction);
  console.log(`Add Liquidity WJU-USDT successful: ${addLiqReceipt.transactionHash}`);
  // The receipt logs will contain info about actual amounts added and LP tokens minted.
}

// Example: Add ~1 WJU and ~100 USDT (assuming this is near the current ratio)
// Set minimums slightly lower for slippage tolerance, e.g., 99%
const wjuToAdd = web3.utils.toWei('1', 'ether');
const usdtToAdd = web3.utils.toWei('100', 'ether'); // Assuming 18 decimals
const wjuMin = web3.utils.toWei('0.99', 'ether'); // 99% of desired WJU
const usdtMin = web3.utils.toWei('99', 'ether');   // 99% of desired USDT

addLiquidityWJUUSDT(wjuToAdd, usdtToAdd, wjuMin, usdtMin);
```

**Steps for Removing Liquidity:**

1. **Get Pair Address**: Determine the address of the liquidity pair contract (e.g., WJU-USDT pair). You can use the Factory's `getPair(tokenA, tokenB)` function or calculate it using `JUV2Library.pairFor` logic (see Solidity example below or use a library).
2. **Approve Router**: Approve the Router contract to spend your LP tokens. Call the `approve(routerAddress, liquidityAmount)` function _on the Pair contract_.
3. **Call Remove Function**: Call `removeLiquidity` or `removeLiquidityETH`, specifying the amount of LP tokens (`liquidity`) to burn and the minimum amounts of underlying tokens you expect back (`amountAMin`, `amountBMin`, `amountETHMin`).

**Example: Remove Liquidity from WJU-USDT (using `removeLiquidityETH`)**

```javascript
// --- Web3.js Example ---
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

// Addresses, ABIs, account, privateKey as above...
const factoryABI = [/* Factory ABI from user */];
const factoryAddress = '0x66682281BdfeC17fCBcae0480C77edFb0c489339';
const pairABI = [/* JUV2Pair ABI - can often use standard ERC20 ABI + burn/mint/sync etc. */];

const factoryContract = new web3.eth.Contract(factoryABI, factoryAddress);

async function removeLiquidityWJUUSDT(liquidityAmount, usdtAmountMin, wjuAmountMin) {
  const deadline = Math.floor(Date.now() / 1000) + 60 * 10;

  // 1. Get Pair Address
  const pairAddress = await factoryContract.methods.getPair(wjuAddress, usdtAddress).call();
  if (pairAddress === '0x0000000000000000000000000000000000000000') {
    console.error("Pair does not exist!");
    return;
  }
  console.log(`Pair Address: ${pairAddress}`);
  const pairContract = new web3.eth.Contract(pairABI, pairAddress);

  // 2. Approve Router to spend LP tokens
  console.log(`Approving Router to spend ${liquidityAmount} LP tokens...`); // LP amounts often don't use 18 decimals
  const approveTx = pairContract.methods.approve(routerAddress, liquidityAmount); // Use raw amount, or convert if needed
  const approveGas = await approveTx.estimateGas({ from: account });
  const approveData = approveTx.encodeABI();
  const signedApproveTx = await web3.eth.accounts.signTransaction({
      to: pairAddress, data: approveData, gas: approveGas, from: account
  }, privateKey);
  const approveReceipt = await web3.eth.sendSignedTransaction(signedApproveTx.rawTransaction);
  console.log(`LP Token Approval successful: ${approveReceipt.transactionHash}`);

  // 3. Remove Liquidity
  console.log(`Removing ${liquidityAmount} liquidity...`);
  const removeLiqTx = routerContract.methods.removeLiquidityETH(
    usdtAddress,        // The ERC20 token address in the pair
    liquidityAmount,    // Amount of LP tokens to burn
    usdtAmountMin,      // Min USDT to receive
    wjuAmountMin,       // Min WJU to receive
    account,            // Address to receive underlying tokens
    deadline
  );

  const removeLiqGas = await removeLiqTx.estimateGas({ from: account });
  const removeLiqData = removeLiqTx.encodeABI();

  const signedRemoveLiqTx = await web3.eth.accounts.signTransaction({
      to: routerAddress,
      data: removeLiqData,
      gas: removeLiqGas,
      from: account
      // No value needed for removeLiquidityETH
  }, privateKey);

  const removeLiqReceipt = await web3.eth.sendSignedTransaction(signedRemoveLiqTx.rawTransaction);
  console.log(`Remove Liquidity WJU-USDT successful: ${removeLiqReceipt.transactionHash}`);
}

// Example: Remove 1 LP token (assuming LP token has 18 decimals, adjust if not!)
const oneLpToken = web3.utils.toWei('1', 'ether');
const minUsdtExpected = web3.utils.toWei('49', 'ether'); // Example: Expect at least 49 USDT
const minWjuExpected = web3.utils.toWei('0.49', 'ether'); // Example: Expect at least 0.49 WJU

removeLiquidityWJUUSDT(oneLpToken, minUsdtExpected, minWjuExpected);
```

**Notes on Liquidity:**

* **Ratio:** When adding liquidity, provide tokens proportionally to the current reserve ratio in the pair. If you provide amounts at a different ratio, the router will use the optimal amount of one token based on the desired amount of the other, potentially leaving some of the first token unused (refunded if WJU, kept in wallet if ERC20).
* **First Provider:** The first liquidity provider sets the initial exchange rate.
* **LP Tokens:** Represent your share of the pool. They can be transferred or staked if other protocols support it. LP token amounts usually have 18 decimals but check the specific pair contract.
* **Impermanent Loss:** Be aware of impermanent loss – the potential difference in value between holding tokens in a liquidity pool versus just holding them in your wallet.

### 6. Using the Factory

While most interactions happen via the Router, you might need the Factory to:

* **Check if a pair exists:** `getPair(address tokenA, address tokenB)` returns the pair address or `address(0)`.
* **Get pair address deterministically:** Use the `pairFor` logic (see `JUV2Library` in the Router code) which involves `create2`, the factory address, token addresses, and the `INIT_CODE_PAIR_HASH` (`0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b`).
* **Create a pair:** `createPair(address tokenA, address tokenB)` creates a new pair contract if it doesn't exist. Usually called by the Router automatically when needed during `addLiquidity` if the pair is new.

**Example: Calculate Pair Address (Solidity Helper)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0; // Use appropriate version for your contract

library PairUtil {
    // Calculates the CREATE2 address for a pair (logic adapted from JUV2Library)
    function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair) {
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        // INIT_CODE_PAIR_HASH for the provided Factory/Pair bytecode
        bytes32 initCodeHash = 0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b;

        pair = address(uint160(uint(keccak256(abi.encodePacked(
                hex'ff',
                factory,
                salt,
                initCodeHash
            )))));
    }
}

contract MyDEXInteractor {
    address constant FACTORY = 0x66682281BdfeC17fCBcae0480C77edFb0c489339;
    address constant WJU = 0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3;
    address constant USDT = 0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02;

    function getWjuUsdtPairAddress() external pure returns (address) {
        return PairUtil.pairFor(FACTORY, WJU, USDT);
    }
}
```

### 7. Key ABI Snippets (Router `0x6A64...`)

Use these snippets with Web3.js/ethers.js. Obtain the full ABI from the block explorer or compile the provided source code. **Note:** The ABIs provided in the prompt for the router were mostly incorrect (looked like ERC20 ABIs); use ABIs derived from the actual `JUV2Router02` code.

```json
[
  // --- Swap Functions ---
  {
    "inputs": [
      { "internalType": "uint256", "name": "amountIn", "type": "uint256" },
      { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" },
      { "internalType": "address[]", "name": "path", "type": "address[]" },
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "deadline", "type": "uint256" }
    ],
    "name": "swapExactTokensForTokens",
    "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ],
    "stateMutability": "nonpayable", "type": "function"
  },
  {
    "inputs": [
      { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" },
      { "internalType": "address[]", "name": "path", "type": "address[]" },
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "deadline", "type": "uint256" }
    ],
    "name": "swapExactETHForTokens",
    "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ],
    "stateMutability": "payable", "type": "function"
  },
  {
    "inputs": [
      { "internalType": "uint256", "name": "amountIn", "type": "uint256" },
      { "internalType": "uint256", "name": "amountOutMin", "type": "uint256" },
      { "internalType": "address[]", "name": "path", "type": "address[]" },
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "deadline", "type": "uint256" }
    ],
    "name": "swapExactTokensForETH",
    "outputs": [ { "internalType": "uint256[]", "name": "amounts", "type": "uint256[]" } ],
    "stateMutability": "nonpayable", "type": "function"
  },
  // --- Liquidity Functions ---
  {
    "inputs": [
      { "internalType": "address", "name": "tokenA", "type": "address" },
      { "internalType": "address", "name": "tokenB", "type": "address" },
      { "internalType": "uint256", "name": "amountADesired", "type": "uint256" },
      { "internalType": "uint256", "name": "amountBDesired", "type": "uint256" },
      { "internalType": "uint256", "name": "amountAMin", "type": "uint256" },
      { "internalType": "uint256", "name": "amountBMin", "type": "uint256" },
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "deadline", "type": "uint256" }
    ],
    "name": "addLiquidity",
    "outputs": [
      { "internalType": "uint256", "name": "amountA", "type": "uint256" },
      { "internalType": "uint256", "name": "amountB", "type": "uint256" },
      { "internalType": "uint256", "name": "liquidity", "type": "uint256" }
    ],
    "stateMutability": "nonpayable", "type": "function"
  },
  {
    "inputs": [
      { "internalType": "address", "name": "token", "type": "address" },
      { "internalType": "uint256", "name": "amountTokenDesired", "type": "uint256" },
      { "internalType": "uint256", "name": "amountTokenMin", "type": "uint256" },
      { "internalType": "uint256", "name": "amountETHMin", "type": "uint256" },
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "deadline", "type": "uint256" }
    ],
    "name": "addLiquidityETH",
    "outputs": [
      { "internalType": "uint256", "name": "amountToken", "type": "uint256" },
      { "internalType": "uint256", "name": "amountETH", "type": "uint256" },
      { "internalType": "uint256", "name": "liquidity", "type": "uint256" }
    ],
    "stateMutability": "payable", "type": "function"
  },
  {
    "inputs": [
      { "internalType": "address", "name": "tokenA", "type": "address" },
      { "internalType": "address", "name": "tokenB", "type": "address" },
      { "internalType": "uint256", "name": "liquidity", "type": "uint256" },
      { "internalType": "uint256", "name": "amountAMin", "type": "uint256" },
      { "internalType": "uint256", "name": "amountBMin", "type": "uint256" },
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "deadline", "type": "uint256" }
    ],
    "name": "removeLiquidity",
    "outputs": [
      { "internalType": "uint256", "name": "amountA", "type": "uint256" },
      { "internalType": "uint256", "name": "amountB", "type": "uint256" }
    ],
    "stateMutability": "nonpayable", "type": "function"
  },
  {
    "inputs": [
      { "internalType": "address", "name": "token", "type": "address" },
      { "internalType": "uint256", "name": "liquidity", "type": "uint256" },
      { "internalType": "uint256", "name": "amountTokenMin", "type": "uint256" },
      { "internalType": "uint256", "name": "amountETHMin", "type": "uint256" },
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "deadline", "type": "uint256" }
    ],
    "name": "removeLiquidityETH",
    "outputs": [
      { "internalType": "uint256", "name": "amountToken", "type": "uint256" },
      { "internalType": "uint256", "name": "amountETH", "type": "uint256" }
    ],
    "stateMutability": "nonpayable", "type": "function"
  },
  // --- Read/Helper Functions ---
   {
    "inputs": [],
    "name": "factory",
    "outputs": [ { "internalType": "address", "name": "", "type": "address" } ],
    "stateMutability": "pure", "type": "function"
  },
   {
    "inputs": [],
    "name": "WETH", // Should return the WJU address
    "outputs": [ { "internalType": "address", "name": "", "type": "address" } ],
    "stateMutability": "pure", "type": "function"
  },
  {
    "inputs": [
      {"internalType": "uint256", "name": "amountIn", "type": "uint256"},
      {"internalType": "address[]", "name": "path", "type": "address[]"}
    ],
    "name": "getAmountsOut",
    "outputs": [ {"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"} ],
    "stateMutability": "view", "type": "function"
  },
  {
    "inputs": [
      {"internalType": "uint256", "name": "amountOut", "type": "uint256"},
      {"internalType": "address[]", "name": "path", "type": "address[]"}
    ],
    "name": "getAmountsIn",
    "outputs": [ {"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"} ],
    "stateMutability": "view", "type": "function"
  }
]
```

**(Remember to obtain and use the correct, complete ABIs for WJU, USDT, Factory, and Pair contracts as needed).**

