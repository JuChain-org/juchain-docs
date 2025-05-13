# 在 JuScan 浏览器上通过图形化界面验证合约源码

对已部署的智能合约进行源码验证，是确保项目透明度和用户信任的关键步骤。验证后，用户可以直接在 Juscan 区块链浏览器上查看合约的源代码，了解其功能并与之安全交互。本指南将详细介绍如何通过 Juscan（基于开源的 Blockscout 浏览器）提供的图形用户界面 (GUI) 来完成合约验证。这为不熟悉命令行工具（如 Hardhat 或 Foundry）的用户提供了一种直观便捷的验证方式。

***

### 准备工作

在开始验证之前，请确保您已准备好以下信息：

1. **合约地址**: 您已经成功部署到 JuChain 上的智能合约地址。
2. **合约源代码**: 完整的 Solidity (`.sol`) 或 Yul (`.yul`) 或 Vyper (`.vy`) 源代码文件。
3. **编译器版本**: 部署合约时使用的确切 Solidity / Yul / Vyper 编译器版本（例如 `0.8.8`）。
4. **EVM 版本** (如果非默认): 编译时指定的目标 EVM 版本。
5. **优化设置**: 部署时是否启用了编译优化，以及优化运行次数（`runs`，Solidity 默认通常是 `200`）。
6. **构造函数参数 ABI 编码**: 如果合约有构造函数，需要准备部署时传递给构造函数的参数经过 ABI 编码后的十六进制字符串 (以 `0x` 开头)。您可以从 Hardhat/Foundry 部署日志中查找，或使用 ethers.js/web3.js 等工具生成此编码值。
7. **库地址** (如果合约链接了外部库): 所有使用的库的名称和部署地址。
8. **许可证标识符 (SPDX License Identifier)** (可选但推荐): 您合约源代码中指定的 SPDX 许可证标识符 (例如 `MIT`, `GPL-3.0`)。
9. **JSON 输入或元数据文件** (如果选择特定验证方式): 编译器生成的标准 JSON 输入文件或 `metadata.json` 文件。

***

### 图形化验证步骤

1. **访问验证页面**:
   * **推荐**: 直接导航到您部署的合约地址页面。在合约详情页的 "Code" (代码) 选项卡下，查找并点击 "Verify & Publish" (验证并发布) 按钮或链接。
   * 或者，在 JuScan 浏览器的主菜单中寻找 "More" (更多) / "Tools" (工具) -> "Verify Contract" (验证合约) 的选项。
   * 或者，尝试直接访问验证 URL:`https://juscan.io/contract-verification`。
2. **输入合约地址 (Contract Address to Verify)**: 如果您不是从合约页面直接跳转过来的，在此处粘贴您要验证的合约地址。
3. **选择合约许可证 (Contract License)**:
   * 从下拉菜单中选择与您合约源代码中 `SPDX-License-Identifier` (如果声明了) 相匹配的开源许可证。强烈建议指定许可证。
   * Juscan/Blockscout 支持多种常见许可证，包括 MIT, GPL 系列, Apache, Unlicense 等。
   * 如果代码不开源或未指定许可证，选择 "No License (None)"。
4. **选择验证方式 (Verification method / Compiler type)**:
   * 此下拉菜单提供多种验证输入类型。请选择与您的项目和编译输出最匹配的方式：
     * **Solidity (Single file / Flattened source code)**: 适用于**单个 `.sol` 或 `.yul` 文件**。如果您的合约包含 `import` 语句，您需要先将所有依赖的代码"扁平化"(flatten)到这一个文件中。这是最直接的方式，但对于复杂项目可能需要预处理。
     * **Solidity (Standard JSON input)**: **推荐方式之一**。您需要上传由 Solidity 编译器生成的标准 JSON 输入文件。该文件包含了所有源码、编译器设置等信息，验证成功率较高。
     * **Solidity (Multi-part files)**: 适用于包含多个 `.sol` 或 `.yul` 文件（例如，主合约、库、接口）并通过 `import` 相互引用的项目。您需要分别上传所有相关的源文件。
     * **Solidity (Sourcify)**: **推荐方式之一**。通过上传 Sourcify 生成的元数据 (`metadata.json`) 和所有相关的源文件进行验证。这确保了构建的可复现性。
     * **Solidity (Hardhat)** / **Solidity (Foundry)**: 允许直接使用 Hardhat 或 Foundry 项目的编译产物（如 build-info 文件）进行验证。请根据界面提示操作。
     * **Vyper (Contract)**: 适用于单个 Vyper 合约文件。
     * **Vyper (Multi-part files)**: 适用于包含多个 Vyper 文件的项目。
