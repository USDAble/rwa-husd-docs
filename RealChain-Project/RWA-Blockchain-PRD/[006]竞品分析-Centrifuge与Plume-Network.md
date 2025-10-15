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

本文档从**核心业务、公链功能、业务耦合、学习借鉴**四个维度,分析 Centrifuge 和 Plume Network 两个 RWA 公链项目,为 USDable Chain 提供产品设计参考。

---

## 2. Centrifuge 分析

### 2.1 核心业务

Centrifuge 专注于**结构化信贷市场的代币化**,已从旧版 Tinlake 升级到 **Centrifuge v3** (基于 Substrate 的 Polkadot Parachain)。

**核心业务**:

1. **信贷资产代币化**: 将房地产贷款、应收账款、发票等信贷资产代币化,为发行方提供融资渠道,为投资者提供稳定收益
2. **资产池管理**: 支持多层级份额 (Multi-Tranche)、自动化利息分配、Hub-and-Spoke 跨链架构
3. **Centrifuge Prime**: 为机构投资者提供合规的 RWA 投资通道,支持 KYC/AML 和链上信用评分

**目标用户**: 资产发行方 (中小企业、房地产开发商)、投资者 (机构/DeFi 协议/个人)、服务提供商 (评估/托管/法律)

**业务规模**: TVL 超 $5 亿美元,30+ 活跃资产池,合作机构包括 MakerDAO、Aave、BlockTower Capital

### 2.2 公链功能

#### 2.2.1 技术架构

Centrifuge Chain 是基于 **Substrate FRAME** 的 **Polkadot Parachain**,通过 **30+ 自定义 Pallets** 实现 RWA 全生命周期管理。

**Pallet**: Substrate 框架中的功能模块,用 Rust 编写,直接编译到区块链运行时,比智能合约更底层、更高效。

**Pallets 分类**: RWA 专用 (10+)、合规 (5+)、跨链 (5+)、预言机 (5+)、治理 (5+)

