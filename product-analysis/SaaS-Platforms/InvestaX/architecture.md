# InvestaX 技术架构分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 10:30:32 CST  
**文档类型**: 技术架构分析  
**定位**: MAS持牌RWA代币化SaaS平台

---

## 📑 目录

1. [系统整体架构](#1-系统整体架构)
2. [核心模块详解](#2-核心模块详解)
3. [技术选型分析](#3-技术选型分析)
4. [合规架构](#4-合规架构)
5. [安全架构](#5-安全架构)

---

## 1. 系统整体架构

### 1.1 InvestaX 整体架构

```mermaid
graph TB
    subgraph "Application Layer"
        Portal[InvestaX Portal<br/>发行方门户]
        Investor[Investor Portal<br/>投资者门户]
        Admin[Admin Dashboard<br/>管理后台]
    end
    
    subgraph "Service Layer"
        Issuance[Issuance Service<br/>发行服务]
        Trading[Trading Service<br/>交易服务]
        Custody[Custody Service<br/>托管服务]
        Lifecycle[Lifecycle Service<br/>生命周期管理]
    end
    
    subgraph "Blockchain Layer"
        Ethereum[Ethereum]
        Polygon[Polygon]
        BASE[BASE]
        Algorand[Algorand]
        Tezos[Tezos]
        Hedera[Hedera]
    end
    
    subgraph "Compliance Layer"
        MAS[MAS Compliance<br/>MAS合规]
        KYC[KYC/AML Service<br/>KYC/AML服务]
        Whitelist[Whitelist Manager<br/>白名单管理]
    end
    
    Portal --> Issuance
    Investor --> Trading
    Admin --> Lifecycle
    Issuance --> Ethereum
    Issuance --> Polygon
    Issuance --> BASE
    Trading --> Ethereum
    Trading --> Polygon
    Custody --> Ethereum
    Lifecycle --> Ethereum
    MAS --> Issuance
    KYC --> Whitelist
    Whitelist --> Trading
    
    style Issuance fill:#4CAF50
    style MAS fill:#2196F3
    style Ethereum fill:#FF9800
```

### 1.2 核心组件说明

| 组件 | 职责 | 关键功能 |
|------|------|----------|
| **Issuance Service** | 代币发行服务 | 智能合约部署、代币铸造、分发 |
| **Trading Service** | 交易服务 | 一级市场、二级市场、IX Exchange |
| **Custody Service** | 托管服务 | 资产托管、密钥管理、安全存储 |
| **Lifecycle Service** | 生命周期管理 | 分红、赎回、公司行动 |
| **MAS Compliance** | MAS合规 | 监管报告、合规检查、审计 |

### 1.3 技术栈

**区块链层**：
- Ethereum（主网）
- Polygon（Layer 2）
- BASE（Coinbase L2）
- Algorand（高性能）
- Tezos（治理）
- Hedera（企业级）

**后端层**：
- Node.js 18.x
- Express.js
- PostgreSQL
- Redis
- RabbitMQ

**前端层**：
- React 18.x
- TypeScript
- Material-UI
- Web3.js

---

## 2. 核心模块详解

### 2.1 Issuance Service（发行服务）

**发行流程**：
```mermaid
sequenceDiagram
    participant Issuer as 发行方
    participant Portal as InvestaX Portal
    participant Issuance as Issuance Service
    participant Blockchain as 区块链
    participant MAS as MAS Compliance
    
    Issuer->>Portal: 创建发行项目
    Portal->>Issuance: 提交发行请求
    Issuance->>MAS: 合规检查
    MAS-->>Issuance: 合规通过
    Issuance->>Blockchain: 部署智能合约
    Blockchain-->>Issuance: 合约地址
    Issuance->>Blockchain: 铸造代币
    Blockchain-->>Issuer: 代币发行完成
```

**支持的代币标准**：
- ERC20（Ethereum、Polygon、BASE）
- ASA（Algorand Standard Asset）
- FA2（Tezos）
- HTS（Hedera Token Service）

**发行配置**：
```typescript
interface IssuanceConfig {
    tokenName: string;
    tokenSymbol: string;
    totalSupply: number;
    blockchain: 'Ethereum' | 'Polygon' | 'BASE' | 'Algorand' | 'Tezos' | 'Hedera';
    compliance: {
        requireKYC: boolean;
        accreditedOnly: boolean;
        jurisdictions: string[];
        lockupPeriod?: number;
    };
    economics: {
        pricePerToken: number;
        minInvestment: number;
        maxInvestment?: number;
        dividendFrequency?: 'monthly' | 'quarterly' | 'annually';
    };
}
```

### 2.2 Trading Service（交易服务）

**交易架构**：
```mermaid
graph LR
    subgraph "Primary Market"
        Issuance[Token Issuance<br/>代币发行]
        Distribution[Distribution<br/>分发]
    end
    
    subgraph "Secondary Market"
        P2P[P2P Trading<br/>P2P交易]
        IXExchange[IX Exchange<br/>集中交易所]
    end
    
    subgraph "Liquidity"
        AMM[AMM Pool<br/>自动做市商]
        OrderBook[Order Book<br/>订单簿]
    end
    
    Issuance --> Distribution
    Distribution --> P2P
    Distribution --> IXExchange
    P2P --> AMM
    IXExchange --> OrderBook
    
    style Issuance fill:#4CAF50
    style IXExchange fill:#2196F3
```

**IX Exchange特点**：
- 集中订单簿
- 实时价格发现
- 高流动性
- 低交易费用

### 2.3 Custody Service（托管服务）

**托管架构**：
```mermaid
graph TB
    subgraph "Custody Service"
        HSM[Hardware Security Module<br/>硬件安全模块]
        MultiSig[Multi-Signature Wallet<br/>多签钱包]
        ColdStorage[Cold Storage<br/>冷存储]
        HotWallet[Hot Wallet<br/>热钱包]
    end
    
    subgraph "Key Management"
        KeyGen[Key Generation<br/>密钥生成]
        KeyRotation[Key Rotation<br/>密钥轮换]
        KeyBackup[Key Backup<br/>密钥备份]
    end
    
    HSM --> MultiSig
    HSM --> ColdStorage
    MultiSig --> HotWallet
    KeyGen --> HSM
    KeyRotation --> HSM
    KeyBackup --> ColdStorage
    
    style HSM fill:#4CAF50
    style MultiSig fill:#2196F3
```

**安全措施**：
- ✅ HSM硬件加密
- ✅ 多签钱包（2/3或3/5）
- ✅ 冷热钱包分离
- ✅ 定期密钥轮换
- ✅ 灾备恢复机制

### 2.4 Lifecycle Service（生命周期管理）

**生命周期事件**：

| 事件类型 | 描述 | 频率 | 自动化 |
|---------|------|------|--------|
| **Dividend** | 分红支付 | 月度/季度 | ✅ 自动 |
| **Redemption** | 代币赎回 | 按需 | ⚙️ 半自动 |
| **Corporate Action** | 公司行动 | 按需 | ⚙️ 半自动 |
| **Voting** | 投票治理 | 按需 | ✅ 自动 |
| **Reporting** | 合规报告 | 月度 | ✅ 自动 |

**分红流程**：
```solidity
// 分红智能合约（简化）
contract DividendDistributor {
    mapping(address => uint256) public dividends;
    
    function distributeDividends(
        address[] memory holders,
        uint256[] memory amounts
    ) external onlyAdmin {
        require(holders.length == amounts.length, "Length mismatch");
        
        for (uint256 i = 0; i < holders.length; i++) {
            dividends[holders[i]] += amounts[i];
        }
        
        emit DividendsDistributed(holders, amounts);
    }
    
    function claimDividend() external {
        uint256 amount = dividends[msg.sender];
        require(amount > 0, "No dividends");
        
        dividends[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
        
        emit DividendClaimed(msg.sender, amount);
    }
}
```

---

## 3. 技术选型分析

### 3.1 多链支持策略

**为什么支持6条链**：

| 区块链 | 优势 | 使用场景 |
|--------|------|---------|
| **Ethereum** | 最成熟、最安全 | 高价值资产 |
| **Polygon** | 低Gas、高速 | 零售投资者 |
| **BASE** | Coinbase支持 | 美国市场 |
| **Algorand** | 高性能、低成本 | 大规模发行 |
| **Tezos** | 链上治理 | 治理代币 |
| **Hedera** | 企业级、合规 | 机构客户 |

**跨链桥接**：
- 使用LayerZero或Wormhole
- 支持资产跨链转移
- 统一流动性池

### 3.2 SaaS订阅模型

**订阅层级**：

| 层级 | 月费 | 功能 | 适用对象 |
|------|------|------|---------|
| **Starter** | $999 | 基础发行、5个项目 | 小型发行方 |
| **Professional** | $2,999 | 无限项目、高级功能 | 中型发行方 |
| **Enterprise** | 定制 | 全功能、专属支持 | 大型机构 |

**收费模式**：
- 固定月费（无交易费）
- 按项目数量计费
- 按交易量计费（可选）

---

## 4. 合规架构

### 4.1 MAS合规框架

**MAS许可证**：
- CMS100635（资本市场服务许可）
- 允许运营数字证券平台
- 受MAS监管和审计

**合规要求**：
```mermaid
graph TB
    subgraph "MAS Compliance"
        KYC[KYC/AML<br/>身份验证]
        AML[AML Screening<br/>反洗钱筛查]
        Reporting[Regulatory Reporting<br/>监管报告]
        Audit[Audit Trail<br/>审计追踪]
    end
    
    subgraph "Investor Classification"
        Accredited[Accredited Investor<br/>合格投资者]
        Institutional[Institutional Investor<br/>机构投资者]
        Retail[Retail Investor<br/>零售投资者]
    end
    
    KYC --> Accredited
    KYC --> Institutional
    KYC --> Retail
    AML --> KYC
    Reporting --> MAS[MAS]
    Audit --> Reporting
    
    style KYC fill:#4CAF50
    style MAS fill:#2196F3
```

### 4.2 KYC/AML流程

**KYC提供商集成**：
- Onfido
- Jumio
- Sumsub
- Chainalysis（链上分析）

**验证流程**：
1. 身份文档上传
2. 人脸识别验证
3. 地址证明验证
4. AML筛查（制裁名单、PEP）
5. 风险评分
6. 人工审核（高风险）

---

## 5. 安全架构

### 5.1 多层安全防护

```mermaid
graph TB
    subgraph "应用层安全"
        WAF[Web Application Firewall]
        DDoS[DDoS Protection]
        Auth[Multi-Factor Authentication]
    end
    
    subgraph "数据层安全"
        Encryption[Data Encryption]
        Backup[Encrypted Backup]
        Access[Access Control]
    end
    
    subgraph "区块链层安全"
        MultiSig[Multi-Signature]
        HSM[Hardware Security Module]
        Audit[Smart Contract Audit]
    end
    
    WAF --> Encryption
    DDoS --> Encryption
    Auth --> Access
    Encryption --> MultiSig
    Backup --> MultiSig
    Access --> HSM
    
    style WAF fill:#4CAF50
    style Encryption fill:#2196F3
    style MultiSig fill:#FF9800
```

### 5.2 审计和认证

**安全审计**：
- ✅ SOC 2 Type II认证
- ✅ ISO 27001认证
- ✅ 智能合约审计（Certik、OpenZeppelin）
- ✅ 渗透测试（年度）

---

## 📚 参考资源

- [InvestaX官网](https://investax.io)
- [MAS官网](https://www.mas.gov.sg)
- [InvestaX文档](https://docs.investax.io)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 10:30:32 CST
