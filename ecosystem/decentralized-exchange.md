# Decentralized Exchange

JuChain offers a decentralized exchange (DEX) functionality through the `JUV2Router02` (Router), `JUV2Factory` (Factory), and `JUV2Pair` (Pair) contracts, enabling token swaps and liquidity provision. This document details how to perform token swaps and manage liquidity on the JuChain network, applicable to tokens like WJU (Wrapped JU) and USDT.

#### Key Highlights

* **Supported Tokens**:
  * WJU (Wrapped JU): `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3`
  * USDT: `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02`
* **Core Contracts**:
  * **Router**: `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0` (Solidity 0.6.6)
  * **Factory**: `0x66682281BdfeC17fCBcae0480C77edFb0c489339` (Solidity 0.5.16)
* **Features**:
  * **Swap**: Exchange tokens, including WJU and USDT.
  * **Liquidity**: Add or remove liquidity for trading pairs.

***

### Testnet Setup

To operate on the JuChain Testnet (or other compatible networks), configure the following:

* **RPC**: `https://testnet-rpc.juchain.org` (verify with official sources).
* **Explorer**: `https://explorer-testnet.juchain.org`.
* **Chain ID**: 202599.

***

### Contract Addresses

| **Contract** | **Address**                                  | **Solidity Version** |
| ------------ | -------------------------------------------- | -------------------- |
| WJU          | `0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3` | 0.4.26               |
| USDT         | `0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02` | 0.8.8                |
| Factory      | `0x66682281BdfeC17fCBcae0480C77edFb0c489339` | 0.5.16               |
| Router       | `0x6A647E09193a130b0CccBF26A1CF442491bDeCc0` | 0.6.6                |

***

### 1. Swap Functionality

#### Overview

The swap feature, powered by `JUV2Router02`, supports:

* Swapping a precise amount of input tokens for target tokens.
* Swapping tokens for a precise amount of WJU (ETH).
* Handling tokens with transfer fees.

#### Router Contract Methods

Key swap methods include:

1. **`swapExactTokensForTokens`**
   * **Purpose**: Swap an exact amount of input tokens for target tokens.
   * **Parameters**:
     * `amountIn`: Input token amount.
     * `amountOutMin`: Minimum output token amount (slippage protection).
     * `path`: Swap path (e.g., `[WJU, USDT]`).
     * `to`: Recipient address.
     * `deadline`: Transaction deadline (Unix timestamp).
   * **State**: `nonpayable`.
2. **`swapExactETHForTokens`**
   * **Purpose**: Swap an exact amount of WJU for target tokens.
   * **Parameters**:
     * `amountOutMin`: Minimum output token amount.
     * `path`: Swap path (e.g., `[WJU, USDT]`).
     * `to`: Recipient address.
     * `deadline`: Transaction deadline.
   * **State**: `payable`.
3. **`swapExactTokensForETH`**
   * **Purpose**: Swap an exact amount of tokens for WJU.
   * **Parameters**:
     * `amountIn`: Input token amount.
     * `amountOutMin`: Minimum WJU amount.
     * `path`: Swap path (e.g., `[USDT, WJU]`).
     * `to`: Recipient address.
     * `deadline`: Transaction deadline.
   * **State**: `nonpayable`.
4. **`swapExactTokensForTokensSupportingFeeOnTransferTokens`**
   * **Purpose**: Swap an exact amount of tokens with transfer fees for target tokens.
   * **Parameters**: Same as `swapExactTokensForTokens`.
   * **State**: `nonpayable`.

#### Usage Steps

**1. Preparation**

* **Approve Tokens**: Call the token’s `approve` method to authorize the Router to transfer tokens.
* **Check Pair**: Ensure the Factory has the target trading pair (e.g., WJU-USDT) with sufficient liquidity.

**2. Example Code**

**Swap WJU for USDT (Web3.js)**

