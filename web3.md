# Web3 开发面试题大全 (100+题)

本指南涵盖了从基础概念到高级开发的 Web3 面试题目，分为以下几个模块：
1. **Web3 基础与区块链原理**
2. **Ethereum 与 EVM 机制**
3. **Solidity 智能合约开发**
4. **智能合约安全**
5. **DeFi (去中心化金融)**
6. **NFT 与 元宇宙**
7. **Layer 2 与 扩容方案**
8. **工具与高级话题**

---

### 一、Web3 基础与区块链原理

1. **什么是 Web3？它与 Web2 有什么区别？**
   - Web1 是“只读”（Read），Web2 是“读写”（Read-Write），Web3 是“读写拥有”（Read-Write-Own）。Web3 基于区块链，强调去中心化、用户掌握数据所有权和无需许可的交互。

2. **什么是区块链？**
   - 区块链是一个去中心化的分布式账本，数据以“区块”形式存储，通过密码学哈希连接成链，不可篡改且公开透明。

3. **解释 PoW (工作量证明) 与 PoS (权益证明) 的区别？**
   - **PoW**：通过算力竞争解决数学难题来记账（挖矿），安全性高但能耗大（如 Bitcoin）。
   - **PoS**：通过质押代币获得记账权（验证），更节能，扩展性通常更好（如 Ethereum 2.0）。

4. **什么是节点（Node）？全节点与轻节点有什么区别？**
   - 节点是运行区块链客户端的计算机。全节点存储完整账本历史，验证所有交易；轻节点只存储区块头，通过 Merkle Proof 验证交易，依赖全节点。

5. **公钥与私钥的作用是什么？**
   - 私钥用于签名交易，证明资产所有权（绝不能泄露）；公钥用于生成地址，供他人转账验证签名。

6. **什么是去中心化钱包（非托管钱包）？**
   - 用户自己掌握私钥/助记词的钱包（如 MetaMask），资产完全由用户控制。相对的托管钱包（如交易所账户）由平台掌握私钥。

7. **什么是哈希函数？区块链中用了什么哈希算法？**
   - 将任意长度输入转换为固定长度输出的函数。比特币用 SHA-256，以太坊用 Keccak-256。

8. **什么是区块（Block）？包含哪些信息？**
   - 包含区块头（父区块哈希、时间戳、Nonce/难度、状态根等）和区块体（交易列表）。

9. **什么是创世区块（Genesis Block）？**
   - 区块链上的第一个区块（高度为 0），通常硬编码在客户端软件中。

10. **什么是默克尔树（Merkle Tree）？有什么作用？**
    - 一种二叉树结构，叶子节点是交易哈希。根哈希（Merkle Root）存入区块头，用于快速验证某笔交易是否在区块中（SPV 验证）。

11. **什么是硬分叉（Hard Fork）与软分叉（Soft Fork）？**
    - **硬分叉**：旧节点无法验证新规则生成的区块，导致链分裂（需强制升级）。
    - **软分叉**：旧节点能认可新规则的区块（向前兼容），通常是收紧规则。

12. **什么是智能合约（Smart Contract）？**
    - 部署在区块链上的自动执行代码，满足条件即执行，无需中介。

13. **什么是 dApp（去中心化应用）？**
    - 后端运行在区块链（智能合约）上，前端通过 Web3.js/Ethers.js 与链交互的应用程序。

14. **什么是 Gas？为什么需要 Gas？**
    - Gas 是执行操作的计算单位。为了防止死循环和滥用网络资源，用户需为计算付费。

15. **什么是 DAO（去中心化自治组织）？**
    - 基于智能合约治理的组织，规则写入代码，成员通过持币投票决策。

---

### 二、Ethereum 与 EVM 机制

16. **什么是以太坊（Ethereum）？**
    - 一个具备图灵完备智能合约功能的开源公共区块链平台。

17. **什么是 EVM（以太坊虚拟机）？**
    - 以太坊的运行环境，是一个基于栈的虚拟机，负责执行智能合约字节码。

18. **Ether 和 Gas 的关系是什么？**
    - Gas 是计算工作量单位，Ether 是支付货币。`Transaction Fee = Gas Used * Gas Price`。

19. **Gwei 和 Wei 是什么？**
    - 1 Ether = 10^18 Wei。1 Gwei = 10^9 Wei。Gas Price 通常用 Gwei 表示。

20. **以太坊账户有哪些类型？**
    - **EOA（外部拥有账户）**：由私钥控制，无代码，可发起交易。
    - **合约账户（Contract Account）**：由代码控制，无私钥，被调用时运行代码。

