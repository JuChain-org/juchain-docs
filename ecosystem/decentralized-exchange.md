# Decentralized Exchange

JuChain provides a decentralized exchange (DEX) functionality based on the Uniswap V2 model, allowing users to perform token swaps and provide liquidity.

{% hint style="info" %}
JuChain currently supports Solidity versions <= 0.8.8 for compilation. Support for later versions will be gradually introduced based on official announcements.
{% endhint %}

### Mainnet Deployment Information

* **Network**: JuChain Mainnet
* **Contract Address**: `0x49E5c7f25711abe668F404307b27f4bE4836B0e7`
* **Deployment Address**: `0x7389F1B4717F5152B6Cc107bce4A42a11dC0b76E`
* **Price Update Address**: `0xa6F32fe2920AcF559699825AFaC493aa4F49Ac1D`
* **Contract Permission Address**: `0x5021A15FaAFEFEC1daCB1c8b24FFE3F3E3f7277b`

### Testnet Deployment Information

**Core Components:**

* **Router (`JUV2Router02`)**: The main entry point contract for user interactions, used for executing swaps and adding/removing liquidity. Similar to Uniswap V2 Router.
* **Factory (`JUV2Factory`)**: Creates and manages trading pair contracts.
* **Trading Pair (`JUV2Pair`)**: Represents a specific token pair (e.g., WJU-USDT), holds the liquidity pool, and serves as an ERC20 LP token.
* **WJU (Wrapped JU)**: The ERC20 wrapped version of the native JU token, used for DEX interactions.
* **USDT**: An example ERC20 token on the network.

**1. Prerequisites and Network Setup**

* **Mainnet**:
  * **RPC URL**: `https://rpc.juchain.org`
  * **Chain ID**: `202599`
  * **Block Explorer**: `https://juscan.io`
* **Testnet**:
  * **RPC URL**: `https://testnet-rpc.juchain.org`
  * **Chain ID**: `202599`
  * **Block Explorer**: `https://testnet.juscan.io`
* **Tools**:
  * Node.js environment and Web3.js library (`const { Web3 } = require('web3');`) or similar libraries (ethers.js).
  * Solidity development environment (optional, for contract interaction or understanding).
* **Account and Credentials**:
  * An external account (EOA) address (`USER_ADDRESS`).
  * The private key (`PRIVATE_KEY`) for this account, which **must be securely stored and managed**, never hardcoded or publicly exposed.
  * The account needs to hold native JU (for gas payments) and the tokens required for interaction (e.g., WJU, USDT).
* **Basic Setup**:
  * Initialize Web3 instance: `const web3 = new Web3(RPC_URL);`
  * Create contract instances: Use the appropriate ABI (see Section 7) and contract addresses to create Web3 Contract objects for WJU, USDT, Router, and Factory. Pair contract instances are typically created dynamically after obtaining their addresses.

**2. Core Contract Addresses and Details**

**Mainnet Contract Addresses**

| Contract           | Address                                                              | Description                                                                                                             |
| ------------------ | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **WJU**            | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`                         | Wrapped JU. `deposit()` (payable) to wrap, `withdraw()` to unwrap. 18 decimals.                                         |
| **USDT**           | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`                         | Standard ERC20 token. Usually 18 decimals (verify with on-chain `decimals()`).                                          |
| **Factory**        | `0xCcbcecDd7d8D115Df79fc85847F38F9A5965326c`                         | Deploys and tracks `JUV2Pair` trading pair contracts.                                                                   |
| **Router**         | `0x09f58Aa3C7A8101062855C66E43a83920EB23511`                         | **Main interaction entry point**. Internally treats WJU address as its `WETH` address (verifiable via `router.WETH()`). |
| **Pair Init Hash** | `0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b` | Used for deterministic pair address calculation (with `create2`).                                                       |

**Testnet Contract Addresses**

