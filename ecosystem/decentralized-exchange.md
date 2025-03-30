# Decentralized Exchange

**Core Components:**

* **Router (`JUV2Router02`)**: The main entry contract for user interaction, used to execute swaps and add/remove liquidity. Similar to Uniswap V2 Router.
* **Factory (`JUV2Factory`)**: Creates and manages pair contracts.
* **Pair (`JUV2Pair`)**: Represents a specific token pair (e.g., WJU-USDT), holds the liquidity pool, and functions as an ERC20 LP token.
* **WJU (Wrapped JU)**: An ERC20 wrapped version of the native JU token, used for DEX interactions.
* **USDT**: An example ERC20 token on the network.

**1. Prerequisites and Network Setup**

* **Network**: JuChain Testnet
  * **RPC URL**: `https://testnet-rpc.juchain.org`
  * **Chain ID**: `202599`
  * **Block Explorer**: `https://testnet.juscan.io`
* **Tools**:
  * Node.js environment and Web3.js library (`const { Web3 } = require('web3');`) or similar libraries (ethers.js).
  * Solidity development environment (optional, for contract interaction or understanding).
* **Account and Credentials**:
  * An external owned account (EOA) address (`USER_ADDRESS`).
  * The private key of that account (`PRIVATE_KEY`), **must be securely stored and managed**, never hardcoded or exposed.
  * The account needs to hold native JU (for gas payments) and tokens required for interaction (such as WJU, USDT).
* **Basic Setup**:
  * Initialize Web3 instance: `const web3 = new Web3(RPC_URL);`
  * Create contract instances: Use the corresponding ABIs (see Section 7) and contract addresses to create Web3 Contract objects for WJU, USDT, Router, and Factory. Pair contract instances are typically created dynamically after obtaining their addresses.

**2. Core Contract Addresses and Details**