21. **以太坊中的 Nonce 是什么？**
    - 账户发出的交易计数器，从 0 开始。防止重放攻击和保证交易顺序。

22. **以太坊的三棵树（Trie）是什么？**
    - **状态树（State Trie）**：存储所有账户余额、Nonce、存储等。
    - **交易树（Transaction Trie）**：存储区块内的交易。
    - **收据树（Receipt Trie）**：存储交易执行后的收据（Log、Gas使用量）。

23. **什么是 EIP？列举几个重要的 EIP。**
    - Ethereum Improvement Proposal。
    - EIP-20 (Token 标准), EIP-721 (NFT), EIP-1559 (Gas 费改革), EIP-4337 (账户抽象)。

24. **EIP-1559 改变了什么？**
    - 引入 Base Fee（销毁）和 Priority Fee（给矿工小费），使 Gas 费更可预测，并带来 ETH 通缩压力。

25. **什么是 The Merge（合并）？**
    - 以太坊主网从 PoW 共识切换到 PoS 共识的升级。

26. **EVM 的存储区域有哪些？（Storage, Memory, Stack, Calldata）**
    - **Storage**：永久存储，昂贵。
    - **Memory**：临时存储，函数执行完清除，较便宜。
    - **Stack**：EVM 计算栈，深度 1024，免费但受限。
    - **Calldata**：交易输入数据，只读，较便宜。

27. **什么是 Logs 和 Events？**
    - 智能合约抛出的事件，存储在 Receipts 中。前端可监听，合约内无法读取。Gas 费比 Storage 低。

28. **交易的生命周期是怎样的？**
    - 签名 -> 广播 -> 内存池（Mempool） -> 验证者打包进区块 -> 共识确认 -> 执行并更新状态。

29. **什么是以太坊分片（Sharding）/ Danksharding？**
    - 旨在提高吞吐量，将网络数据可用性分片，降低 Layer 2 数据上传成本（Proto-Danksharding, EIP-4844）。

30. **什么是 Solidity 的 fallback 和 receive 函数？**
    - **receive()**：接收纯 ETH 转账且 msg.data 为空时触发。
    - **fallback()**：调用不存在的函数或 msg.data 不为空且无匹配函数时触发。

---

### 三、Solidity 智能合约开发

31. **Solidity 有哪些数据类型？**
    - 值类型：bool, int/uint, address, bytes1~32, enum.
    - 引用类型：array, struct, mapping, string, bytes.

32. **public, private, internal, external 的区别？**
    - **public**：内外部均可调，自动生成 getter。
    - **private**：仅本合约可见。
    - **internal**：本合约及继承合约可见。
    - **external**：仅外部可调（内部需用 `this.f()`），calldata 传参更省 Gas。

33. **view 和 pure 修饰符的区别？**
    - **view**：读取但不修改状态。
    - **pure**：既不读取也不修改状态。

34. **mapping 和 array 的区别？如何选择？**
    - **mapping**：哈希表，O(1) 访问，无法遍历（除非自行维护索引）。
    - **array**：可遍历，支持 push/pop。
    - 需要遍历用 array，需要快速查找用 mapping。

35. **Solidity 中的 Error Handling 有哪些？**
    - **require()**：条件不满足回滚，退还剩余 Gas（常用）。
    - **revert()**：类似 require，用于复杂逻辑分支。
    - **assert()**：用于检查内部不变性，失败扣光 Gas（Panic）。

36. **如何发送 ETH？transfer vs send vs call**
    - **transfer**：失败 revert，固定 2300 Gas（不推荐）。
    - **send**：失败返回 false，固定 2300 Gas（不推荐）。
    - **call**：`address.call{value: x}("")`，返回 `(bool, bytes)`，Gas 可调（推荐，防重入需注意）。

37. **什么是 delegatecall？**
    - 调用目标合约代码，但在当前合约的上下文（Storage, msg.sender, msg.value）中执行。常用于代理模式。

38. **msg.sender 和 tx.origin 的区别？**
    - **msg.sender**：当前调用的直接发起者（可能是合约）。
    - **tx.origin**：交易的原始发起者（一定是 EOA）。禁止用 tx.origin 做鉴权。

39. **什么是 Library？**
    - 无状态的合约代码库，使用 `using A for B` 挂载。内部函数直接嵌入，外部函数通过 delegatecall 调用。

40. **interface 和 abstract contract 的区别？**
    - **interface**：只能定义函数声明，无实现，无变量。
    - **abstract**：可以包含部分实现和变量，不能直接部署。