| Contract           | Address                                                              | Description                                                                                                             |
| ------------------ | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **WJU**            | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`                         | Wrapped JU. `deposit()` (payable) to wrap, `withdraw()` to unwrap. 18 decimals.                                         |
| **USDT**           | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`                         | Standard ERC20 token. Usually 18 decimals (verify with on-chain `decimals()`).                                          |
| **Factory**        | `0x66682281BdfeC17fCBcae0480C77edFb0c489339`                         | Deploys and tracks `JUV2Pair` trading pair contracts.                                                                   |
| **Router**         | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0`                         | **Main interaction entry point**. Internally treats WJU address as its `WETH` address (verifiable via `router.WETH()`). |
| **Pair Init Hash** | `0x2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b` | Used for deterministic pair address calculation (with `create2`).                                                       |

**Key Configuration:** The Router contract (`JUV2Router02`) **must** have its `WETH` address pointing to the WJU address on JuChain (`0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`). This enables the router's functions with `ETH` suffix to properly handle WJU.

**3. Interacting with WJU (Wrapping/Unwrapping)**

Converting native JU to ERC20-compatible WJU is a prerequisite for DEX interaction.

* **Wrapping JU**: Call the WJU contract's `deposit()` payable function.
  * **Operation:** Construct a transaction to the WJU contract address, calling `deposit()` (provided in ABI), with a `value` field containing the amount of native JU you wish to wrap (in Wei).
* **Unwrapping WJU**: Call the WJU contract's `withdraw(uint256 wad)` function.
  * **Operation:** Call the `withdraw` function with parameter `wad` being the amount of WJU you wish to unwrap back to native JU (in Wei). This transaction does not include a `value`.

**4. Swap Functionality (Using Router)**

Perform token swaps through the router contract using existing liquidity pools.

**Common Swap Functions:**

* `swapExactETHForTokens`: Swap an **exact amount** of WJU for **at least** a certain amount of ERC20 tokens.
* `swapExactTokensForETH`: Swap an **exact amount** of ERC20 tokens for **at least** a certain amount of WJU.
* `swapExactTokensForTokens`: Swap an **exact amount** of input ERC20 tokens for **at least** a certain amount of output ERC20 tokens.

**Core Parameters:**

* `amountIn` / `amountOut`: The amounts involved.
* `amountOutMin` / `amountInMax`: Slippage control, ensuring receiving not less than/paying not more than these values.
* `path`: Array of addresses defining the swap path, e.g., `[WJU_ADDRESS, USDT_ADDRESS]` or `[USDT_ADDRESS, WJU_ADDRESS]`.
* `to`: Address to receive the tokens.
* `deadline`: Unix timestamp after which the transaction becomes invalid.

**Process: Swapping WJU for USDT (Using `swapExactETHForTokens`)**

1. **Determine Input:** Decide the exact amount of WJU to spend (`wjuAmountIn`, in Wei).
2. **Calculate Minimum Output (Slippage Control):**
   * Call the router's `getAmountsOut(wjuAmountIn, [WJU_ADDRESS, USDT_ADDRESS])` view function to get the expected USDT output `expectedUsdtAmount`.
   * Calculate `usdtAmountOutMin = expectedUsdtAmount * (1 - slippageTolerance)` based on acceptable slippage percentage (e.g., 1% or 0.01). **Note:** Use BigInt for large number calculations.
3. **Prepare Transaction:**
   * Set `deadline` (e.g., current time + 10 minutes).
   * Construct `swapExactETHForTokens` function call with parameters: `usdtAmountOutMin`, `path = [WJU_ADDRESS, USDT_ADDRESS]`, `to = USER_ADDRESS`, `deadline`.
4. **Send Transaction:**
   * Sign the constructed transaction (using `PRIVATE_KEY`).
   * Send transaction with **`value: wjuAmountIn`**, as WJU is passed as native currency.
   * Recommend using `estimateGas` for Gas Limit estimation, with some buffer (e.g., \* 1.1).

**Process: Swapping USDT for WJU (Using `swapExactTokensForETH`)**

1. **Determine Input:** Decide the exact amount of USDT to spend (`usdtAmountIn`, in Wei).
2. **Approve Router:**
   * Call **USDT contract's** `approve(ROUTER_ADDRESS, usdtAmountIn)` function.
   * **Must wait** for this approval transaction to be confirmed before proceeding.
   * _Optimization:_ Can check existing allowance via `usdtContract.methods.allowance(USER_ADDRESS, ROUTER_ADDRESS)` and skip approval if sufficient.
3. **Calculate Minimum Output (Slippage Control):**
   * Call router's `getAmountsOut(usdtAmountIn, [USDT_ADDRESS, WJU_ADDRESS])` to get expected WJU output `expectedWjuAmount`.
   * Calculate `wjuAmountOutMin = expectedWjuAmount * (1 - slippageTolerance)`.
4. **Prepare Transaction:**
   * Set `deadline`.
   * Construct `swapExactTokensForETH` function call with parameters: `usdtAmountIn`, `wjuAmountOutMin`, `path = [USDT_ADDRESS, WJU_ADDRESS]`, `to = USER_ADDRESS`, `deadline`.
5. **Send Transaction:**
   * Sign and send transaction.
   * This transaction does **not** include `value`.
   * Recommend estimating Gas.

**Swap Considerations:**

* **Slippage:** `amountOutMin`/`amountInMax` are crucial protections against unfavorable price movements.
* **Gas:** All write operations require Gas (native JU).
* **Deadline:** Prevents transactions from being stuck in the mempool too long and executing at unfavorable prices.
* **Path (`path`):** Ensure correct path; complex swaps may need to route through WJU as an intermediary.

**5. Liquidity Management (Using Router)**

Provide liquidity to trading pairs to earn trading fees and receive LP tokens representing your share.

**Common Liquidity Functions:**

* `addLiquidityETH`: Add liquidity with WJU and an ERC20 token.
* `removeLiquidityETH`: Remove liquidity of WJU and an ERC20 token.
* `addLiquidity`: Add liquidity with two ERC20 tokens.
* `removeLiquidity`: Remove liquidity of two ERC20 tokens.

**Process: Adding WJU-USDT Liquidity (Using `addLiquidityETH`)**

1. **Determine Desired Amounts:** Decide the desired amounts of WJU (`wjuAmountDesired`) and USDT (`usdtAmountDesired`) to provide. Note: The actual ratio will be determined by the current pool, your amounts are maximums and desired ratios.
2. **Calculate Minimum Acceptable Amounts (Slippage Control):** Calculate `wjuAmountMin` and `usdtAmountMin` based on acceptable slippage.
3. **Check Balances:** Ensure account has sufficient WJU (provided via `value`) and USDT.
4. **Approve Router (for USDT):**
   * Call **USDT contract's** `approve(ROUTER_ADDRESS, usdtAmountDesired)` function.
   * Wait for approval transaction confirmation.
   * _Optimization:_ Can check existing `allowance` first and skip if sufficient.
5. **Prepare Transaction:**
   * Set `deadline`.
   * Construct `addLiquidityETH` function call with parameters: `token = USDT_ADDRESS`, `amountTokenDesired = usdtAmountDesired`, `amountTokenMin = usdtAmountMin`, `amountETHMin = wjuAmountMin`, `to = USER_ADDRESS`, `deadline`.
6. **Send Transaction:**
   * Sign and send transaction.
   * **Must include `value: wjuAmountDesired`** to provide WJU.
   * Recommend estimating Gas. Upon success, `USER_ADDRESS` will receive WJU-USDT LP tokens.

**Process: Removing WJU-USDT Liquidity (Using `removeLiquidityETH`)**

1. **Determine Removal Amount:** Decide the amount of LP tokens to remove (`liquidityAmount`, in Wei). LP tokens typically have 18 decimals.
2. **Calculate Minimum Return Amounts (Slippage Control):** Based on current pool ratio and acceptable slippage, estimate minimum WJU (`wjuAmountMin`) and USDT (`usdtAmountMin`) to receive. Can use Pair contract's `getReserves` and LP token's `totalSupply` to assist calculations.
3. **Get Pair Address:** Call `factoryContract.methods.getPair(WJU_ADDRESS, USDT_ADDRESS)` to get the WJU-USDT Pair contract address (`pairAddress`).
4. **Approve Router (for LP tokens):**
   * Create Pair contract instance: `const pairContract = new web3.eth.Contract(ABIs.JUV2PAIR_ABI, pairAddress);`
   * Call **Pair contract's** `approve(ROUTER_ADDRESS, liquidityAmount)` function.
   * Wait for approval transaction confirmation.
5. **Prepare Transaction:**
   * Set `deadline`.
   * Construct `removeLiquidityETH` function call with parameters: `token = USDT_ADDRESS`, `liquidity = liquidityAmount`, `amountTokenMin = usdtAmountMin`, `amountETHMin = wjuAmountMin`, `to = USER_ADDRESS`, `deadline`.
6. **Send Transaction:**
   * Sign and send transaction.
   * This transaction does **not** include `value`.
   * Recommend estimating Gas. Upon success, `USER_ADDRESS` will receive WJU and USDT.

**Liquidity Management Considerations:**

* **Impermanent Loss:** Inherent risk of providing liquidity, may occur when token relative prices change.
* **LP Tokens:** LP tokens themselves are ERC20 tokens and can be transferred or used in other DeFi protocols (if supported).
* **First Provider:** If you're the first provider, you'll set the initial exchange rate.

**6. Using the Factory**

The factory contract is primarily used for creating and querying trading pairs.

* **Query Pair Address:**
  * Call `getPair(address tokenA, address tokenB)`.
  * If returns non-zero address, pair exists.
  * If returns `0x000...000`, pair doesn't exist.
* **Create Pair:**
  * If `getPair` confirms pair doesn't exist, can call `createPair(address tokenA, address tokenB)`.
  * `tokenA` and `tokenB` addresses are unordered.
  * Factory will deploy new Pair contract using `create2` and emit `PairCreated` event.
  * Usually, `addLiquidity*` functions will automatically call `createPair` if needed, but manual call ensures existence.
* **Get All Pairs Info:** `allPairsLength()` and `allPairs(uint index)` are available, but iteration may be costly.
* **Deterministic Pair Address Calculation:** Use `pairFor` logic (see Solidity example below) with factory address, sorted token addresses, and `initCodeHash` to calculate pair addresses off-chain.

**Example: Calculate Pair Address (Solidity Helper Library)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

library PairUtil {
    // Calculate pair address for given factory and token pair
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

    // Get expected WJU-USDT pair address
    function getWjuUsdtPairAddress() external pure returns (address) {
        return PairUtil.pairFor(FACTORY, WJU, USDT);
    }
}
```