| Contract           | Address                                                              | Description                                                                                               |
| ------------------ | -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **WJU**            | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`                         | Wrapped JU. Use `deposit()` (payable) to wrap, `withdraw()` to unwrap. 18 decimals.                       |
| **USDT**           | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`                         | Standard ERC20 token. Usually 18 decimals (confirm with on-chain `decimals()`).                           |
| **Factory**        | `0x66682281BdfeC17fCBcae0480C77edFb0c489339`                         | Deploys and tracks `JUV2Pair` contracts.                                                                  |
| **Router**         | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0`                         | **Main interaction entry point**. Treats WJU address as its `WETH` address (verify with `router.WETH()`). |
| **Pair Init Hash** | `0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b` | Used for deterministic pair address calculation (with `create2`).                                         |

**Key Configuration:** The router contract (`JUV2Router02`) **must** point to the WJU address on JuChain (`0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`) as its `WETH` address. This allows functions with the `ETH` suffix in the router to handle WJU correctly.

**3. Interacting with WJU (Wrapping/Unwrapping)**

Converting native JU to ERC20-compatible WJU is a prerequisite for interacting with the DEX.

* **Wrap JU**: Call the `deposit()` payable function on the WJU contract.
  * **Operation:** Construct a transaction to the WJU contract address, call `deposit()` (provided in the ABI), and include a `value` field in the transaction with the amount of native JU you wish to wrap (in Wei).
* **Unwrap WJU**: Call the `withdraw(uint256 wad)` function on the WJU contract.
  * **Operation:** Call the `withdraw` function with the parameter `wad` set to the amount of WJU you wish to unwrap back to native JU (in Wei). This transaction does not include a `value`.

**4. Swap Functionality (Using the Router)**

Use the router contract to swap tokens using existing liquidity pools.

**Common Swap Functions:**

* `swapExactETHForTokens`: Swap an **exact amount** of WJU for **at least** a specified amount of an ERC20 token.
* `swapExactTokensForETH`: Swap an **exact amount** of an ERC20 token for **at least** a specified amount of WJU.
* `swapExactTokensForTokens`: Swap an **exact amount** of an input ERC20 token for **at least** a specified amount of an output ERC20 token.

**Core Parameters:**

* `amountIn` / `amountOut`: The amounts involved.
* `amountOutMin` / `amountInMax`: Slippage control, ensuring you receive no less than/pay no more than this value.
* `path`: An array of addresses defining the swap path, e.g., `[WJU_ADDRESS, USDT_ADDRESS]` or `[USDT_ADDRESS, WJU_ADDRESS]`.
* `to`: The address receiving the tokens.
* `deadline`: Unix timestamp after which the transaction becomes invalid.

**Process: Swapping WJU for USDT (calling `swapExactETHForTokens`)**

1. **Determine Input:** Decide the exact amount of WJU to spend (`wjuAmountIn`, in Wei).
2. **Calculate Minimum Output (Slippage Control):**
   * Call the router's `getAmountsOut(wjuAmountIn, [WJU_ADDRESS, USDT_ADDRESS])` view function to get the expected USDT output amount `expectedUsdtAmount`.
   * Calculate `usdtAmountOutMin = expectedUsdtAmount * (1 - slippageTolerance)` based on your acceptable slippage percentage (e.g., 1% or 0.01). **Note:** Use BigInt when handling large number calculations.
3. **Prepare Transaction:**
   * Set `deadline` (e.g., current time + 10 minutes).
   * Construct a `swapExactETHForTokens` function call with parameters: `usdtAmountOutMin`, `path = [WJU_ADDRESS, USDT_ADDRESS]`, `to = USER_ADDRESS`, `deadline`.
4. **Send Transaction:**
   * Sign the constructed transaction (using `PRIVATE_KEY`).
   * When sending the transaction, **you must include `value: wjuAmountIn`** as WJU is transferred as the native token.
   * It's recommended to use `estimateGas` to estimate the Gas Limit and potentially add some buffer (e.g., \* 1.1).

**Process: Swapping USDT for WJU (calling `swapExactTokensForETH`)**

1. **Determine Input:** Decide the exact amount of USDT to spend (`usdtAmountIn`, in Wei).
2. **Approve Router:**
   * Call the **USDT contract's** `approve(ROUTER_ADDRESS, usdtAmountIn)` function.
   * **Must wait** for this approval transaction to be confirmed before proceeding.
   * _Optimization:_ First call `usdtContract.methods.allowance(USER_ADDRESS, ROUTER_ADDRESS)` to check existing allowance; skip approval if sufficient.
3. **Calculate Minimum Output (Slippage Control):**
   * Call the router's `getAmountsOut(usdtAmountIn, [USDT_ADDRESS, WJU_ADDRESS])` to get the expected WJU output amount `expectedWjuAmount`.
   * Calculate `wjuAmountOutMin = expectedWjuAmount * (1 - slippageTolerance)`.
4. **Prepare Transaction:**
   * Set `deadline`.
   * Construct a `swapExactTokensForETH` function call with parameters: `usdtAmountIn`, `wjuAmountOutMin`, `path = [USDT_ADDRESS, WJU_ADDRESS]`, `to = USER_ADDRESS`, `deadline`.
5. **Send Transaction:**
   * Sign and send the transaction.
   * This transaction **does not include a `value`**.
   * Similarly, it's recommended to estimate Gas.

**Swap Considerations:**

* **Slippage:** `amountOutMin`/`amountInMax` are crucial protections against unfavorable price movements.
* **Gas:** All write operations require Gas (native JU).
* **Deadline:** Prevents transactions from staying in the mempool too long and executing at prices that significantly differ from expectations.
* **Path (`path`):** Ensure the path is correct; complex swaps may require WJU as an intermediate route.

**5. Liquidity Management (Using the Router)**

Provide liquidity to token pairs to earn trading fees and receive LP tokens representing your share.

**Common Liquidity Functions:**

* `addLiquidityETH`: Add liquidity for WJU and an ERC20 token.
* `removeLiquidityETH`: Remove liquidity for WJU and an ERC20 token.
* `addLiquidity`: Add liquidity for two ERC20 tokens.
* `removeLiquidity`: Remove liquidity for two ERC20 tokens.

**Process: Adding WJU-USDT Liquidity (calling `addLiquidityETH`)**

1. **Determine Desired Amounts:** Decide the desired amount of WJU (`wjuAmountDesired`) and USDT (`usdtAmountDesired`) to provide. Note: The actual ratio added will be determined by the current pool, with your provided amounts serving as maximums and desired ratios.
2. **Calculate Minimum Acceptable Amounts (Slippage Control):** Based on acceptable slippage, calculate `wjuAmountMin` and `usdtAmountMin`.
3. **Check Balances:** Ensure your account has sufficient WJU (provided via `value`) and USDT.
4. **Approve Router (for USDT):**
   * Call the **USDT contract's** `approve(ROUTER_ADDRESS, usdtAmountDesired)` function.
   * Wait for the approval transaction to be confirmed.
   * _Optimization:_ First check `allowance`; skip if sufficient.
5. **Prepare Transaction:**
   * Set `deadline`.
   * Construct an `addLiquidityETH` function call with parameters: `token = USDT_ADDRESS`, `amountTokenDesired = usdtAmountDesired`, `amountTokenMin = usdtAmountMin`, `amountETHMin = wjuAmountMin`, `to = USER_ADDRESS`, `deadline`.
6. **Send Transaction:**
   * Sign and send the transaction.
   * **Must include `value: wjuAmountDesired`** to provide WJU.
   * It's recommended to estimate Gas. Upon success, `USER_ADDRESS` will receive WJU-USDT LP tokens.

**Process: Removing WJU-USDT Liquidity (calling `removeLiquidityETH`)**

1. **Determine Removal Amount:** Decide the amount of LP tokens to remove (`liquidityAmount`, in Wei). LP tokens typically also have 18 decimals.
2. **Calculate Minimum Return Amounts (Slippage Control):** Based on the current pool ratio and acceptable slippage, estimate the minimum WJU (`wjuAmountMin`) and USDT (`usdtAmountMin`) to receive. You can query the Pair contract's `getReserves` and LP token's `totalSupply` to assist in calculations.
3. **Get Pair Address:** Call `factoryContract.methods.getPair(WJU_ADDRESS, USDT_ADDRESS)` to get the WJU-USDT Pair contract address (`pairAddress`).
4. **Approve Router (for LP Tokens):**
   * Create a Pair contract instance: `const pairContract = new web3.eth.Contract(ABIs.JUV2PAIR_ABI, pairAddress);`
   * Call the **Pair contract's** `approve(ROUTER_ADDRESS, liquidityAmount)` function.
   * Wait for the approval transaction to be confirmed.
5. **Prepare Transaction:**
   * Set `deadline`.
   * Construct a `removeLiquidityETH` function call with parameters: `token = USDT_ADDRESS`, `liquidity = liquidityAmount`, `amountTokenMin = usdtAmountMin`, `amountETHMin = wjuAmountMin`, `to = USER_ADDRESS`, `deadline`.
6. **Send Transaction:**
   * Sign and send the transaction.
   * This transaction **does not include a `value`**.
   * It's recommended to estimate Gas. Upon success, `USER_ADDRESS` will receive WJU and USDT.

**Liquidity Management Considerations:**

* **Impermanent Loss:** An inherent risk when providing liquidity, which occurs when the relative price of tokens changes.
* **LP Tokens:** LP tokens themselves are ERC20 tokens that can be transferred or used in other DeFi protocols (if supported).
* **First Addition:** If you're the first provider, you'll set the initial exchange rate.

**6. Using the Factory**

The factory contract is primarily used to create and query pairs.

* **Query Pair Address:**
  * Call `getPair(address tokenA, address tokenB)`.
  * If a non-zero address is returned, the pair exists.
  * If `0x000...000` is returned, the pair doesn't exist.
* **Create Pair:**
  * If you confirm through `getPair` that the pair doesn't exist, you can call `createPair(address tokenA, address tokenB)`.
  * The `tokenA` and `tokenB` addresses can be in any order.
  * The factory will deploy a new Pair contract using `create2` and emit a `PairCreated` event.
  * Typically, the `addLiquidity*` functions automatically call `createPair` if the pair doesn't exist, but manual calling ensures its existence.
* **Get All Pairs Information:** `allPairsLength()` and `allPairs(uint index)` are available, but iterating may be costly.
* **Deterministically Calculate Pair Address:** Using the `pairFor` logic (see Solidity example below), combined with the factory address, sorted token addresses, and `initCodeHash`, you can pre-calculate the pair address off-chain.

**Example: Calculating Pair Address (Solidity Helper Library)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

library PairUtil {
    // Calculate the pair address for a given factory and token pair
    function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair) {
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        // JuChain DEX Pair contract init code hash
        bytes32 initCodeHash = 0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b;
        pair = address(uint160(uint(keccak256(abi.encodePacked(
                hex'ff', factory, salt, initCodeHash
            )))));
    }
}

contract MyDEXInteractor {
    address constant FACTORY = 0x66682281BdfeC17fCBcae0480C77edFb0c489339;
    address constant WJU = 0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3;
    address constant USDT = 0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02;

    // Get the expected address of the WJU-USDT pair
    function getWjuUsdtPairAddress() external pure returns (address) {
        return PairUtil.pairFor(FACTORY, WJU, USDT);
    }
}
```