41. **什么是变量打包（Variable Packing）？**
    - Solidity 将多个小于 32 字节的变量存储在同一个 32 字节 Slot 中以节省 Gas（如两个 uint128 存一个 Slot）。

42. **immutable 和 constant 的区别？**
    - **constant**：编译时确定的常量。
    - **immutable**：部署时（构造函数中）赋值一次，之后不可变，运行时写入字节码。

43. **Solidity 0.8.0 之后的算术运算有什么变化？**
    - 自动检查溢出（Overflow/Underflow），无需再用 SafeMath。

44. **如何获取当前区块时间？安全吗？**
    - `block.timestamp`。不完全安全，矿工可在一定范围内操纵（15秒规则），不应用作强随机源。

45. **什么是 ABI（Application Binary Interface）？**
    - 定义了如何与合约交互的标准（函数选择器 + 参数编码）。

46. **如何实现一个可升级合约？**
    - 使用代理模式（Proxy Pattern）。用户 -> Proxy（存状态） -> delegatecall -> Logic（存代码）。升级时修改 Proxy 指向的 Logic 地址。

47. **什么是 Create2？**
    - 允许在部署前预测合约地址。`hash(0xFF, sender, salt, bytecode)`。

48. **Solidity 中的修饰器（Modifier）是什么？**
    - 用于在函数执行前后插入逻辑（如权限检查 `onlyOwner`）。`_;` 表示原函数体。

49. **payable 关键字的作用？**
    - 标记函数或地址可以接收 ETH。

50. **Gas 优化有哪些常见技巧？**
    - 变量打包、使用 calldata 代替 memory、避免循环修改状态、使用 unchecked 块（0.8+）、短路计算、使用事件代替存储等。

---

### 四、智能合约安全

51. **什么是重入攻击（Reentrancy Attack）？**
    - 攻击者合约在接收 ETH 的回调（fallback）中再次调用受害者合约，利用受害者未更新的状态重复提款。

52. **如何防止重入攻击？**
    - **检查-生效-交互（Checks-Effects-Interactions）**模式。
    - 使用重入锁（ReentrancyGuard）。

53. **什么是整型溢出（Overflow/Underflow）？**
    - 数值超过类型最大值回绕。Solidity 0.8 前需 SafeMath，0.8 后自带检查。

54. **什么是抢跑（Front-running）/ 三明治攻击？**
    - 矿工或套利机器人在 Mempool 看到你的交易，支付更高 Gas 在你之前插入交易（买入），推高价格后再卖出获利。

55. **什么是 DoS（拒绝服务）攻击？**
    - 通过耗尽 Gas Limit（如遍历大数组）或恶意回退（Revert）导致合约无法正常执行。

56. **依赖时间戳（Timestamp Dependence）有什么风险？**
    - 矿工可微调时间戳，依赖精确时间戳的彩票/随机数可能被操纵。

57. **为什么随机数在链上很难实现？**
    - 区块链是确定性的，所有链上数据（blockhash, timestamp）都可被预测。

58. **如何安全获取随机数？**
    - 使用预言机（如 Chainlink VRF）提供可验证的真随机数。

59. **什么是 tx.origin 钓鱼攻击？**
    - 诱导用户调用恶意合约，恶意合约利用 `tx.origin == user` 通过脆弱的权限检查。应检查 `msg.sender`。

60. **什么是重放攻击（Replay Attack）？**
    - 签名在另一条链或旧合约中被重复使用。防御：签名包含 Nonce 和 ChainID。

61. **什么是闪电贷攻击（Flash Loan Attack）？**
    - 利用巨额资金瞬间操纵价格预言机（特别是单一 DEX 价格），进行套利或清算，同一交易内归还资金。

62. **什么是蜜罐（Honeypot）？**
    - 看似有漏洞或高收益，实则只能进不能出（如重写 transfer 函数阻止卖出）的恶意合约。

63. **权限控制（Access Control）常见漏洞？**
    - 忘记初始化 Owner、修饰器缺失、未校验调用者。

64. **什么是签名重放？**
    - 离线签名未包含足够上下文（如合约地址、ChainID），导致签名在其他地方有效。

65. **常用的审计工具？**
    - Slither (静态分析), MythX, Echidna (模糊测试)。

---

### 五、DeFi (去中心化金融)

66. **什么是 DeFi？**
    - 基于智能合约构建的开放金融系统（借贷、交易、衍生品）。

67. **什么是 AMM（自动做市商）？**
    - 使用数学公式（如 x*y=k）而非订单簿来定价和提供流动性的 DEX 机制。

