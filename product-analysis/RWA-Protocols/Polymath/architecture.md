# Polymath 技术架构分析

**文档版本**: v2.1
**创建时间**: 2025-10-14 09:35:00 CST
**文档类型**: 技术架构分析
**定位**: Security Token Standard (ST-20/ERC-1400)
**信息来源**: GitHub 官方合约 + ERC-1400 标准

---

## 📑 目录

1. [系统整体架构](#1-系统整体架构)
2. [ST-20/ERC-1400 架构](#2-st-20erc-1400架构)
3. [核心模块详解](#3-核心模块详解)
4. [技术选型分析](#4-技术选型分析)
5. [数据流程](#5-数据流程)
6. [安全架构](#6-安全架构)

---

## 1. 系统整体架构

### 1.1 Polymath 整体架构

```mermaid
graph TB
    subgraph "Application Layer"
        Platform[Polymath Platform<br/>Web应用]
        TokenStudio[Token Studio<br/>Token管理]
    end

    subgraph "Factory Layer"
        STFactory[STFactory<br/>Token工厂]
        ModuleFactory[ModuleFactory<br/>模块工厂]
    end

    subgraph "Token Layer"
        SecurityToken[SecurityToken<br/>ST-20/ERC-1400]
        Partitions[Partitions<br/>分区管理]
    end

    subgraph "Module Layer"
        GTM[GeneralTransferManager<br/>通用转账管理]
        CTM[CountTransferManager<br/>持有人数量限制]
        PTM[PercentageTransferManager<br/>持有比例限制]
        VRTM[VolumeRestrictionTM<br/>交易量限制]
    end

    subgraph "Registry Layer"
        ModuleRegistry[ModuleRegistry<br/>模块注册表]
        SecurityTokenRegistry[SecurityTokenRegistry<br/>Token注册表]
    end

    Platform --> STFactory
    TokenStudio --> SecurityToken
    STFactory --> SecurityToken
    STFactory --> ModuleFactory
    SecurityToken --> Partitions
    SecurityToken --> GTM
    SecurityToken --> CTM
    SecurityToken --> PTM
    SecurityToken --> VRTM
    ModuleFactory --> ModuleRegistry
    STFactory --> SecurityTokenRegistry

    style SecurityToken fill:#4CAF50
    style ModuleRegistry fill:#2196F3
    style STFactory fill:#FF9800
```

### 1.2 核心组件说明

| 组件                       | 职责           | 关键功能                       |
| -------------------------- | -------------- | ------------------------------ |
| **SecurityToken**          | ST-20/ERC-1400 | ERC-20 + 合规 + 分区           |
| **STFactory**              | Token 工厂     | 标准化部署、版本管理           |
| **ModuleRegistry**         | 模块注册表     | 模块发现、版本管理、审核       |
| **GeneralTransferManager** | 通用转账管理   | 白名单、KYC/AML、锁定期        |
| **CountTransferManager**   | 持有人数量限制 | 最大持有人数量限制             |
| **PercentageTransferManager** | 持有比例限制 | 单个投资者最大持有比例限制     |
| **VolumeRestrictionTM**    | 交易量限制     | 24小时交易量限制               |

---

## 2. ST-20/ERC-1400 架构

### 2.1 SecurityToken 结构

```mermaid
graph TB
    subgraph "SecurityToken Contract"
        ERC20[ERC-20 Implementation<br/>标准代币功能]
        ERC1400[ERC-1400 Extension<br/>分区功能]
        ModuleInterface[Module Interface<br/>模块接口]
    end

    subgraph "Compliance Flow"
        CheckModules[Check Modules<br/>检查模块]
        ExecuteTransfer[Execute Transfer<br/>执行转账]
        EmitEvents[Emit Events<br/>发出事件]
    end

    ERC20 --> ModuleInterface
    ERC1400 --> ModuleInterface
    ModuleInterface --> CheckModules
    CheckModules --> ExecuteTransfer
    ExecuteTransfer --> EmitEvents

    style ERC20 fill:#4CAF50
    style ERC1400 fill:#2196F3
```

### 2.2 ERC-1400 分区功能

**分区(Partitions)**：
- 允许将 Token 分成不同的"分区"
- 每个分区可以有不同的转账限制
- 支持部分转账和部分赎回

**分区类型**：
- **Locked Partition**: 锁定分区(不可转账)
- **Unlocked Partition**: 解锁分区(可转账)
- **Reserved Partition**: 预留分区(特殊用途)

---

## 3. 核心模块详解

### 3.1 SecurityToken 合约

**核心方法**:

```solidity
// 转账(带模块检查)
function transfer(address to, uint256 value) public returns (bool) {
    require(_verifyTransfer(msg.sender, to, value), "Transfer not allowed");
    return super.transfer(to, value);
}

// 模块检查
function _verifyTransfer(address from, address to, uint256 value) internal returns (bool) {
    for (uint i = 0; i < modules.length; i++) {
        if (!modules[i].verifyTransfer(from, to, value)) {
            return false;
        }
    }
    return true;
}

// 添加模块
function addModule(address module) public onlyOwner {
    modules.push(IModule(module));
}
```

### 3.2 GeneralTransferManager (GTM)

**核心功能**:
- 白名单管理
- KYC/AML 验证
- 锁定期管理
- 投资者类型管理

**核心方法**:

```solidity
// 验证转账
function verifyTransfer(address from, address to, uint256 value) public returns (bool) {
    // 检查发送方白名单
    if (!whitelist[from]) return false;
    
    // 检查接收方白名单
    if (!whitelist[to]) return false;
    
    // 检查锁定期
    if (block.timestamp < lockupTime[from]) return false;
    
    return true;
}

// 添加到白名单
function modifyWhitelist(address investor, uint256 fromTime, uint256 toTime) public onlyOwner {
    whitelist[investor] = true;
    fromTime[investor] = fromTime;
    toTime[investor] = toTime;
}
```

### 3.3 CountTransferManager (CTM)

**核心功能**:
- 限制最大持有人数量
- 符合 Reg D 506(c) 要求(最多 2000 人)

**核心方法**:

```solidity
// 验证转账
function verifyTransfer(address from, address to, uint256 value) public returns (bool) {
    // 如果接收方已经持有,直接允许
    if (balanceOf(to) > 0) return true;
    
    // 检查是否超过最大持有人数量
    if (holderCount >= maxHolderCount) return false;
    
    return true;
}
```

---

## 4. 技术选型分析

### 4.1 为什么选择 ERC-1400

**优势**：
- ✅ **行业标准**：广泛认可的 Security Token 标准
- ✅ **分区功能**：支持复杂的合规需求
- ✅ **ERC-20 兼容**：与现有 DeFi 生态兼容
- ✅ **可扩展**：支持自定义模块

**ERC-1400 vs 其他标准**：

| 特性       | ERC-1400 | R-Token | ERC-3643 |
| ---------- | -------- | ------- | -------- |
| ERC-20 兼容 | ✅ | ✅ | ✅ |
| 分区功能   | ✅ | ❌ | ❌ |
| 模块化     | ✅ | ✅ | ✅ |
| 行业采用   | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## 5. 数据流程

### 5.1 Security Token 发行流程

```mermaid
sequenceDiagram
    participant Issuer as 发行方
    participant Platform as Polymath Platform
    participant STFactory as STFactory
    participant SecurityToken as SecurityToken
    participant ModuleRegistry as ModuleRegistry

    Issuer->>Platform: 创建 Security Token
    Platform->>STFactory: deployToken()
    STFactory->>SecurityToken: 部署合约
    SecurityToken-->>STFactory: Token 地址
    STFactory->>ModuleRegistry: 注册 Token
    Platform->>SecurityToken: 添加模块
    SecurityToken-->>Issuer: Token 发行完成
```

### 5.2 转账流程

```mermaid
sequenceDiagram
    participant Sender as 发送方
    participant SecurityToken as SecurityToken
    participant GTM as GeneralTransferManager
    participant CTM as CountTransferManager
    participant Receiver as 接收方

    Sender->>SecurityToken: transfer(to, value)
    SecurityToken->>GTM: verifyTransfer()
    GTM-->>SecurityToken: true
    SecurityToken->>CTM: verifyTransfer()
    CTM-->>SecurityToken: true
    SecurityToken->>Receiver: 转账成功
```

---

## 6. 安全架构

### 6.1 多层安全防护

```mermaid
graph TB
    subgraph "合约级安全"
        Audit[智能合约审计]
        Access[访问控制]
        Pausable[紧急暂停]
    end

    subgraph "模块级安全"
        ModuleAudit[模块审核]
        Whitelist[白名单]
        Lockup[锁定期]
    end

    subgraph "网络级安全"
        Ethereum[Ethereum 安全]
        Events[事件日志]
        Audit2[审计追踪]
    end

    Audit --> ModuleAudit
    Access --> Whitelist
    Pausable --> Lockup
    ModuleAudit --> Ethereum
    Whitelist --> Events
    Lockup --> Audit2

    style Audit fill:#4CAF50
    style ModuleAudit fill:#2196F3
    style Ethereum fill:#FF9800
```

---

## 📚 参考资源

- [Polymath GitHub](https://github.com/PolymathNetwork/polymath-core)
- [ERC-1400 标准](https://github.com/ethereum/EIPs/issues/1400)
- [Polymath 文档](https://docs.polymath.network)
- [ST-20 标准](https://github.com/PolymathNetwork/polymath-core/blob/master/docs/ST20.md)

---

**文档维护**: RWA-HUSD 技术团队  
**最后更新**: 2025-10-14 09:35:00 CST

