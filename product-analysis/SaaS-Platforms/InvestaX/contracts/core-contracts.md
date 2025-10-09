# InvestaX 核心合约分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 10:32:00 CST  
**框架**: Multi-Chain (Ethereum, Polygon, BASE, Algorand, Tezos, Hedera)

---

## 📑 目录

1. [合约列表](#1-合约列表)
2. [数据结构](#2-数据结构)
3. [主要接口](#3-主要接口)

---

## 1. 合约列表

### 1.1 核心合约（Ethereum/Polygon/BASE）

| 合约名称 | 职责 | 代码行数 |
|---------|------|---------|
| **SecurityToken.sol** | 证券代币 | ~500行 |
| **WhitelistManager.sol** | 白名单管理 | ~300行 |
| **DividendDistributor.sol** | 分红分配 | ~400行 |
| **TransferRestrictor.sol** | 转账限制 | ~350行 |
| **ComplianceModule.sol** | 合规模块 | ~450行 |

### 1.2 Algorand合约（ASA）

| 合约名称 | 职责 | 代码行数 |
|---------|------|---------|
| **AssetConfig** | 资产配置 | ~200行 |
| **TransferLogic** | 转账逻辑 | ~250行 |
| **ClawbackManager** | 回收管理 | ~200行 |

---

## 2. 数据结构

### 2.1 SecurityToken结构

```solidity
contract SecurityToken is ERC20 {
    struct TokenInfo {
        string name;
        string symbol;
        uint256 totalSupply;
        address issuer;
        uint256 issuanceDate;
        uint256 maturityDate;
        TokenStatus status;
    }
    
    struct InvestorInfo {
        bool isWhitelisted;
        uint256 balance;
        uint256 lockupEnd;
        string jurisdiction;
        InvestorType investorType;
    }
    
    enum TokenStatus {
        Active,
        Paused,
        Matured,
        Redeemed
    }
    
    enum InvestorType {
        Retail,
        Accredited,
        Institutional
    }
    
    mapping(address => InvestorInfo) public investors;
    TokenInfo public tokenInfo;
}
```

### 2.2 Dividend结构

```solidity
struct DividendPayment {
    uint256 paymentId;
    uint256 totalAmount;
    uint256 paymentDate;
    uint256 recordDate;
    address currency;
    PaymentStatus status;
}

struct InvestorDividend {
    uint256 amount;
    bool claimed;
    uint256 claimDate;
}

enum PaymentStatus {
    Pending,
    Distributed,
    Completed
}
```

---

## 3. 主要接口

### 3.1 SecurityToken接口

```solidity
interface ISecurityToken {
    // 转账（带合规检查）
    function transfer(address to, uint256 amount) external returns (bool);
    
    // 检查转账是否允许
    function canTransfer(address from, address to, uint256 amount) 
        external view returns (bool, string memory);
    
    // 添加到白名单
    function addToWhitelist(address investor, InvestorType investorType) 
        external;
    
    // 设置锁定期
    function setLockup(address investor, uint256 lockupEnd) external;
    
    // 暂停/恢复
    function pause() external;
    function unpause() external;
}
```

### 3.2 DividendDistributor接口

```solidity
interface IDividendDistributor {
    // 创建分红
    function createDividend(
        uint256 totalAmount,
        uint256 recordDate,
        address currency
    ) external returns (uint256 paymentId);
    
    // 分配分红
    function distributeDividends(uint256 paymentId) external;
    
    // 领取分红
    function claimDividend(uint256 paymentId) external;
    
    // 查询分红
    function getDividend(address investor, uint256 paymentId) 
        external view returns (uint256);
}
```

### 3.3 事件定义

```solidity
// SecurityToken事件
event Transfer(address indexed from, address indexed to, uint256 value);
event WhitelistAdded(address indexed investor, InvestorType investorType);
event LockupSet(address indexed investor, uint256 lockupEnd);
event TokenPaused();
event TokenUnpaused();

// DividendDistributor事件
event DividendCreated(uint256 indexed paymentId, uint256 totalAmount);
event DividendDistributed(uint256 indexed paymentId);
event DividendClaimed(address indexed investor, uint256 amount);
```

---

## 📚 参考资源

- [InvestaX GitHub](https://github.com/investax)
- [ERC20标准](https://eips.ethereum.org/EIPS/eip-20)
- [Algorand ASA](https://developer.algorand.org/docs/get-details/asa/)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 10:32:00 CST