**7. Contract ABIs**

**Strongly Recommended:** Always verify and obtain the latest, most complete official ABIs for specified contract addresses from the JuChain block explorer (`https://testnet.juscan.io`).

```json
{
  "WJU_ABI": [
    {"constant": false, "inputs": [{"name": "guy", "type": "address"}, {"name": "wad", "type": "uint256"}], "name": "approve", "outputs": [{"name": "", "type": "bool"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": false, "inputs": [], "name": "deposit", "outputs": [], "payable": true, "stateMutability": "payable", "type": "function"},
    {"constant": false, "inputs": [{"name": "dst", "type": "address"}, {"name": "wad", "type": "uint256"}], "name": "transfer", "outputs": [{"name": "", "type": "bool"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": false, "inputs": [{"name": "src", "type": "address"}, {"name": "dst", "type": "address"}, {"name": "wad", "type": "uint256"}], "name": "transferFrom", "outputs": [{"name": "", "type": "bool"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": false, "inputs": [{"name": "wad", "type": "uint256"}], "name": "withdraw", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"payable": true, "stateMutability": "payable", "type": "fallback"},
    {"anonymous": false, "inputs": [{"indexed": true, "name": "src", "type": "address"}, {"indexed": true, "name": "guy", "type": "address"}, {"indexed": false, "name": "wad", "type": "uint256"}], "name": "Approval", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "name": "src", "type": "address"}, {"indexed": true, "name": "dst", "type": "address"}, {"indexed": false, "name": "wad", "type": "uint256"}], "name": "Transfer", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "name": "dst", "type": "address"}, {"indexed": false, "name": "wad", "type": "uint256"}], "name": "Deposit", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "name": "src", "type": "address"}, {"indexed": false, "name": "wad", "type": "uint256"}], "name": "Withdrawal", "type": "event"},
    {"constant": true, "inputs": [{"name": "", "type": "address"}, {"name": "", "type": "address"}], "name": "allowance", "outputs": [{"name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [{"name": "", "type": "address"}], "name": "balanceOf", "outputs": [{"name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "decimals", "outputs": [{"name": "", "type": "uint8"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "name", "outputs": [{"name": "", "type": "string"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "symbol", "outputs": [{"name": "", "type": "string"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "totalSupply", "outputs": [{"name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"}
  ],
  "USDT_ABI": [
    {"inputs": [], "stateMutability": "nonpayable", "type": "constructor"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "owner", "type": "address"}, {"indexed": true, "internalType": "address", "name": "spender", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "value", "type": "uint256"}], "name": "Approval", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "from", "type": "address"}, {"indexed": true, "internalType": "address", "name": "to", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "value", "type": "uint256"}], "name": "Transfer", "type": "event"},
    {"inputs": [{"internalType": "address", "name": "owner", "type": "address"}, {"internalType": "address", "name": "spender", "type": "address"}], "name": "allowance", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "spender", "type": "address"}, {"internalType": "uint256", "name": "amount", "type": "uint256"}], "name": "approve", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "account", "type": "address"}], "name": "balanceOf", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
    {"inputs": [], "name": "decimals", "outputs": [{"internalType": "uint8", "name": "", "type": "uint8"}], "stateMutability": "view", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "spender", "type": "address"}, {"internalType": "uint256", "name": "subtractedValue", "type": "uint256"}], "name": "decreaseAllowance", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "spender", "type": "address"}, {"internalType": "uint256", "name": "addedValue", "type": "uint256"}], "name": "increaseAllowance", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [], "name": "name", "outputs": [{"internalType": "string", "name": "", "type": "string"}], "stateMutability": "view", "type": "function"},
    {"inputs": [], "name": "symbol", "outputs": [{"internalType": "string", "name": "", "type": "string"}], "stateMutability": "view", "type": "function"},
    {"inputs": [], "name": "totalSupply", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "amount", "type": "uint256"}], "name": "transfer", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "from", "type": "address"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "amount", "type": "uint256"}], "name": "transferFrom", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "stateMutability": "nonpayable", "type": "function"}
  ],
  "FACTORY_ABI": [
    {"inputs": [{"internalType": "address", "name": "_feeToSetter", "type": "address"}], "payable": false, "stateMutability": "nonpayable", "type": "constructor"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "token0", "type": "address"}, {"indexed": true, "internalType": "address", "name": "token1", "type": "address"}, {"indexed": false, "internalType": "address", "name": "pair", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "", "type": "uint256"}], "name": "PairCreated", "type": "event"},
    {"constant": true, "inputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "name": "allPairs", "outputs": [{"internalType": "address", "name": "pair", "type": "address"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "allPairsLength", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "tokenA", "type": "address"}, {"internalType": "address", "name": "tokenB", "type": "address"}], "name": "createPair", "outputs": [{"internalType": "address", "name": "pair", "type": "address"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [], "name": "feeTo", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "feeToSetter", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [{"internalType": "address", "name": "", "type": "address"}, {"internalType": "address", "name": "", "type": "address"}], "name": "getPair", "outputs": [{"internalType": "address", "name": "pair", "type": "address"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "_feeTo", "type": "address"}], "name": "setFeeTo", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "_feeToSetter", "type": "address"}], "name": "setFeeToSetter", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"}
  ],
  "ROUTER_ABI": [
    {"inputs": [{"internalType": "address", "name": "_factory", "type": "address"}, {"internalType": "address", "name": "_WETH", "type": "address"}], "stateMutability": "nonpayable", "type": "constructor"},
    {"stateMutability": "payable", "type": "receive"},
    {"inputs": [], "name": "WETH", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "stateMutability": "pure", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "tokenA", "type": "address"}, {"internalType": "address", "name": "tokenB", "type": "address"}, {"internalType": "uint256", "name": "amountADesired", "type": "uint256"}, {"internalType": "uint256", "name": "amountBDesired", "type": "uint256"}, {"internalType": "uint256", "name": "amountAMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountBMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "addLiquidity", "outputs": [{"internalType": "uint256", "name": "amountA", "type": "uint256"}, {"internalType": "uint256", "name": "amountB", "type": "uint256"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "token", "type": "address"}, {"internalType": "uint256", "name": "amountTokenDesired", "type": "uint256"}, {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "addLiquidityETH", "outputs": [{"internalType": "uint256", "name": "amountToken", "type": "uint256"}, {"internalType": "uint256", "name": "amountETH", "type": "uint256"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}], "stateMutability": "payable", "type": "function"},
    {"inputs": [], "name": "factory", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "stateMutability": "pure", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountOut", "type": "uint256"}, {"internalType": "uint256", "name": "reserveIn", "type": "uint256"}, {"internalType": "uint256", "name": "reserveOut", "type": "uint256"}], "name": "getAmountIn", "outputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}], "stateMutability": "pure", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}, {"internalType": "uint256", "name": "reserveIn", "type": "uint256"}, {"internalType": "uint256", "name": "reserveOut", "type": "uint256"}], "name": "getAmountOut", "outputs": [{"internalType": "uint256", "name": "amountOut", "type": "uint256"}], "stateMutability": "pure", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountOut", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}], "name": "getAmountsIn", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "view", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}], "name": "getAmountsOut", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "view", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountA", "type": "uint256"}, {"internalType": "uint256", "name": "reserveA", "type": "uint256"}, {"internalType": "uint256", "name": "reserveB", "type": "uint256"}], "name": "quote", "outputs": [{"internalType": "uint256", "name": "amountB", "type": "uint256"}], "stateMutability": "pure", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "tokenA", "type": "address"}, {"internalType": "address", "name": "tokenB", "type": "address"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}, {"internalType": "uint256", "name": "amountAMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountBMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "removeLiquidity", "outputs": [{"internalType": "uint256", "name": "amountA", "type": "uint256"}, {"internalType": "uint256", "name": "amountB", "type": "uint256"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "token", "type": "address"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}, {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "removeLiquidityETH", "outputs": [{"internalType": "uint256", "name": "amountToken", "type": "uint256"}, {"internalType": "uint256", "name": "amountETH", "type": "uint256"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "token", "type": "address"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}, {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "removeLiquidityETHSupportingFeeOnTransferTokens", "outputs": [{"internalType": "uint256", "name": "amountETH", "type": "uint256"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "token", "type": "address"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}, {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}, {"internalType": "bool", "name": "approveMax", "type": "bool"}, {"internalType": "uint8", "name": "v", "type": "uint8"}, {"internalType": "bytes32", "name": "r", "type": "bytes32"}, {"internalType": "bytes32", "name": "s", "type": "bytes32"}], "name": "removeLiquidityETHWithPermit", "outputs": [{"internalType": "uint256", "name": "amountToken", "type": "uint256"}, {"internalType": "uint256", "name": "amountETH", "type": "uint256"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "token", "type": "address"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}, {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}, {"internalType": "bool", "name": "approveMax", "type": "bool"}, {"internalType": "uint8", "name": "v", "type": "uint8"}, {"internalType": "bytes32", "name": "r", "type": "bytes32"}, {"internalType": "bytes32", "name": "s", "type": "bytes32"}], "name": "removeLiquidityETHWithPermitSupportingFeeOnTransferTokens", "outputs": [{"internalType": "uint256", "name": "amountETH", "type": "uint256"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "address", "name": "tokenA", "type": "address"}, {"internalType": "address", "name": "tokenB", "type": "address"}, {"internalType": "uint256", "name": "liquidity", "type": "uint256"}, {"internalType": "uint256", "name": "amountAMin", "type": "uint256"}, {"internalType": "uint256", "name": "amountBMin", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}, {"internalType": "bool", "name": "approveMax", "type": "bool"}, {"internalType": "uint8", "name": "v", "type": "uint8"}, {"internalType": "bytes32", "name": "r", "type": "bytes32"}, {"internalType": "bytes32", "name": "s", "type": "bytes32"}], "name": "removeLiquidityWithPermit", "outputs": [{"internalType": "uint256", "name": "amountA", "type": "uint256"}, {"internalType": "uint256", "name": "amountB", "type": "uint256"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}, {"internalType": "uint256", "name": "amountOutMin", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapExactETHForTokens", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "payable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountOutMin", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapExactETHForTokensSupportingFeeOnTransferTokens", "outputs": [], "stateMutability": "payable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}, {"internalType": "uint256", "name": "amountOutMin", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapExactTokensForETH", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}, {"internalType": "uint256", "name": "amountOutMin", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapExactTokensForETHSupportingFeeOnTransferTokens", "outputs": [], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}, {"internalType": "uint256", "name": "amountOutMin", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapExactTokensForTokens", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountIn", "type": "uint256"}, {"internalType": "uint256", "name": "amountOutMin", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapExactTokensForTokensSupportingFeeOnTransferTokens", "outputs": [], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountOut", "type": "uint256"}, {"internalType": "uint256", "name": "amountInMax", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapTokensForExactETH", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountOut", "type": "uint256"}, {"internalType": "uint256", "name": "amountInMax", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapTokensForExactTokens", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "nonpayable", "type": "function"},
    {"inputs": [{"internalType": "uint256", "name": "amountOut", "type": "uint256"}, {"internalType": "address[]", "name": "path", "type": "address[]"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}], "name": "swapETHForExactTokens", "outputs": [{"internalType": "uint256[]", "name": "amounts", "type": "uint256[]"}], "stateMutability": "payable", "type": "function"}
  ],
  "JUV2PAIR_ABI": [
    {"inputs": [], "payable": false, "stateMutability": "nonpayable", "type": "constructor"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "owner", "type": "address"}, {"indexed": true, "internalType": "address", "name": "spender", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "value", "type": "uint256"}], "name": "Approval", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "sender", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "amount0", "type": "uint256"}, {"indexed": false, "internalType": "uint256", "name": "amount1", "type": "uint256"}, {"indexed": true, "internalType": "address", "name": "to", "type": "address"}], "name": "Burn", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "sender", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "amount0", "type": "uint256"}, {"indexed": false, "internalType": "uint256", "name": "amount1", "type": "uint256"}], "name": "Mint", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "sender", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "amount0In", "type": "uint256"}, {"indexed": false, "internalType": "uint256", "name": "amount1In", "type": "uint256"}, {"indexed": false, "internalType": "uint256", "name": "amount0Out", "type": "uint256"}, {"indexed": false, "internalType": "uint256", "name": "amount1Out", "type": "uint256"}, {"indexed": true, "internalType": "address", "name": "to", "type": "address"}], "name": "Swap", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": false, "internalType": "uint112", "name": "reserve0", "type": "uint112"}, {"indexed": false, "internalType": "uint112", "name": "reserve1", "type": "uint112"}], "name": "Sync", "type": "event"},
    {"anonymous": false, "inputs": [{"indexed": true, "internalType": "address", "name": "from", "type": "address"}, {"indexed": true, "internalType": "address", "name": "to", "type": "address"}, {"indexed": false, "internalType": "uint256", "name": "value", "type": "uint256"}], "name": "Transfer", "type": "event"},
    {"constant": true, "inputs": [], "name": "DOMAIN_SEPARATOR", "outputs": [{"internalType": "bytes32", "name": "", "type": "bytes32"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "MINIMUM_LIQUIDITY", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "pure", "type": "function"},
    {"constant": true, "inputs": [], "name": "PERMIT_TYPEHASH", "outputs": [{"internalType": "bytes32", "name": "", "type": "bytes32"}], "payable": false, "stateMutability": "pure", "type": "function"},
    {"constant": true, "inputs": [{"internalType": "address", "name": "owner", "type": "address"}, {"internalType": "address", "name": "spender", "type": "address"}], "name": "allowance", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "spender", "type": "address"}, {"internalType": "uint256", "name": "value", "type": "uint256"}], "name": "approve", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [{"internalType": "address", "name": "owner", "type": "address"}], "name": "balanceOf", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "to", "type": "address"}], "name": "burn", "outputs": [{"internalType": "uint256", "name": "amount0", "type": "uint256"}, {"internalType": "uint256", "name": "amount1", "type": "uint256"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [], "name": "decimals", "outputs": [{"internalType": "uint8", "name": "", "type": "uint8"}], "payable": false, "stateMutability": "pure", "type": "function"},
    {"constant": true, "inputs": [], "name": "factory", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "getReserves", "outputs": [{"internalType": "uint112", "name": "_reserve0", "type": "uint112"}, {"internalType": "uint112", "name": "_reserve1", "type": "uint112"}, {"internalType": "uint32", "name": "_blockTimestampLast", "type": "uint32"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "_token0", "type": "address"}, {"internalType": "address", "name": "_token1", "type": "address"}], "name": "initialize", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [], "name": "kLast", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "to", "type": "address"}], "name": "mint", "outputs": [{"internalType": "uint256", "name": "liquidity", "type": "uint256"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [], "name": "name", "outputs": [{"internalType": "string", "name": "", "type": "string"}], "payable": false, "stateMutability": "pure", "type": "function"},
    {"constant": true, "inputs": [{"internalType": "address", "name": "owner", "type": "address"}], "name": "nonces", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "owner", "type": "address"}, {"internalType": "address", "name": "spender", "type": "address"}, {"internalType": "uint256", "name": "value", "type": "uint256"}, {"internalType": "uint256", "name": "deadline", "type": "uint256"}, {"internalType": "uint8", "name": "v", "type": "uint8"}, {"internalType": "bytes32", "name": "r", "type": "bytes32"}, {"internalType": "bytes32", "name": "s", "type": "bytes32"}], "name": "permit", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [], "name": "price0CumulativeLast", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "price1CumulativeLast", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "to", "type": "address"}], "name": "skim", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "uint256", "name": "amount0Out", "type": "uint256"}, {"internalType": "uint256", "name": "amount1Out", "type": "uint256"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "bytes", "name": "data", "type": "bytes"}], "name": "swap", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [], "name": "symbol", "outputs": [{"internalType": "string", "name": "", "type": "string"}], "payable": false, "stateMutability": "pure", "type": "function"},
    {"constant": false, "inputs": [], "name": "sync", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": true, "inputs": [], "name": "token0", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "token1", "outputs": [{"internalType": "address", "name": "", "type": "address"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": true, "inputs": [], "name": "totalSupply", "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}], "payable": false, "stateMutability": "view", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "value", "type": "uint256"}], "name": "transfer", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "payable": false, "stateMutability": "nonpayable", "type": "function"},
    {"constant": false, "inputs": [{"internalType": "address", "name": "from", "type": "address"}, {"internalType": "address", "name": "to", "type": "address"}, {"internalType": "uint256", "name": "value", "type": "uint256"}], "name": "transferFrom", "outputs": [{"internalType": "bool", "name": "", "type": "bool"}], "payable": false, "stateMutability": "nonpayable", "type": "function"}
  ]
}
```