68. **什么是无常损失（Impermanent Loss）？**
    - 向 AMM 提供流动性时，因代币价格背离导致的资产价值相对于仅持有代币的损失。

69. **什么是流动性矿池（Liquidity Pool）？**
    - 锁定在智能合约中的代币储备，用于支持交易。

70. **什么是流动性挖矿（Yield Farming）？**
    - 用户为协议提供流动性，获得协议代币作为奖励。

71. **CPMM（恒定乘积做市商）公式是什么？**
    - `x * y = k`。Uniswap V2 的核心模型。

72. **Uniswap V3 的核心改进是什么？**
    - **集中流动性（Concentrated Liquidity）**：允许 LP 在特定价格区间提供流动性，提高资金利用率。

73. **什么是借贷协议（如 Aave, Compound）？**
    - 超额抵押借贷。存入资产赚利息，抵押资产借出其他代币。

74. **什么是闪电贷（Flash Loan）？**
    - 无需抵押，必须在同一笔交易中借出并归还资金，否则交易回滚。

75. **稳定币有哪些类型？**
    - **法币抵押**：USDT, USDC (中心化托管)。
    - **加密资产抵押**：DAI (超额抵押 ETH 等)。
    - **算法稳定币**：通过算法调整供应量（风险较高）。

76. **DAI 是如何保持锚定 1 美元的？**
    - 通过超额抵押债务仓位（CDP/Vault）和清算机制维持偿付能力。

77. **什么是 DEX 聚合器（如 1inch）？**
    - 搜索多个 DEX 寻找最优价格路径的工具。

78. **什么是预言机（Oracle）？**
    - 将链下数据（如币价、天气）安全传送到链上的中间件（如 Chainlink）。

79. **什么是 TWAP（时间加权平均价格）？**
    - 防止预言机被瞬间价格操纵的一种定价机制。

80. **DeFi 中的清算（Liquidation）是什么？**
    - 当抵押品价值跌破阈值，协议允许第三方低价买走抵押品还债，保证协议不亏空。

---

### 六、NFT 与 元宇宙

81. **什么是 NFT（非同质化代币）？**
    - 具有唯一标识符的代币，不可互换。

82. **ERC-721 和 ERC-1155 的区别？**
    - **ERC-721**：每个代币独立 ID，标准 NFT。
    - **ERC-1155**：多代币标准，支持同质化和非同质化，支持批量转账，Gas 更低。

83. **NFT 的元数据（Metadata）存哪里？**
    - 通常存在 IPFS 或 Arweave 等去中心化存储中，链上只存 TokenURI。

84. **什么是 IPFS？**
    - 星际文件系统，基于内容寻址的 P2P 分布式文件系统。

85. **NFT 版税（Royalties）是如何实现的？**
    - EIP-2981 标准。但强制执行依赖交易市场（Marketplace）的支持，链上无法完全强制。

86. **什么是灵魂绑定代币（SBT）？**
    - 不可转移的 NFT，用于代表身份、学历、信誉等。

87. **什么是动态 NFT？**
    - 元数据可随外部条件（预言机数据、时间）变化的 NFT。

88. **什么是 NFT 碎片化？**
    - 将高价 NFT 锁仓，发行 ERC-20 代币代表其份额。

89. **GameFi 的 Play-to-Earn 模式是什么？**
    - 玩家通过玩游戏获得代币或 NFT 资产，并可在市场变现。

90. **常见的 NFT 交易市场协议？**
    - OpenSea (Seaport), Blur。

---

### 七、Layer 2 与 扩容方案

91. **区块链的不可能三角（Trilemma）是什么？**
    - 难以同时满足：去中心化、安全性、可扩展性。

92. **什么是 Layer 2？**
    - 构建在主链（Layer 1）之上的协议，处理交易并最终结算到 L1，以降低费用提高速度。

93. **Rollup 的原理是什么？**
    - 将大量交易在链下执行，压缩数据（Calldata）上传至 L1。

94. **Optimistic Rollup 和 ZK Rollup 的区别？**
    - **Optimistic**：假设交易有效，有 7 天挑战期（欺诈证明），兼容 EVM 容易。
    - **ZK**：生成零知识证明（有效性证明）提交上链，即时确认，技术难度大。

95. **什么是欺诈证明（Fraud Proof）？**
    - 在挑战期内，任何人可提交证据证明某笔状态转换是错误的，若验证属实则回滚并惩罚排序器。