5. **填写详细信息 (根据所选验证方式变化)**:
   * 根据您选择的验证方式，下方会显示不同的输入字段。
   * **通用字段**:
     * **Compiler Version**: 选择与您部署时使用的**完全相同**的编译器版本 (例如 `0.8.17`)。避免选择 Nightly Builds。
     * **EVM Version**: 选择编译时指定的目标 EVM 版本。如果使用编译器默认值，通常选择 `default` 或对应编译器的默认 EVM。
     * **Optimization Enabled**: 如果编译时启用了优化，选择 "Yes" (是)，并填写优化运行次数 **Optimization Runs** (例如 `200`)。如果未启用，选择 "No" (否)。
   * **对于 Solidity (Single file / Flattened)**:
     * **Is Yul contract?**: 如果您上传的是 Yul (`.yul`) 代码，勾选此项。
     * **Enter the Solidity Contract Code**: 将您的完整、扁平化后的 Solidity 或 Yul 源代码粘贴到此文本框中。
     * **Constructor Arguments**: 如果合约有构造函数，在此处粘贴 ABI 编码后的参数字符串 (以 `0x` 开头)。
     * **Contract Libraries**: 如果合约链接了外部库，点击 "Add Contract Library"，并分别填入库的**名称**和已部署的**地址**。
   * **对于 Solidity (Standard JSON input)**:
     * 上传包含标准 JSON 输入的 `.json` 文件。
   * **对于 Solidity (Multi-part files)**:
     * 逐个上传所有的 `.sol` 或 `.yul` 源文件，包括主合约和所有依赖文件。
   * **对于 Solidity (Sourcify)**:
     * 上传 `metadata.json` 文件以及所有相关的源文件。
6. **提交验证**: 仔细核对所有信息后，点击 "Verify and Publish" (验证并发布) 或类似名称的按钮提交验证请求。

***

### 检查验证结果

* **成功**: 如果所有信息都正确且与链上字节码匹配，页面会显示成功消息。在合约地址的详情页面，"Code" (代码) 选项卡旁边将出现一个绿色的勾号 ✅ 标记，表明合约已成功验证。同时会增加 "Read Contract" (读取合约) 和 "Write Contract" (写入合约) 选项卡，允许用户交互。
* **失败**: 如果验证失败，会显示错误信息，例如 "There was an error compiling your contract", "Bytecode does not match", "Compiler version mismatch" 等。请仔细阅读错误提示，它将指明问题所在。

***

### 常见问题与排查 (Troubleshooting)

验证失败时，请按以下步骤检查最常见的问题：

1. **编译器版本不匹配**: 确保选择的编译器版本 (包括修订号 `x.y.z`) 与部署时使用的**完全一致**。
2. **优化设置不匹配**: 确保优化启用状态 (Yes/No) 和优化运行次数 (`runs`) 与部署时完全一致。这是最常见的错误。
3. **EVM 版本不匹配**: 如果编译时指定了非默认的 EVM 版本，确保在此处也正确选择。
4. **源代码不匹配**:
   * 检查提交的源代码（无论是单个文件、多个文件还是 JSON 输入中的代码）是否与部署时的版本**逐字逐句完全一致**，包括空格、注释等。
   * 对于 Single file/Flattened 模式，确保所有 `import` 都已正确内联。
   * 源代码中的 `SPDX-License-Identifier` 应与您在表单中选择的许可证类型匹配。
5. **构造函数参数错误**: 确保提供的 ABI 编码参数字符串 (以 `0x` 开头) 与部署交易中使用的完全一致。编码方式、参数顺序、参数值都必须正确。
6. **库地址错误**: 如果使用了外部库，确保填写的库名称和已部署地址准确无误。
7. **选择了错误的验证方式**: 例如，对多文件合约使用了 Single file 方式而未扁平化。
8. **Yul 合约标识**: 如果验证的是 Yul 合约，检查是否已正确标记。
9. **网络或浏览器问题**: 尝试刷新页面、清除浏览器缓存或使用不同的浏览器重试。
