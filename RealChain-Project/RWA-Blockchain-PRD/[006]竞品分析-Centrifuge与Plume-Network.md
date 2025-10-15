# USDable Chain 竞品分析 - Centrifuge 与 Plume Network

**文档编号**: [006]  
**文档版本**: v1.0  
**创建时间**: 2025-10-15 16:25 CST  
**文档类型**: 竞品分析报告  
**所属项目**: USDable Chain 全栈公链开发

---

## 目录

-   [1. 分析目的](#1-分析目的)
-   [2. Centrifuge 分析](#2-centrifuge-分析)
-   [3. Plume Network 分析](#3-plume-network-分析)
-   [4. 对比总结](#4-对比总结)
-   [5. 对 USDable Chain 的启示](#5-对-usdable-chain-的启示)

---

## 1. 分析目的

本文档分析 Centrifuge 和 Plume Network 两个 RWA 公链项目,从以下四个维度进行研究:

1. **核心业务**: 他们的产品核心业务是什么
2. **公链功能**: 他们的公链提供哪些功能
3. **业务耦合**: 公链功能如何支撑核心业务
4. **学习借鉴**: 值得 USDable Chain 学习的地方

**分析方法**: 仅提出需求和功能设计,不涉及具体技术实现细节。

---

## 2. Centrifuge 分析

### 2.1 核心业务

#### 2.1.1 主营业务

Centrifuge 专注于 **结构化信贷市场的代币化**,核心业务包括:

1. **信贷资产代币化**

    - 将房地产贷款、应收账款、发票等信贷资产代币化
    - 为资产发行方提供融资渠道
    - 为投资者提供稳定收益的 RWA 投资机会

2. **Tinlake 资产池管理**

    - 创建和管理资产池 (Asset Pools)
    - 支持高级/次级份额 (Senior/Junior Tranches)
    - 自动化利息分配和本金偿还

3. **Centrifuge Prime (机构级服务)**
    - 为机构投资者提供合规的 RWA 投资通道
    - 支持 KYC/AML 合规流程
    - 提供链上信用评分和风险评估

#### 2.1.2 目标用户

-   **资产发行方**: 中小企业、房地产开发商、供应链金融公司
-   **投资者**: 机构投资者、DeFi 协议、个人投资者
-   **服务提供商**: 资产评估机构、托管机构、法律服务商

#### 2.1.3 业务规模

-   **TVL**: 超过 $5 亿美元 (历史峰值)
-   **资产池数量**: 30+ 个活跃资产池
-   **合作机构**: MakerDAO、Aave、BlockTower Capital 等

### 2.2 公链功能

#### 2.2.1 公链如何包含这些功能

Centrifuge Chain 是基于 **Substrate FRAME** 框架构建的 **Polkadot Parachain**,通过 **30+ 自定义 Pallets (模块)** 实现 RWA 全生命周期管理。

**Pallet 是什么?**

-   Pallet 是 Substrate 框架中的功能模块,类似于智能合约,但更底层、更高效
-   每个 Pallet 都是用 Rust 编写的链上逻辑,直接编译到区块链运行时 (Runtime)
-   Pallets 可以相互调用,形成完整的业务逻辑

**Centrifuge 的 30+ Pallets 分类**:

-   **RWA 专用模块**: 10+ pallets (pool-system, loans, pool-registry, pool-fees 等)
-   **合规模块**: 5+ pallets (permissions, transfer-allowlist, restricted-tokens 等)
-   **跨链模块**: 5+ pallets (liquidity-pools, axelar-router 等)
-   **数据和预言机模块**: 5+ pallets (oracle-collection, oracle-feed, anchors 等)
-   **治理和工具模块**: 5+ pallets (governance, treasury 等)

**参考资料**: [Centrifuge Pallets 官方文档](https://reference.centrifuge.io/)

#### 2.2.2 核心功能模块详解

**1. 资产存证模块 (anchors Pallet)**

-   **需求**: 记录资产的所有权、估值、合同文件哈希
-   **公链实现方式**:
    -   通过 `anchors` Pallet 在链上存储文档哈希 (POD 网络)
    -   每个资产对应一个唯一的文档哈希,存储在链上
    -   支持文档版本控制,每次更新生成新的哈希
-   **功能**: 为每个资产生成唯一 NFT,存储元数据
-   **特点**: 不可篡改、可审计、支持版本控制
-   **官方文档**: https://reference.centrifuge.io/pallet_anchors/

**2. 资产池管理模块 (pool-system Pallet)**

-   **需求**: 创建和管理多层级资产池
-   **公链实现方式**:
    -   通过 `pool-system` Pallet 管理投资池的创建、投资、赎回
    -   支持多层级 Tranches (高级/次级份额)
    -   自动化利息分配和本金偿还逻辑内置于 Pallet
-   **功能**: 支持高级/次级份额、自动化利息分配、流动性管理
-   **特点**: 灵活的池参数配置、实时净值计算
-   **官方文档**: https://reference.centrifuge.io/pallet_pool_system/

**3. 贷款管理模块 (loans Pallet)**

-   **需求**: 管理资产抵押和借款
-   **公链实现方式**:
    -   通过 `loans` Pallet 管理贷款的创建、借款、还款
    -   锁定抵押品 NFT 到池中,记录贷款价值和未偿债务
    -   自动计算利息和还款金额
-   **功能**: 抵押品管理、借款、还款、利息计算
-   **特点**: 链上自动化执行,无需人工干预
-   **官方文档**: https://reference.centrifuge.io/pallet_loans/

**4. 合规模块 (permissions + transfer-allowlist Pallets)**

-   **需求**: 满足全球监管要求
-   **公链实现方式**:
    -   通过 `permissions` Pallet 实现基于角色的权限管理
    -   通过 `transfer-allowlist` Pallet 管理转账白名单
    -   通过 `restricted-tokens` Pallet 限制代币转账
-   **功能**: KYC/AML 白名单、地域限制、投资者资格验证
-   **特点**: 可插拔的合规规则引擎,支持自定义规则
-   **官方文档**:
    -   https://reference.centrifuge.io/pallet_permissions/
    -   https://reference.centrifuge.io/pallet_transfer_allowlist/

**5. 预言机模块 (oracle-collection + oracle-feed Pallets)**

-   **需求**: 获取链下资产数据
-   **公链实现方式**:
    -   通过 `oracle-collection` Pallet 收集和聚合预言机数据
    -   通过 `oracle-feed` Pallet 提供预言机数据
    -   支持多个预言机源,自动聚合和验证
-   **功能**: 资产估值更新、利率数据、违约事件通知
-   **特点**: 多源数据聚合、数据验证机制
-   **官方文档**:
    -   https://reference.centrifuge.io/pallet_oracle_collection/
    -   https://reference.centrifuge.io/pallet_oracle_feed/

**6. 跨链桥模块 (liquidity-pools + axelar-router Pallets)**

-   **需求**: 与以太坊等主流链互操作
-   **公链实现方式**:
    -   通过 `liquidity-pools` Pallet 管理跨链流动性池
    -   通过 `axelar-router` Pallet 实现 Axelar 跨链路由
    -   通过 Polkadot XCM 协议与其他 Parachains 通信
-   **功能**: 资产跨链转移、流动性共享
-   **特点**: 基于 Polkadot XCM 协议 + Axelar 跨链桥
-   **官方文档**:
    -   https://reference.centrifuge.io/pallet_liquidity_pools/
    -   https://reference.centrifuge.io/pallet_axelar_router/

**7. 治理模块 (governance Pallet)**

-   **需求**: 社区驱动的协议升级
-   **公链实现方式**:
    -   通过 Substrate 内置的 `governance` Pallet 实现链上治理
    -   CFG 代币持有者可以创建提案、投票、执行
    -   支持无分叉升级 (Forkless Upgrade)
-   **功能**: 提案创建、投票、执行
-   **特点**: CFG 代币持有者参与治理,链上自动执行

#### 2.2.3 Hub-and-Spoke 架构

Centrifuge 采用 **Hub-and-Spoke 架构**,实现跨链资产管理:

**Hub (中心) - Centrifuge Chain**:

-   职责: 池管理、权限控制、资产记账、跨链消息协调
-   核心 Pallets: pool-system, loans, permissions, oracle-collection
-   功能: 创建池、管理 Share Classes、控制投资权限、处理资产

**Spoke (辐条) - EVM Chains (Ethereum, Base, Arbitrum)**:

-   职责: 投资者交互、流动性提供、代币发行
-   核心合约: Vault (ERC-7540), ShareToken (ERC-20), Escrow
-   功能: 部署 Vaults、发行 Share Tokens、处理投资/赎回请求

**跨链消息传递**:

-   Hub 通过 `liquidity-pools` Pallet 向 Spoke 链发送消息
-   Spoke 通过 Vault 接口向 Hub 发送投资/赎回请求
-   使用 Axelar 跨链桥实现消息传递

**参考资料**: [Centrifuge Protocol Overview](https://docs.centrifuge.io/developer/protocol/overview/)

#### 2.2.4 技术特性

-   **共识机制**: NPoS (Nominated Proof of Stake),继承 Polkadot 共享安全
-   **区块时间**: 6 秒
-   **最终性**: 即时最终性 (Instant Finality,GRANDPA)
-   **Gas 费用**: 极低 (约 $0.001/交易)
-   **开发语言**: Rust (Pallets) + Solidity (EVM 合约)
-   **验证者**: 约 300 个 Polkadot 验证者保护整个网络

### 2.3 公链与业务的耦合

#### 2.3.1 耦合方式

Centrifuge 的公链功能与业务深度耦合:

1. **资产代币化流程**

    ```
    资产发行方 → 资产存证模块 (生成 NFT)
               → 资产池管理模块 (创建资产池)
               → 合规模块 (KYC 验证)
               → 发行代币 (ERC-20 份额代币)
    ```

2. **投资者购买流程**

    ```
    投资者 → 合规模块 (KYC 验证)
          → 资产池管理模块 (购买份额代币)
          → 自动分红 (利息自动分配到钱包)
    ```

3. **资产估值更新流程**
    ```
    预言机模块 → 获取链下估值数据
               → 资产存证模块 (更新 NFT 元数据)
               → 资产池管理模块 (重新计算净值)
    ```

#### 2.3.2 耦合优势

-   **业务逻辑链上化**: 所有核心业务逻辑在链上执行,透明可审计
-   **自动化执行**: 利息分配、本金偿还自动化,无需人工干预
-   **合规内置**: 合规规则内置于链上,无法绕过
-   **跨链互操作**: 通过 Polkadot 生态实现跨链资产流动

### 2.4 值得学习的地方

#### 2.4.1 产品设计

1. **多层级资产池设计**

    - 需求: 支持不同风险偏好的投资者
    - 方案: 高级份额 (低风险低收益) + 次级份额 (高风险高收益)
    - 启示: USDable Chain 可以借鉴这种分层设计,满足不同投资者需求

2. **实时净值计算**

    - 需求: 投资者需要实时了解资产池价值
    - 方案: 链上自动计算净值,考虑资产估值、利息、违约等因素
    - 启示: USDable Chain 需要提供实时的资产估值和收益计算

3. **灵活的合规规则引擎**
    - 需求: 不同资产类型和地域有不同合规要求
    - 方案: 可插拔的合规规则,支持自定义 KYC/AML 流程
    - 启示: USDable Chain 应设计灵活的合规框架,适应不同监管环境

#### 2.4.2 业务模式

1. **机构级服务 (Centrifuge Prime)**

    - 需求: 机构投资者需要更高的合规标准和服务质量
    - 方案: 提供专属的机构级服务,包括托管、审计、法律支持
    - 启示: USDable Chain 可以考虑为机构用户提供差异化服务

2. **与 DeFi 协议集成**

    - 需求: 扩大资产流动性和用户基础
    - 方案: 与 MakerDAO、Aave 等 DeFi 协议集成,将 RWA 作为抵押品
    - 启示: USDable Chain 应积极与主流 DeFi 协议合作

3. **社区驱动的治理**
    - 需求: 协议升级需要社区共识
    - 方案: CFG 代币持有者参与治理,决定协议参数和升级方向
    - 启示: USDable Chain 的 veABLE 治理模型可以借鉴这种社区驱动方式

---

## 3. Plume Network 分析

### 3.1 核心业务

#### 3.1.1 主营业务

Plume Network 定位为 **全栈 RWA 解决方案**,核心业务包括:

1. **Arc 代币化引擎**

    - 为任何实物资产提供一键代币化服务
    - 支持房地产、艺术品、碳信用、私募股权等多种资产类型
    - 提供端到端的代币化工具链 (KYC、合规、发行、交易)

2. **Nest 质押协议**

    - 为 RWA 代币提供质押和收益生成服务
    - 支持跨资产质押 (例如质押房地产代币获得稳定币收益)
    - 提供流动性挖矿和收益聚合

3. **RWA 市场和交易**
    - 提供 RWA 代币的二级市场交易
    - 支持限价单、市价单、拍卖等多种交易方式
    - 提供流动性池和做市商服务

#### 3.1.2 目标用户

-   **资产发行方**: 房地产开发商、艺术品收藏家、碳信用项目方
-   **投资者**: 零售投资者、机构投资者、DeFi 用户
-   **开发者**: 构建 RWA DApp 的开发者

#### 3.1.3 业务规模

-   **生态项目**: 180+ 个协议
-   **资产规模**: $40 亿美元 (预计代币化规模)
-   **主网上线**: 2025 年 6 月 5 日

### 3.2 公链功能

#### 3.2.1 公链如何包含这些功能

Plume Network 是一个 **EVM 兼容的 Layer 1 区块链**,专为 RWA 设计。与 Centrifuge 的 Pallet 模式不同,Plume 通过 **原生集成的智能合约** 和 **定制化的共识机制** 实现 RWA 功能。

**Plume 的技术架构**:

-   **基础层**: EVM 兼容的 Layer 1 区块链 (Solidity 智能合约)
-   **共识层**: Proof of Representation (PoR) + PoS 混合共识
-   **应用层**: Arc 代币化引擎 + Nest 质押协议 + RWA 市场 (原生集成)
-   **数据层**: Celestia 数据可用性层 (DA Layer)

**与 Centrifuge 的对比**:

-   Centrifuge: Substrate Pallets (Rust) → 链上逻辑直接编译到 Runtime
-   Plume: Solidity 智能合约 (EVM) → 链上逻辑通过智能合约执行

**参考资料**: [Plume Network 官方文档](https://docs.plumenetwork.xyz/)

#### 3.2.2 核心功能模块详解

**1. Arc 代币化引擎 (原生集成)**

-   **需求**: 简化资产代币化流程
-   **公链实现方式**:
    -   Arc 是 Plume 原生集成的代币化引擎,通过一组智能合约实现
    -   提供可视化界面,用户无需编写代码即可代币化资产
    -   自动化合规检查 (与 Stobox 等服务商集成)
    -   元数据管理 (存储在链上或 IPFS)
-   **功能**: 一键创建 RWA 代币、自动化合规检查、元数据管理
-   **特点**: 全栈解决方案,无需编写智能合约
-   **官方文档**: [Arc 代币化引擎](https://plume.org/blog/plume-arc)

**2. Proof of Representation (PoR) 共识**

-   **需求**: 验证链下资产的真实性
-   **公链实现方式**:
    -   PoR 是 Plume 定制化的共识机制,专为 RWA 设计
    -   数据提供者网络 (Data Provider Network) 验证资产数据
    -   使用 zkTLS 证明 (零知识传输层安全) 验证链下数据
    -   多源数据聚合,确保数据真实性
    -   数据提供者需要质押 PLUME 代币,恶意行为会被惩罚
-   **功能**: 数据提供者网络验证资产数据、zkTLS 证明、多源数据聚合
-   **特点**: 专为 RWA 设计的共识机制,确保链下资产数据的真实性
-   **官方文档**: [Proof of Representation](https://docs.plumenetwork.xyz/plume/architecture/consensus)

**3. 合规框架 (Compliance Framework)**

-   **需求**: 满足全球监管要求
-   **公链实现方式**:
    -   通过智能合约实现 KYC/AML 白名单管理
    -   与 Stobox 等合规服务商集成,提供企业级 API
    -   支持地域限制、投资者资格验证
    -   合规规则可配置,适应不同监管环境
-   **功能**: KYC/AML 集成、地域限制、投资者资格验证
-   **特点**: 与 Stobox 等合规服务商集成,提供企业级合规解决方案
-   **官方文档**: [Plume & Stobox 合作](https://blog.stobox.io/plume-stobox-arc/)

**4. 跨链互操作 (Cross-Chain)**

-   **需求**: 与以太坊、BSC 等主流链互操作
-   **公链实现方式**:
    -   EVM 兼容,支持以太坊生态的所有工具和协议
    -   通过跨链桥 (如 LayerZero、Axelar) 实现资产跨链转移
    -   支持主流稳定币 (USDC、USDT) 和 DeFi 协议
-   **功能**: 资产跨链桥、流动性共享
-   **特点**: EVM 兼容,支持主流跨链协议
-   **官方文档**: [Plume Network 跨链](https://docs.plumenetwork.xyz/plume/architecture/interoperability)

**5. Nest 质押协议 (原生集成)**

-   **需求**: 为 RWA 代币提供收益生成
-   **公链实现方式**:
    -   Nest 是 Plume 原生集成的质押协议,通过智能合约实现
    -   支持跨资产质押 (例如质押房地产代币获得稳定币收益)
    -   提供流动性挖矿和收益聚合功能
    -   质押奖励来自 RWA 资产的收益 (如租金、利息)
-   **功能**: 质押、流动性挖矿、收益聚合
-   **特点**: 原生集成,无需第三方协议,支持跨资产质押
-   **官方文档**: [Nest 质押协议](https://plume.org/blog/plume-arc)

**6. RWA 市场 (原生集成)**

-   **需求**: 提供 RWA 代币交易场所
-   **公链实现方式**:
    -   Plume 原生集成的 RWA 市场,通过智能合约实现
    -   支持订单簿、AMM (自动做市商)、拍卖等多种交易方式
    -   专为 RWA 优化的交易体验 (如支持部分份额交易)
    -   流动性池由 Nest 质押协议提供
-   **功能**: 订单簿、AMM、拍卖
-   **特点**: 专为 RWA 优化的交易体验,原生集成
-   **官方文档**: [Plume Network 市场](https://docs.plumenetwork.xyz/plume/features/marketplace)

#### 3.2.3 技术特性

-   **共识机制**: Proof of Representation (PoR) + PoS 混合共识
-   **区块时间**: 2-3 秒
-   **TPS**: 1000+ TPS
-   **最终性**: 快速最终性 (约 6-12 秒)
-   **Gas 费用**: 极低 (约 $0.001/交易)
-   **开发语言**: Solidity (智能合约) + Go/Rust (节点)
-   **数据可用性**: Celestia DA Layer
-   **主网上线**: 2025 年 6 月 5 日

### 3.3 公链与业务的耦合

#### 3.3.1 耦合方式

Plume Network 的公链功能与业务深度耦合:

1. **资产代币化流程 (Arc)**

    ```
    资产发行方 → Arc 引擎 (上传资产信息)
               → PoR 共识 (验证资产真实性)
               → 合规框架 (KYC 验证)
               → 自动发行 RWA 代币
    ```

2. **投资者购买流程**

    ```
    投资者 → 合规框架 (KYC 验证)
          → RWA 市场 (购买代币)
          → Nest 质押 (质押获得收益)
    ```

3. **资产数据验证流程**
    ```
    数据提供者 → 提交资产数据 + zkTLS 证明
               → PoR 共识 (多源验证)
               → 链上记录验证结果
               → 更新 RWA 代币元数据
    ```

#### 3.3.2 耦合优势

-   **全栈解决方案**: 从代币化到交易到质押,全流程在链上完成
-   **降低门槛**: 无需编写智能合约,通过 Arc 引擎一键代币化
-   **数据验证**: PoR 共识确保链下资产数据的真实性
-   **EVM 兼容**: 开发者可以使用熟悉的 Solidity 和 Web3 工具

### 3.4 值得学习的地方

#### 3.4.1 产品设计

1. **Arc 一键代币化引擎**

    - 需求: 降低资产代币化门槛
    - 方案: 提供可视化界面,无需编写代码即可代币化资产
    - 启示: USDable Chain 应提供类似的低代码/无代码工具,降低房东上链门槛

2. **Proof of Representation 共识**

    - 需求: 验证链下资产的真实性
    - 方案: 数据提供者网络 + zkTLS 证明 + 多源验证
    - 启示: USDable Chain 的 Oracle Hub 可以借鉴这种多源验证机制

3. **Nest 质押协议**
    - 需求: 为 RWA 代币提供收益生成
    - 方案: 支持跨资产质押,例如质押房地产代币获得稳定币收益
    - 启示: USDable Chain 可以考虑类似的质押和收益聚合功能

#### 3.4.2 业务模式

1. **全栈 RWA 解决方案**

    - 需求: 用户需要端到端的 RWA 服务
    - 方案: 从代币化到交易到质押,全流程在一个平台完成
    - 启示: USDable Chain 应提供完整的 RWA 生态,而不仅仅是基础设施

2. **与 Centrifuge 集成**

    - 需求: 扩大资产类型和流动性
    - 方案: 成为首个集成 Centrifuge v3 的区块链,引入结构化信贷资产
    - 启示: USDable Chain 应积极与其他 RWA 协议合作,实现跨生态互操作

3. **开发者友好**
    - 需求: 吸引开发者构建 RWA DApp
    - 方案: EVM 兼容 + 丰富的 SDK 和文档
    - 启示: USDable Chain 应提供完善的开发者工具和文档

---

## 4. 对比总结

### 4.1 核心业务对比

| 维度         | Centrifuge                | Plume Network                     |
| ------------ | ------------------------- | --------------------------------- |
| **主营业务** | 结构化信贷市场代币化      | 全栈 RWA 解决方案                 |
| **资产类型** | 信贷资产 (贷款、应收账款) | 多种资产 (房地产、艺术品、碳信用) |
| **目标用户** | 机构投资者为主            | 零售投资者 + 机构投资者           |
| **业务规模** | $5 亿+ TVL                | $40 亿预计规模                    |
| **成熟度**   | 成熟稳定 (多年运行)       | 新兴项目 (2025 年主网上线)        |

### 4.2 公链功能对比

| 功能模块       | Centrifuge         | Plume Network           |
| -------------- | ------------------ | ----------------------- |
| **资产存证**   | ✅ Asset Registry  | ✅ Arc 引擎             |
| **资产池管理** | ✅ Pool Management | ✅ RWA 市场             |
| **合规模块**   | ✅ Compliance      | ✅ Compliance Framework |
| **预言机**     | ✅ Oracle          | ✅ PoR 共识             |
| **跨链桥**     | ✅ Polkadot XCM    | ✅ EVM 跨链桥           |
| **质押协议**   | ❌ (依赖第三方)    | ✅ Nest 质押            |
| **DEX**        | ❌ (依赖第三方)    | ✅ RWA 市场             |

### 4.3 技术特性对比

| 技术特性     | Centrifuge       | Plume Network  |
| ------------ | ---------------- | -------------- |
| **共识机制** | NPoS (Polkadot)  | PoR + PoS      |
| **区块时间** | 6 秒             | 2-3 秒         |
| **TPS**      | 1000+            | 1000+          |
| **Gas 费用** | 极低             | 极低           |
| **开发语言** | Rust + Substrate | Solidity (EVM) |
| **学习曲线** | 陡峭             | 平缓           |

---

## 5. 对 USDable Chain 的启示

### 5.1 产品设计启示

1. **降低门槛**

    - 借鉴 Plume 的 Arc 引擎,提供可视化的资产代币化工具
    - 房东无需了解区块链技术,即可完成房地产上链

2. **全栈解决方案**

    - 借鉴 Plume 的全栈模式,提供从代币化到交易到分红的完整流程
    - 不仅是基础设施,更是完整的 RWA 生态

3. **多层级资产池**

    - 借鉴 Centrifuge 的分层设计,支持不同风险偏好的投资者
    - 例如: 高级份额 (稳定租金收益) + 次级份额 (房价增值收益)

4. **实时数据验证**
    - 借鉴 Plume 的 PoR 共识,建立多源数据验证机制
    - 确保房地产估值、租金数据的真实性

### 5.2 业务模式启示

1. **机构级服务**

    - 借鉴 Centrifuge Prime,为机构投资者提供差异化服务
    - 例如: 托管服务、审计报告、法律支持

2. **DeFi 集成**

    - 借鉴 Centrifuge 与 MakerDAO 的合作,将 XRWA 代币作为 DeFi 抵押品
    - 扩大资产流动性和用户基础

3. **跨生态合作**

    - 借鉴 Plume 与 Centrifuge 的集成,实现跨生态互操作
    - 与其他 RWA 协议合作,引入更多资产类型

4. **社区驱动治理**
    - 借鉴 Centrifuge 的治理模式,让 veABLE 持有者参与协议升级
    - 建立社区驱动的发展路线图

### 5.3 技术架构启示

1. **EVM 兼容**

    - 借鉴 Plume 的 EVM 兼容性,降低开发者门槛
    - 支持 Solidity 和主流 Web3 工具

2. **混合共识**

    - 借鉴 Plume 的 PoR 共识,在 PoS + BFT 基础上增加数据验证层
    - 确保链下资产数据的真实性

3. **模块化设计**

    - 借鉴 Centrifuge 的 Pallets 模块化设计,提高系统灵活性
    - 支持可插拔的合规规则和业务逻辑

4. **跨链互操作**
    - 借鉴两者的跨链方案,支持与以太坊、BSC、Polkadot 等主流链互操作
    - 扩大资产流动性和用户基础

### 5.4 USDable Chain 的差异化

基于竞品分析,USDable Chain 应在以下方面形成差异化:

1. **专注房地产 RWA**

    - Centrifuge 专注信贷,Plume 覆盖多种资产
    - USDable Chain 专注房地产,提供更深度的垂直解决方案

2. **租金分红自动化**

    - 竞品主要关注资产代币化和交易
    - USDable Chain 提供自动化的租金分红系统 (Distributor)

3. **节点生态**

    - 竞品主要依赖验证者节点
    - USDable Chain 建立 4 类业务节点 (资产、合规、服务、推广),形成完整生态

4. **veABLE 治理**
    - 竞品采用简单的代币治理
    - USDable Chain 采用 veABLE 模型,锁仓越久投票权越大,激励长期持有

---

**文档结束**

**下一步行动**:

1. 基于竞品分析,优化 USDable Chain 的产品设计
2. 制定与 Centrifuge、Plume 的合作策略
3. 确定 USDable Chain 的差异化定位和竞争优势