96. **什么是侧链（Sidechain）？与 L2 的区别？**
    - 侧链有独立的共识机制和安全性，不依赖主链安全（如 Polygon PoS）。L2 继承主链安全性。

97. **什么是状态通道（State Channels）？**
    - 双方链下多次交互，仅开启和关闭通道时上链（如 Bitcoin 闪电网络）。

98. **什么是 Validium？**
    - 类似 ZK Rollup，但数据存储在链下（Data Availability off-chain），更便宜但安全性稍低。

99. **跨链桥（Bridge）的原理？**
    - **Lock and Mint**：A 链锁定资产，B 链铸造映射资产。
    - **Burn and Mint**：B 链销毁，A 链释放。
    - **流动性互换**：两边资金池原子交换。

100. **什么是 zkEVM？**
     - 支持运行以太坊智能合约的 ZK Rollup，致力于完全兼容 EVM 操作码。

---

### 八、工具与高级话题

101. **Hardhat, Truffle, Foundry 的比较？**
     - **Truffle**：老牌，JS 生态。
     - **Hardhat**：JS/TS 生态，插件丰富，调试方便（console.log），主流选择。
     - **Foundry**：基于 Rust，用 Solidity 写测试，速度极快，Fuzzing 强大。

102. **Ethers.js 和 Web3.js 的区别？**
     - Ethers.js 更轻量，API 更友好，支持 TypeScript，目前更流行。

103. **什么是 The Graph？**
     - 去中心化索引协议，通过编写 Subgraph 用 GraphQL 查询链上复杂数据。

104. **什么是账户抽象（Account Abstraction, ERC-4337）？**
     - 让智能合约账户像 EOA 一样发起交易，支持社交恢复、代付 Gas、批量交易等 Web2 体验。

105. **什么是 MEV（最大可提取价值）？**
     - 矿工/排序器通过重新排序、插入或审查交易来获得的额外利润（如抢跑、套利）。

106. **什么是 Flashbots？**
     - 旨在缓解 MEV 负面影响的工具，允许搜索者将交易包直接发送给矿工，避免在 Mempool 竞价引发 Gas 战。

107. **什么是钻石标准（EIP-2535）？**
     - 解决合约大小限制（24KB）的模块化代理模式，一个代理对应多个逻辑合约（Facets）。

108. **Merkle Proof 的验证代码是怎样的？**
     - 循环哈希当前节点与兄弟节点，最终对比计算出的 Root 与存储的 Root。

109. **ZK-SNARKs 和 ZK-STARKs 的区别？**
     - SNARKs 需要可信设置，证明小，验证快。STARKs 无需可信设置，抗量子，证明较大。

110. **如何签名并验证消息（EIP-191 / EIP-712）？**
     - 链下用私钥对 `hash(message)` 签名，链上用 `ecrecover` 恢复地址进行比对。EIP-712 提供了结构化数据签名，对用户更可读。

### 九、密码学与共识算法进阶
111. **ECDSA（椭圆曲线数字签名算法）的原理？**
     - 基于椭圆曲线离散对数难题。生成 `(r, s, v)` 签名，验证者利用公钥和消息哈希验证签名的有效性。
112. **secp256k1 和 ed25519 的区别？**
     - **secp256k1**：Bitcoin 和 Ethereum 使用的曲线，Koblitz 曲线。
     - **ed25519**：Solana, Near 使用的曲线，生成签名更快，更安全（无侧信道攻击）。
113. **什么是零知识证明（Zero-Knowledge Proof）？**
     - 证明者（Prover）向验证者（Verifier）证明自己知道某个秘密，而不泄露该秘密本身。
114. **PBFT（实用拜占庭容错）算法？**
     - 允许 1/3 的节点作恶（f < n/3），通过三阶段提交（Pre-prepare, Prepare, Commit）达成共识，通信复杂度 O(N^2)，适合联盟链。
115. **什么是 Casper (FFG vs CBC)？**
     - 以太坊的 PoS 共识机制。FFG（Finality Gadget）提供最终性，CBC（Correct-by-Construction）是构建共识的框架。
116. **什么是 DPoS（委托权益证明）？**
     - 持币者投票选出少量（如 21 个）超级节点负责出块（EOS, Tron），速度快但中心化程度高。
117. **PoA（权威证明）是什么？**
     - 基于身份信誉的共识，验证者是经过认证的已知实体（如测试网 Goerli/Sepolia），效率极高。
118. **什么是 Verifiable Delay Function (VDF)？**
     - 可验证延迟函数，用于生成不可预测且不可操纵的随机数（Solana PoH 使用）。
