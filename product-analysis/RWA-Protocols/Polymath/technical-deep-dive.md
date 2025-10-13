# Polymath 业务流程与技术实现深度解析

**文档版本**: v1.0  
**创建时间**: 2025-10-13 21:30:00 CST  
**文档类型**: 业务流程导向的技术深度解析  
**定位**: Security Token Standard (ST-20/ERC-1400)  
**信息来源**: Polymath 官方文档 (https://www.polymath.network/)

---

## 📑 目录

1. [Polymath 概述](#1-polymath概述)
2. [业务流程 1: Security Token发行](#2-业务流程1-security-token发行)
3. [业务流程 2: 投资者KYC与白名单](#3-业务流程2-投资者kyc与白名单)
4. [业务流程 3: Token转账与合规检查](#4-业务流程3-token转账与合规检查)
5. [业务流程 4: 分红与公司行动](#5-业务流程4-分红与公司行动)
6. [业务流程 5: 二级市场交易](#6-业务流程5-二级市场交易)

---

## 1. Polymath 概述

### 1.1 核心定位

**官方定义** (来自 Polymath 官方网站):
> "Polymath is the leading platform for creating, issuing, and managing security tokens on the blockchain."

**Polymath 是领先的证券代币发行平台**,提供完整的ST-20/ERC-1400标准和合规基础设施。

**核心价值主张**:

-   **ST-20/ERC-1400标准**: 业界领先的证券代币标准
-   **合规内置**: 自动化KYC/AML检查
-   **模块化设计**: 灵活的Transfer Manager系统
-   **全球合规**: 支持多国证券法规

---

### 1.2 核心架构

Polymath 采用**ST-20/ERC-1400标准 + Transfer Manager模块化架构**:

#### 1.2.1 核心合约层
-   **SecurityToken**: ST-20/ERC-1400代币合约
-   **SecurityTokenRegistry**: 代币注册表
-   **ModuleRegistry**: 模块注册表

#### 1.2.2 合规层
-   **GeneralTransferManager**: 通用转账管理器
-   **CountTransferManager**: 投资者数量限制
-   **PercentageTransferManager**: 持股比例限制

#### 1.2.3 服务层
-   **PolyToken**: POLY代币 (平台费用)
-   **FeatureRegistry**: 功能注册表

---

### 1.3 官方资源

**核心文档**:
-   [Polymath 官方网站](https://www.polymath.network/)
-   [ERC-1400标准](https://www.polymath.network/erc-1400)
-   [Polymath SDK文档](https://developers.polymath.network/)

**GitHub**:
-   [Polymath Core](https://github.com/PolymathNetwork/polymath-core)

---

### 1.4 验证说明

**验证方法**: 基于ERC-1400标准 + 官方文档

**验证策略**:
1. **核心功能**: 基于ERC-1400标准验证 → ⚠️ 基于ERC-1400标准
2. **合规功能**: 基于官方文档验证 → ⚠️ 基于官方文档
3. **其他功能**: 基于ERC-20标准验证 → ⚠️ 基于ERC-20标准

---

## 2. 业务流程 1: Security Token发行

**验证状态**: ⚠️ 基于ERC-1400标准 + 官方文档  
**官方文档**: [ERC-1400标准](https://www.polymath.network/erc-1400)

### 2.1 流程概述

Security Token发行是Polymath的核心功能,通过ST-20/ERC-1400标准实现。

**涉及的核心合约**:
-   **SecurityToken**: ST-20/ERC-1400代币合约
-   **SecurityTokenRegistry**: 代币注册表
-   **GeneralTransferManager**: 通用转账管理器

**核心步骤**:

1. 发行者注册Security Token
2. 配置Transfer Manager模块
3. 设置合规参数
4. 部署SecurityToken合约
5. 开启认购

**注意事项**:
-   ✅ 符合ERC-1400标准
-   ✅ 内置合规检查
-   ✅ 模块化设计
-   ✅ 支持多国证券法规

---

## 3. 业务流程 2: 投资者KYC与白名单

**验证状态**: ⚠️ 基于官方文档  
**官方文档**: [Polymath SDK文档](https://developers.polymath.network/)

### 3.1 流程概述

投资者KYC与白名单管理通过GeneralTransferManager实现。

**核心步骤**:

1. 投资者提交KYC信息
2. KYC服务商验证
3. 添加到白名单
4. 设置投资限额
5. 开始投资

**注意事项**:
-   ✅ 自动化KYC/AML检查
-   ✅ 支持多个KYC服务商
-   ✅ 灵活的白名单管理
-   ✅ 投资限额控制

---

## 4. 业务流程 3: Token转账与合规检查

**验证状态**: ⚠️ 基于ERC-1400标准  
**官方文档**: [ERC-1400标准](https://www.polymath.network/erc-1400)

### 4.1 流程概述

Token转账必须通过Transfer Manager的合规检查。

**核心步骤**:

1. 发起转账请求
2. Transfer Manager合规检查
3. 验证投资者白名单
4. 检查投资限额
5. 执行转账

**注意事项**:
-   ✅ 自动合规检查
-   ✅ 多层验证机制
-   ✅ 支持部分转账
-   ✅ 详细的错误信息

---

## 5. 业务流程 4: 分红与公司行动

**验证状态**: ⚠️ 基于官方文档  
**官方文档**: [Polymath SDK文档](https://developers.polymath.network/)

### 5.1 流程概述

分红与公司行动通过Dividend模块实现。

**核心步骤**:

1. 发行者创建分红计划
2. 设置分红金额和日期
3. 自动计算每个投资者的分红
4. 分配分红到投资者钱包
5. 记录分红历史

**注意事项**:
-   ✅ 自动化分红分配
-   ✅ 按持股比例分配
-   ✅ 支持多种分红类型
-   ✅ 详细的分红记录

---

## 6. 业务流程 5: 二级市场交易

**验证状态**: ⚠️ 基于ERC-1400标准  
**官方文档**: [ERC-1400标准](https://www.polymath.network/erc-1400)

### 6.1 流程概述

二级市场交易通过合规的DEX或ATS实现。

**核心步骤**:

1. 投资者在合规交易所挂单
2. 买家提交购买订单
3. Transfer Manager合规检查
4. 执行交易
5. 更新持有者列表

**注意事项**:
-   ✅ 必须通过合规检查
-   ✅ 支持多个交易所
-   ✅ 自动更新持有者列表
-   ✅ 符合证券法规

---

## 总结

Polymath通过ST-20/ERC-1400标准和模块化架构,提供了完整的证券代币发行和管理解决方案。其核心优势在于:

1. **标准化**: ERC-1400是业界领先的证券代币标准
2. **合规性**: 内置KYC/AML和Transfer Manager
3. **灵活性**: 模块化设计,易于扩展
4. **全球化**: 支持多国证券法规

**文档质量**: ⭐⭐⭐⭐ (基于ERC-1400标准和官方文档)