**参考**: [Centrifuge Pallets 官方文档](https://reference.centrifuge.io/)

#### 2.2.2 核心功能模块

**1. 资产存证 (anchors Pallet)**

-   **功能**: 为每个资产生成唯一 NFT,链上存储文档哈希 (POD 网络),支持版本控制
-   **特点**: 不可篡改、可审计
-   **文档**: https://reference.centrifuge.io/pallet_anchors/

**2. 资产池管理 (pool-system Pallet)**

-   **功能**: 管理投资池创建/投资/赎回,支持多层级 Tranches,自动化利息分配和本金偿还
-   **特点**: 灵活配置、实时净值计算
-   **文档**: https://reference.centrifuge.io/pallet_pool_system/

**3. 贷款管理 (loans Pallet)**

-   **功能**: 管理贷款创建/借款/还款,锁定抵押品 NFT,自动计算利息
-   **特点**: 链上自动化执行,无需人工干预
-   **文档**: https://reference.centrifuge.io/pallet_loans/

**4. 合规模块 (permissions + transfer-allowlist Pallets)**

-   **功能**: KYC/AML 白名单、地域限制、投资者资格验证
-   **特点**: 可插拔的合规规则引擎,支持自定义规则
-   **文档**: https://reference.centrifuge.io/pallet_permissions/

**5. 预言机模块 (oracle-collection + oracle-feed Pallets)**

-   **功能**: 资产估值更新、利率数据、违约事件通知
-   **特点**: 多源数据聚合、数据验证机制
-   **文档**: https://reference.centrifuge.io/pallet_oracle_collection/

**6. 跨链桥模块 (liquidity-pools + axelar-router Pallets)**

-   **功能**: 资产跨链转移、流动性共享
-   **特点**: 基于 Polkadot XCM + Axelar 跨链桥
-   **文档**: https://reference.centrifuge.io/pallet_liquidity_pools/

**7. 治理模块 (governance Pallet)**

-   **功能**: 提案创建、投票、执行,支持无分叉升级
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

**核心流程**:

1. **资产代币化**: 资产发行方 → 存证 (生成 NFT) → 创建资产池 → KYC 验证 → 发行份额代币
2. **投资者购买**: 投资者 → KYC 验证 → 购买份额代币 → 自动分红
3. **估值更新**: 预言机获取数据 → 更新 NFT 元数据 → 重新计算净值

**耦合优势**: 业务逻辑链上化 (透明可审计)、自动化执行 (无需人工干预)、合规内置 (无法绕过)、跨链互操作 (Polkadot 生态)

### 2.4 值得学习的地方

**产品设计**:

1. **多层级资产池**: 高级份额 (低风险) + 次级份额 (高风险),满足不同投资者需求
2. **实时净值计算**: 链上自动计算净值,考虑估值、利息、违约等因素
3. **灵活合规引擎**: 可插拔的合规规则,适应不同监管环境

**业务模式**:

1. **机构级服务**: Centrifuge Prime 提供托管、审计、法律支持等差异化服务
2. **DeFi 集成**: 与 MakerDAO、Aave 集成,将 RWA 作为抵押品,扩大流动性
3. **社区治理**: CFG 代币持有者参与协议升级决策

---

## 3. Plume Network 分析

### 3.1 核心业务

Plume Network 定位为**全栈 RWA 解决方案**,提供从代币化到交易到质押的完整服务。

**核心业务**:

1. **Arc 代币化引擎**: 一键代币化服务,支持房地产、艺术品、碳信用等多种资产,提供端到端工具链 (KYC、合规、发行、交易)
2. **Nest 质押协议**: 为 RWA 代币提供质押和收益生成,支持跨资产质押 (如质押房地产代币获得稳定币收益)
3. **RWA 市场**: 二级市场交易,支持限价单/市价单/拍卖,提供流动性池和做市商服务

**目标用户**: 资产发行方 (房地产开发商、艺术品收藏家)、投资者 (零售/机构/DeFi 用户)、开发者 (构建 RWA DApp)

**业务规模**: 180+ 生态项目,$40 亿预计代币化规模,2025 年 6 月 5 日主网上线

### 3.2 公链功能

#### 3.2.1 技术架构

Plume Network 是 **EVM 兼容的 Layer 1 区块链**,专为 RWA 设计,通过原生集成的智能合约和定制化共识机制实现 RWA 功能。

**架构分层**: 基础层 (EVM 兼容)、共识层 (PoR + PoS 混合)、应用层 (Arc + Nest + RWA 市场原生集成)、数据层 (Celestia DA)

**与 Centrifuge 对比**: Centrifuge 用 Substrate Pallets (Rust) 直接编译到 Runtime,Plume 用 Solidity 智能合约 (EVM) 执行

**参考**: [Plume Network 官方文档](https://docs.plumenetwork.xyz/)

#### 3.2.2 核心功能模块

**1. Arc 代币化引擎 (原生集成)**

-   **功能**: 一键创建 RWA 代币,可视化界面无需编码,自动化合规检查 (与 Stobox 集成),元数据管理 (链上/IPFS)
-   **特点**: 全栈解决方案,无需编写智能合约
-   **文档**: [Arc 代币化引擎](https://plume.org/blog/plume-arc)

**2. Proof of Representation (PoR) 共识**

-   **功能**: 数据提供者网络验证资产数据,zkTLS 证明验证链下数据,多源数据聚合,质押 PLUME 代币防作恶
-   **特点**: 专为 RWA 设计的共识机制,确保链下资产数据真实性
-   **文档**: [Proof of Representation](https://docs.plumenetwork.xyz/plume/architecture/consensus)

**3. 合规框架 (Compliance Framework)**

-   **功能**: KYC/AML 集成、地域限制、投资者资格验证,与 Stobox 等服务商集成
-   **特点**: 企业级合规解决方案,规则可配置
-   **文档**: [Plume & Stobox 合作](https://blog.stobox.io/plume-stobox-arc/)

**4. 跨链互操作 (Cross-Chain)**

-   **功能**: 资产跨链桥、流动性共享,支持 LayerZero/Axelar 跨链桥,支持主流稳定币和 DeFi 协议
-   **特点**: EVM 兼容,支持以太坊生态所有工具
-   **文档**: [Plume Network 跨链](https://docs.plumenetwork.xyz/plume/architecture/interoperability)

**5. Nest 质押协议 (原生集成)**

-   **功能**: 质押、流动性挖矿、收益聚合,支持跨资产质押 (如质押房地产代币获得稳定币收益),奖励来自 RWA 资产收益
-   **特点**: 原生集成,无需第三方协议
-   **文档**: [Nest 质押协议](https://plume.org/blog/plume-arc)

**6. RWA 市场 (原生集成)**

-   **功能**: 订单簿、AMM、拍卖,支持部分份额交易,流动性池由 Nest 提供
-   **特点**: 专为 RWA 优化的交易体验
-   **文档**: [Plume Network 市场](https://docs.plumenetwork.xyz/plume/features/marketplace)

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

**核心流程**:

1. **资产代币化**: 资产发行方 → Arc 引擎上传信息 → PoR 验证真实性 → KYC 验证 → 自动发行代币
2. **投资者购买**: 投资者 → KYC 验证 → RWA 市场购买 → Nest 质押获得收益
3. **数据验证**: 数据提供者提交数据 + zkTLS 证明 → PoR 多源验证 → 链上记录 → 更新元数据

**耦合优势**: 全栈解决方案 (代币化到交易到质押全流程)、降低门槛 (Arc 一键代币化)、数据验证 (PoR 确保真实性)、EVM 兼容 (熟悉的开发工具)

### 3.4 值得学习的地方

**产品设计**:

1. **Arc 一键代币化**: 可视化界面无需编码,降低资产代币化门槛
2. **PoR 共识**: 数据提供者网络 + zkTLS 证明 + 多源验证,确保链下资产真实性
3. **Nest 质押协议**: 支持跨资产质押,为 RWA 代币提供收益生成

**业务模式**:

1. **全栈解决方案**: 从代币化到交易到质押,全流程在一个平台完成
2. **跨生态合作**: 成为首个集成 Centrifuge v3 的区块链,引入结构化信贷资产
3. **开发者友好**: EVM 兼容 + 丰富的 SDK 和文档,吸引开发者构建 RWA DApp

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

1. **降低门槛**: 借鉴 Plume Arc 引擎,提供可视化资产代币化工具,房东无需了解区块链即可上链
2. **全栈解决方案**: 借鉴 Plume 全栈模式,提供从代币化到交易到分红的完整流程
3. **多层级资产池**: 借鉴 Centrifuge 分层设计,支持不同风险偏好 (如高级份额稳定租金 + 次级份额房价增值)
4. **实时数据验证**: 借鉴 Plume PoR 共识,建立多源数据验证机制,确保房地产估值和租金数据真实性

### 5.2 业务模式启示

1. **机构级服务**: 借鉴 Centrifuge Prime,为机构投资者提供托管、审计、法律支持等差异化服务
2. **DeFi 集成**: 借鉴 Centrifuge 与 MakerDAO 合作,将 XRWA 代币作为 DeFi 抵押品,扩大流动性
3. **跨生态合作**: 借鉴 Plume 与 Centrifuge 集成,与其他 RWA 协议合作,引入更多资产类型
4. **社区驱动治理**: 借鉴 Centrifuge 治理模式,让 veABLE 持有者参与协议升级

### 5.3 技术架构启示

1. **EVM 兼容**: 借鉴 Plume EVM 兼容性,降低开发者门槛,支持 Solidity 和主流 Web3 工具
2. **混合共识**: 借鉴 Plume PoR 共识,在 PoS + BFT 基础上增加数据验证层
3. **模块化设计**: 借鉴 Centrifuge Pallets 模块化设计,支持可插拔的合规规则和业务逻辑
4. **跨链互操作**: 借鉴两者跨链方案,支持与以太坊、BSC、Polkadot 等主流链互操作

### 5.4 USDable Chain 的差异化

1. **专注房地产 RWA**: Centrifuge 专注信贷,Plume 覆盖多种资产,USDable Chain 专注房地产垂直解决方案
2. **租金分红自动化**: 竞品主要关注代币化和交易,USDable Chain 提供自动化租金分红系统 (Distributor)
3. **节点生态**: 竞品依赖验证者节点,USDable Chain 建立 4 类业务节点 (资产/合规/服务/推广),形成完整生态
4. **veABLE 治理**: 竞品采用简单代币治理,USDable Chain 采用 veABLE 模型,锁仓越久投票权越大,激励长期持有

---

**文档结束**

**下一步行动**:

1. 基于竞品分析,优化 USDable Chain 的产品设计
2. 制定与 Centrifuge、Plume 的合作策略
3. 确定 USDable Chain 的差异化定位和竞争优势