```javascript
const { Web3 } = require('web3');
const web3 = new Web3('https://testnet-rpc.juchain.org');

const routerAddress = '0x6A647E09193a130b0CccBF26A1CF442491bDeCc0';
const wjuAddress = '0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3';
const usdtAddress = '0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02';
const routerABI = [/* Router ABI */];
const wjuABI = [/* WJU ABI */];

const router = new web3.eth.Contract(routerABI, routerAddress);
const wju = new web3.eth.Contract(wjuABI, wjuAddress);

async function swapWJUForUSDT(amountIn, amountOutMin, account, privateKey) {
  const path = [wjuAddress, usdtAddress];
  const deadline = Math.floor(Date.now() / 1000) + 60 * 20;

  // Approve Router to use WJU
  await wju.methods.approve(routerAddress, amountIn).send({ from: account });

  const tx = router.methods.swapExactTokensForTokens(
    amountIn,
    amountOutMin,
    path,
    account,
    deadline
  );
  const gas = await tx.estimateGas({ from: account });
  const receipt = await tx.send({ from: account, gas });

  console.log(`Swap successful, Tx Hash: ${receipt.transactionHash}`);
}

// Example: Swap 1 WJU for USDT
swapWJUForUSDT(web3.utils.toWei('1', 'ether'), web3.utils.toWei('0.9', 'ether'), '0xYourAddress', '0xYourPrivateKey');
```

**Swap USDT for WJU (Solidity)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.6;

interface IJUV2Router02 {
    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)
        external returns (uint[] memory amounts);
}

interface IERC20 {
    function approve(address spender, uint value) external returns (bool);
}

contract SwapExample {
    address constant ROUTER = 0x6A647E09193a130b0CccBF26A1CF442491bDeCc0;
    address constant USDT = 0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02;
    address constant WJU = 0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3;

    IJUV2Router02 public router = IJUV2Router02(ROUTER);

    function swapUSDTForWJU(uint amountIn, uint amountOutMin) external {
        IERC20(USDT).approve(ROUTER, amountIn);
        address[] memory path = new address[](2);
        path[0] = USDT;
        path[1] = WJU;
        router.swapExactTokensForETH(amountIn, amountOutMin, path, msg.sender, block.timestamp + 1200);
    }
}
```

#### Notes

* **Slippage Protection**: Set `amountOutMin` to guard against price volatility.
* **Pair Liquidity**: Insufficient liquidity may cause swaps to fail.
* **Gas Optimization**: Estimate gas and add a buffer.

***

### 2. Liquidity Addition and Removal

#### Overview

The liquidity feature lets users add or remove liquidity for pairs like WJU-USDT, supporting swaps. Adding liquidity mints LP tokens (Liquidity Provider Tokens), while removing it returns the underlying tokens.

#### Router Contract Methods

1. **`addLiquidity`**
   * **Purpose**: Add liquidity to a token pair.
   * **Parameters**:
     * `tokenA`: Token A address.
     * `tokenB`: Token B address.
     * `amountADesired`: Desired Token A amount.
     * `amountBDesired`: Desired Token B amount.
     * `amountAMin`: Minimum Token A amount.
     * `amountBMin`: Minimum Token B amount.
     * `to`: LP token recipient.
     * `deadline`: Transaction deadline.
   * **State**: `nonpayable`.
   * **Returns**: `(amountA, amountB, liquidity)`.
2. **`addLiquidityETH`**
   * **Purpose**: Add liquidity using WJU and a token.
   * **Parameters**:
     * `token`: Token address.
     * `amountTokenDesired`: Desired token amount.
     * `amountTokenMin`: Minimum token amount.
     * `amountETHMin`: Minimum WJU amount.
     * `to`: LP token recipient.
     * `deadline`: Transaction deadline.
   * **State**: `payable`.
   * **Returns**: `(amountToken, amountETH, liquidity)`.
3. **`removeLiquidity`**
   * **Purpose**: Remove liquidity and retrieve tokens.
   * **Parameters**:
     * `tokenA`: Token A address.
     * `tokenB`: Token B address.
     * `liquidity`: LP token amount to remove.
     * `amountAMin`: Minimum Token A amount.
     * `amountBMin`: Minimum Token B amount.
     * `to`: Recipient address.
     * `deadline`: Transaction deadline.
   * **State**: `nonpayable`.
   * **Returns**: `(amountA, amountB)`.
4. **`removeLiquidityETH`**
   * **Purpose**: Remove liquidity from a WJU-token pair.
   * **Parameters**:
     * `token`: Token address.
     * `liquidity`: LP token amount to remove.
     * `amountTokenMin`: Minimum token amount.
     * `amountETHMin`: Minimum WJU amount.
     * `to`: Recipient address.
     * `deadline`: Transaction deadline.
   * **State**: `nonpayable`.
   * **Returns**: `(amountToken, amountETH)`.

#### Usage Steps

**1. Preparation**

* **Approve Tokens**: Approve the Router for both tokens and LP tokens.
* **Create Pair**: If the pair doesn’t exist, use Factory’s `createPair` to initialize it.

**2. Example Code**

**Add WJU-USDT Liquidity (Web3.js)**

```javascript
async function addLiquidityWJUUSDT(amountWJU, amountUSDT, account, privateKey) {
  const usdt = new web3.eth.Contract(usdtABI, usdtAddress);
  const deadline = Math.floor(Date.now() / 1000) + 60 * 20;

  // Approve Router to use USDT
  await usdt.methods.approve(routerAddress, amountUSDT).send({ from: account });

  const tx = router.methods.addLiquidityETH(
    usdtAddress,
    amountUSDT,
    amountUSDT, // amountTokenMin
    amountWJU,  // amountETHMin
    account,
    deadline
  );
  const gas = await tx.estimateGas({ from: account, value: amountWJU });
  const receipt = await tx.send({ from: account, gas, value: amountWJU });

  console.log(`Liquidity added, Tx Hash: ${receipt.transactionHash}`);
}