119. **Merkle Patricia Trie (MPT) 的结构？**
     - 结合了 Merkle Tree 和 Patricia Trie（前缀树），用于高效存储和验证以太坊状态（Key-Value 映射）。
120. **Bloom Filter（布隆过滤器）在区块链中的应用？**
     - 用于轻节点快速过滤日志（Logs），判断某笔交易是否可能存在于区块中。

### 十、DeFi 进阶机制
121. **Curve Finance 的 StableSwap 算法特点？**
     - 专为稳定币兑换设计，结合恒定和（x+y=k）和恒定乘积（x*y=k），在价格 1.0 附近提供极低滑点。
122. **Balancer 的加权池（Weighted Pool）？**
     - 允许非 50/50 比例的资产池（如 80/20），支持多达 8 种代币。
123. **GMX 等衍生品交易所的 GLP 机制？**
     - 零滑点交易（基于预言机价格），LP 提供一篮子资产（GLP）作为对手方，以此赚取交易费和亏损。
124. **什么是流动性引导池（LBP）？**
     - 权重随时间变化的池子，用于新币发行（IDO），防止机器人抢跑，实现公平价格发现。
125. **什么是算法稳定币的 Rebase 机制？**
     - 当价格 > 1$ 时增发代币（分给持有者），价格 < 1$ 时通缩（Ampleforth 为代表），旨在改变供应量而非价格。
126. **什么是 veToken 模型（Vote-Escrowed）？**
     - 锁仓代币越久，获得的投票权和收益权重越高（Curve 首创），绑定长期利益。
127. **Compound V3 (Comet) 的改进？**
     - 单一可借贷资产（USDC），多种抵押品。提高资本效率，降低 Gas。
128. **Aave 的 Flash Loan 费用是多少？**
     - 通常是 0.09%。
129. **Uniswap V2 的 `update()` 函数中为何有 `block.timestamp`？**
     - 用于累积 TWAP 价格预言机所需的 `price0CumulativeLast`。
130. **什么是 MEV-Boost？**
     - 以太坊 PoS 下的中间件，将区块构建权（Builder）和提议权（Proposer/Validator）分离（PBS），优化 MEV 分配。

### 十一、基础设施与工具
131. **IPFS 是永久存储吗？**
     - 不是。节点只缓存自己感兴趣的内容。需配合 Filecoin 或 Pinning Service（如 Pinata）保证数据不丢失。
132. **Arweave 与 IPFS 的区别？**
     - Arweave 是基于区块链的永久存储（一次付费，永久存储，基于访问证明 PoA）。
133. **Filecoin 的证明机制？**
     - **PoRep（复制证明）**：证明数据已存储。
     - **PoSt（时空证明）**：证明数据在一段时间内持续存储。
134. **Chainlink 的工作原理？**
     - 去中心化预言机网络（DON），多个节点从不同数据源获取数据，聚合后上链。
135. **The Graph 的 Indexer, Curator, Delegator 角色？**
     - Indexer：运行节点索引数据。
     - Curator：指明哪些 Subgraph 有价值。
     - Delegator：质押代币给 Indexer 赚取收益。
136. **Infura/Alchemy 是什么？**
     - 区块链节点服务提供商（RPC Provider），开发者无需自建节点即可连接区块链。
137. **Gnosis Safe (Safe) 是什么？**
     - 最流行的多签钱包合约，支持模块化扩展。
138. **Snapshot 投票原理？**
     - 链下签名投票（IPFS 存储），Gas 费为 0。结果可由 Oracle 上链执行（Reality.eth）。
139. **WalletConnect 协议？**
     - 钱包与 DApp 之间的通信标准，基于中继服务器转发加密消息。
140. **Tenderly 是什么？**
     - 智能合约监控、调试和模拟平台。

### 十二、其他公链生态
141. **Solana 的核心创新（PoH）？**
     - 历史证明（Proof of History），利用 VDF 生成时间戳序列，节点无需实时通信即可并行处理交易。
142. **Polkadot 的架构（Relay Chain & Parachain）？**
     - 中继链负责共识和安全，平行链负责业务，通过 XCMP 跨链消息传递。
143. **Cosmos 的 IBC 协议？**
     - 区块链间通信协议，允许独立主权链（Zone）通过 Hub 进行互操作。
144. **Avalanche 的共识机制？**
     - 雪崩共识（Avalanche Consensus），基于重复随机采样的亚稳态机制，速度极快。
145. **Move 语言（Aptos/Sui）的特点？**
     - 面向资源的编程，资产作为 Resource 存储，不可复制不可丢弃，安全性高。