{% hint style="info" %}
**Best Practices for Interaction (Reference Implementation):**

* **Transaction Sending Wrapper:** Use helper functions to handle nonce retrieval, gas price setting, transaction signing (`web3.eth.accounts.signTransaction`) and sending (`web3.eth.sendSignedTransaction`), including `try...catch` error handling.
* **Gas Estimation:** Use contract method's `.estimateGas()` before sending transactions, and consider adding a small buffer (e.g., 10-20%) to prevent Gas insufficiency due to chain state changes.
* **Slippage Control:** For Swaps and Liquidity operations, always calculate and use `amount*Min` or `amount*Max` parameters.
* **Approve Check:** Before executing operations that spend ERC20 tokens, check existing `allowance` to avoid unnecessary `approve` transactions.
* **Deadline:** Set reasonable `deadline` for time-sensitive operations (Swap, Liquidity).
* **Large Number Handling:** Use `BigInt` (Javascript native) or `web3.utils.toWei/fromWei` and BN.js (built into Web3.js) for token amount calculations.
{% endhint %}

{% hint style="warning" %}
Before deploying to mainnet or handling real assets, please thoroughly test on testnet. Always refer to the official JuChain documentation and block explorer for the most up-to-date, authoritative information and complete ABIs. Manage your private keys securely!
{% endhint %}