// Example: Add 1 WJU and 1 USDT
addLiquidityWJUUSDT(web3.utils.toWei('1', 'ether'), web3.utils.toWei('1', 'ether'), '0xYourAddress', '0xYourPrivateKey');
```

**Remove WJU-USDT Liquidity (Solidity)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.6;

interface IJUV2Router02 {
    function removeLiquidityETH(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external returns (uint amountToken, uint amountETH);
}

interface IJUV2Pair {
    function approve(address spender, uint value) external returns (bool);
}

contract LiquidityExample {
    address constant ROUTER = 0x6A647E09193a130b0CccBF26A1CF442491bDeCc0;
    address constant USDT = 0xf173cD2DD28f94F6b7F6B0817E498fe842bC5D02;
    address constant WJU = 0x2c67A8Ee92C5dD55b1D133631a32451e123Be1d3;
    IJUV2Router02 public router = IJUV2Router02(ROUTER);

    function removeLiquidity(uint liquidity, uint amountUSDTMin, uint amountWJUMin) external {
        address pairAddress = pairFor(USDT, WJU); // Calculate pair address
        IJUV2Pair(pairAddress).approve(ROUTER, liquidity);

        router.removeLiquidityETH(
            USDT,
            liquidity,
            amountUSDTMin,
            amountWJUMin,
            msg.sender,
            block.timestamp + 1200
        );
    }

    // Simplified pairFor (use JUV2Library in practice)
    function pairFor(address tokenA, address tokenB) internal pure returns (address pair) {
        pair = address(uint(keccak256(abi.encodePacked(
            hex'ff',
            0x66682281BdfeC17fCBcae0480C77edFb0c489339,
            keccak256(abi.encodePacked(tokenA < tokenB ? tokenA : tokenB, tokenA < tokenB ? tokenB : tokenA)),
            hex'2d5d6553271f0bbe36b13a3628f44898e95763f6f3692c2de666389cb179309b'
        ))));
    }
}
```

#### Notes

* **Ratio Matching**: Liquidity must match the pair’s current reserve ratio.
* **LP Tokens**: Receive LP tokens upon adding liquidity; approve Router to remove them.
* **Minimum Amounts**: Set `amountAMin` and `amountBMin` to mitigate slippage.
* **First Addition**: If a pair lacks liquidity, the initial ratio sets the precedent.

***

### ABI Data

Partial key ABI for the Router:

```json
[
  {
    "inputs": [
      {"internalType": "address", "name": "token", "type": "address"},
      {"internalType": "uint256", "name": "amountTokenDesired", "type": "uint256"},
      {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"},
      {"internalType": "address", "name": "to", "type": "address"},
      {"internalType": "uint256", "name": "deadline", "type": "uint256"}
    ],
    "name": "addLiquidityETH",
    "outputs": [
      {"internalType": "uint256", "name": "amountToken", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETH", "type": "uint256"},
      {"internalType": "uint256", "name": "liquidity", "type": "uint256"}
    ],
    "stateMutability": "payable",
    "type": "function"
  },
  {
    "inputs": [
      {"internalType": "address", "name": "token", "type": "address"},
      {"internalType": "uint256", "name": "liquidity", "type": "uint256"},
      {"internalType": "uint256", "name": "amountTokenMin", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETHMin", "type": "uint256"},
      {"internalType": "address", "name": "to", "type": "address"},
      {"internalType": "uint256", "name": "deadline", "type": "uint256"}
    ],
    "name": "removeLiquidityETH",
    "outputs": [
      {"internalType": "uint256", "name": "amountToken", "type": "uint256"},
      {"internalType": "uint256", "name": "amountETH", "type": "uint256"}
    ],
    "stateMutability": "nonpayable",
    "type": "function"
  }
]
```