146. **EVM 兼容链（BSC, Polygon, Fantom）的优势？**
     - 开发者可用 Solidity/Hardhat 无缝迁移 DApp，用户体验一致（MetaMask）。
147. **Near 的分片技术（Nightshade）？**
     - 将区块分块（Chunk），验证者只验证自己分片的状态，动态调整分片。
148. **Celestia (Modular Blockchain)？**
     - 专注于数据可用性（DA）层，将执行和结算解耦给 Rollup。
149. **Filecoin Virtual Machine (FVM)？**
     - 在 Filecoin 存储网络上引入智能合约，实现可编程存储。
150. **BTC Layer 2 (Lightning Network) 原理？**
     - 支付通道网络，链下哈希时间锁合约（HTLC）实现原子交换。

### 十三、Solidity 进阶与优化
151. **Solidity 中的 `assembly` (Yul) 有什么用？**
     - 直接编写内联汇编（EVM Opcode），绕过 Solidity 限制，极致优化 Gas 或操作内存 slot。
152. **`mload`, `mstore`, `sload`, `sstore` 指令？**
     - mload/mstore：读写内存。
     - sload/sstore：读写存储（最贵）。
153. **EIP-1167 (Minimal Proxy Contract)？**
     - 极简代理合约标准，字节码仅 45 字节，用于低成本克隆合约。
154. **什么是 Gas Golfing？**
     - 像打高尔夫一样追求最低 Gas 消耗的编程技巧（如用 `!= 0` 代替 `> 0`，unchecked i++）。
155. **Solidity 0.8.x 的自定义错误（Custom Error）？**
     - `error InsufficientBalance(); revert InsufficientBalance();` 比字符串 `require(..., "msg")` 更省 Gas。
156. **`delete` 关键字的作用？**
     - 将变量重置为默认值（0, false, 0x0），释放 Storage 会返还 Gas（Gas Refund，现已削弱）。
157. **`block.chainid` 的作用？**
     - 获取当前链 ID，防止跨链重放攻击。
158. **`ecrecover` 的陷阱？**
     - 可能返回 0 地址（签名无效时），必须检查返回值是否非零且等于预期地址。
159. **什么是 Commit-Reveal 模式？**
     - 防止抢跑或隐瞒。第一阶段提交哈希（Commit），第二阶段公开明文（Reveal）并验证哈希。
160. **OpenZeppelin 库中的 `SafeERC20`？**
     - 处理非标准 ERC20 Token（如 transfer 失败不 revert 而是返回 false 或无返回值）的兼容性。

### 十四、安全与漏洞案例
161. **Poly Network 黑客事件原因？**
     - 跨链桥合约的权限管理逻辑漏洞，黑客替换了 Keeper 公钥。
162. **Wormhole 跨链桥漏洞（Solana）？**
     - 签名验证指令被替换（Syscall 欺骗），导致伪造了 12万 ETH 的存款凭证。
163. **Nomad 桥漏洞？**
     - 默克尔树根验证初始化为 0，导致攻击者可构造任意有效证明。
164. **The DAO 事件？**
     - 经典的重入攻击，导致以太坊硬分叉回滚（ETC 的诞生）。
165. **Parity 多签钱包冻结事件？**
     - 库合约（Library）被意外初始化并 kill（销毁），导致依赖该库的所有钱包瘫痪。
166. **什么是 Front-end 攻击（如 BadgerDAO, Curve DNS 劫持）？**
     - 攻击者入侵前端网站或 DNS，将合约地址替换为恶意地址，诱导用户授权（Approve）。
167. **无限授权（Infinite Approval）的风险？**
     - 用户授权 `uint256.max`，若协议被黑，攻击者可转走用户钱包内所有代币。
168. **Rug Pull（跑路）的常见迹象？**
     - 匿名团队、合约未开源、流动性未锁定、拥有者权限过大（可铸币/暂停/提款）。
169. **什么是针对 ERC-777 的重入攻击？**
     - ERC-777 在转账时会触发 `tokensReceived` 钩子函数，若未加防范极易重入。
170. **未初始化的代理合约（Uninitialized Proxy）风险？**
     - 逻辑合约未初始化，攻击者可调用 `initialize` 成为 Owner 并自毁逻辑合约（导致 Proxy 不可用）。

### 十五、开发环境与测试
171. **Hardhat 的 `console.log` 原理？**
     - 在合约中插入特殊静态调用（StaticCall），Hardhat Network 捕获该调用并打印参数。