```json
```

{% hint style="info" %}
**Best Practices for Interaction (Reference Implementation):**

* **Transaction Sending Wrapper:** Use helper functions to handle nonce retrieval, gas price setting, transaction signing (`web3.eth.accounts.signTransaction`), and sending (`web3.eth.sendSignedTransaction`), including `try...catch` error handling.
* **Gas Estimation:** Use the contract method's `.estimateGas()` before sending transactions, and consider adding a small buffer (e.g., 10-20%) to prevent gas insufficiency due to chain state changes.
* **Slippage Control:** For Swap and liquidity operations, always calculate and use `amount*Min` or `amount*Max` parameters.
* **Approve Check:** Before executing operations that spend ERC20 tokens, check the existing `allowance` to avoid unnecessary `approve` transactions.
* **Deadline:** Set reasonable `deadline` for time-sensitive operations (Swap, Liquidity).
* **Large Number Handling:** Use `BigInt` (Javascript native) or `web3.utils.toWei/fromWei` and BN.js (built into Web3.js) to handle token amounts.
{% endhint %}

{% hint style="warning" %}
Always thoroughly test on testnet before deploying to mainnet or handling real assets. Always refer to the official JuChain documentation and block explorer for the latest, most authoritative information and complete ABIs. Manage your private keys securely!
{% endhint %}
