# RealT 核心合约分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 10:44:00 CST  
**框架**: Ethereum + Gnosis Chain

---

## 📑 目录

1. [合约列表](#1-合约列表)
2. [数据结构](#2-数据结构)
3. [主要接口](#3-主要接口)

---

## 1. 合约列表

### 1.1 核心合约

| 合约名称 | 职责 | 代码行数 |
|---------|------|---------|
| **RealToken.sol** | 房产代币 | ~400行 |
| **RentDistributor.sol** | 租金分配 | ~500行 |
| **PropertyRegistry.sol** | 房产注册表 | ~300行 |
| **RMMPool.sol** | 流动性池 | ~600行 |

---

## 2. 数据结构

### 2.1 RealToken结构

```solidity
contract RealToken is ERC20 {
    struct TokenInfo {
        string propertyAddress;
        uint256 totalSupply;
        uint256 tokenPrice;
        uint256 issuanceDate;
        TokenStatus status;
    }
    
    struct PropertyInfo {
        string llcName;
        uint256 purchasePrice;
        uint256 propertyValue;
        uint256 annualRent;
        uint256 propertyTax;
        uint256 insurance;
        uint256 maintenance;
        uint256 managementFee;
    }
    
    struct RentInfo {
        uint256 lastDistribution;
        uint256 totalDistributed;
        uint256 accumulatedRent;
    }
    
    enum TokenStatus {
        Active,
        Paused,
        Sold
    }
    
    TokenInfo public tokenInfo;
    PropertyInfo public propertyInfo;
    RentInfo public rentInfo;
}
```

### 2.2 Rent结构

```solidity
struct DailyRent {
    uint256 date;
    uint256 totalAmount;
    uint256 distributed;
    mapping(address => uint256) holderRents;
    bool finalized;
}

struct HolderRentHistory {
    uint256 totalReceived;
    uint256 lastClaimDate;
    uint256[] claimDates;
}
```

---

## 3. 主要接口

### 3.1 RealToken接口

```solidity
interface IRealToken {
    // ERC20标准
    function transfer(address to, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
    
    // 租金查询
    function getDailyRent(address holder) external view returns (uint256);
    function getTotalRentReceived(address holder) external view returns (uint256);
    
    // 房产信息
    function getPropertyInfo() external view returns (PropertyInfo memory);
    function getTokenInfo() external view returns (TokenInfo memory);
}
```

### 3.2 RentDistributor接口

```solidity
interface IRentDistributor {
    // 分配租金
    function distributeDailyRent(uint256 date, uint256 amount) external;
    
    // 查询租金
    function getDailyRent(uint256 date, address holder) 
        external view returns (uint256);
    
    // 领取租金（如果未自动分配）
    function claimRent(uint256 date) external;
}
```

### 3.3 事件定义

```solidity
// RealToken事件
event Transfer(address indexed from, address indexed to, uint256 value);
event PropertyUpdated(string propertyAddress, uint256 propertyValue);
event TokenPaused();
event TokenUnpaused();

// RentDistributor事件
event RentDistributed(address indexed holder, uint256 amount, uint256 date);
event RentClaimed(address indexed holder, uint256 amount, uint256 date);
event DailyRentFinalized(uint256 date, uint256 totalAmount);
```

---

## 📚 参考资源

- [RealT GitHub](https://github.com/realtplatform)
- [ERC20标准](https://eips.ethereum.org/EIPS/eip-20)
- [Gnosis Chain](https://docs.gnosischain.com)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 10:44:00 CST