172. **Foundry (Forge) 的 Fuzzing 测试？**
     - 模糊测试，自动生成大量随机输入调用测试函数，寻找边界情况下的 Panic 或断言失败。
173. **Mainnet Forking（主网分叉测试）？**
     - 将本地测试网状态模拟为主网当前状态（依赖 Alchemy/Infura），方便测试与现有 DeFi 协议的交互。
174. **Gas Reporter 插件？**
     - 统计单元测试中每个函数的 Gas 消耗，生成报告。
175. **Etherscan 代码验证（Verify）？**
     - 上传源码和编译配置，使区块浏览器能显示合约源码，增加透明度。
176. **ABI 编码方式（encode vs encodePacked）？**
     - `abi.encode`：标准填充（32字节），无歧义。
     - `abi.encodePacked`：紧凑编码，节省空间，但可能哈希碰撞（如 `("a", "bc")` 和 `("ab", "c")`）。
177. **Typechain 是什么？**
     - 为 Ethers.js/Web3.js 生成 TypeScript 类型定义，提供智能合约交互的代码补全。
178. **Ganache 是什么？**
     - 本地以太坊区块链模拟器（Truffle 套件），现多被 Hardhat Network 取代。
179. **Slither 静态分析能查出什么？**
     - 未使用的变量、重入漏洞、Tx.origin 使用、未检查的返回值等。
180. **MythX？**
     - 综合性安全分析工具（云端），包含静态、动态和模糊测试。

### 十六、Web3 登录与用户体验
181. **SIWE (Sign-In with Ethereum)？**
     - EIP-4361 标准，使用以太坊账户登录 Web2 应用，验证签名确认身份。
182. **ENS (Ethereum Name Service)？**
     - 将复杂地址（0x...）解析为可读域名（alice.eth），基于 ERC-721。
183. **Lens Protocol？**
     - 去中心化社交图谱协议（SocialFi），用户拥有自己的 Profile NFT 和数据。
184. **Meta-Transaction (元交易)？**
     - 用户签名意图，Relayer（中继者）代付 Gas 上链，用户仅需支付 Token。
185. **Gasless 交易如何实现？**
     - 通过元交易或账户抽象（Paymaster），由项目方补贴 Gas。
186. **IPNS (InterPlanetary Name System)？**
     - IPFS 内容哈希是变的，IPNS 提供基于 PeerID 的可变指针（指向最新内容）。
187. **Ceramic Network？**
     - 去中心化数据流网络，适合存储动态、可变的用户数据（如社交帖子）。
188. **Lit Protocol？**
     - 去中心化访问控制（基于加密），拥有特定 NFT 才能解密/访问内容。
189. **Deep Link 在移动端钱包的应用？**
     - 唤起 MetaMask/Trust Wallet App 进行签名或支付。
190. **Web3 浏览器的 `window.ethereum`？**
     - EIP-1193 标准，注入到页面的 Provider 对象，用于请求账户和签名。

### 十七、趋势与未来
191. **ZK-EVM 的不同类型（Vitalik 分类）？**
     - Type 1 (完全等效), Type 2 (EVM 等效), Type 3 (几乎等效), Type 4 (高级语言等效)。
192. **Modular Blockchain（模块化区块链）架构？**
     - Execution（执行）, Settlement（结算）, Consensus（共识）, Data Availability（数据可用性）四层解耦。
193. **Restaking (如 EigenLayer)？**
     - 复用 ETH 质押的安全性来保护其他协议（如预言机、桥），提高资本效率。
194. **Parallel EVM (并行 EVM)？**
     - 像 Solana 一样并行执行互不冲突的 EVM 交易（如 Monad, Sei）。
195. **SBT 在 DID（去中心化身份）中的作用？**
     - 作为不可转移的凭证（学历、KYC），构建链上声誉系统。
196. **DePIN (Decentralized Physical Infrastructure Networks)？**
     - 去中心化物理基础设施网络（如 Helium, Filecoin, Render），用 Token 激励硬件部署。
197. **RWA (Real World Assets)？**
     - 现实资产上链（国债、房地产、股票），DeFi 引入传统金融收益。
198. **Fully On-Chain Games (全链游戏)？**
     - 游戏逻辑、状态、资产全部在链上，真正去中心化和可组合。
199. **ERC-6551 (Token Bound Accounts)？**
     - 给每个 NFT 绑定一个智能合约钱包，使 NFT 可以拥有资产和交互能力。
200. **Web3 的 Mass Adoption（大规模采用）瓶颈？**
     - 用户体验（助记词、Gas）、监管合规、扩展性、安全性。

