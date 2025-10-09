# Securitize 核心合约分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 10:38:00 CST  
**框架**: DS Protocol (Ethereum, Polygon)

---

## 📑 目录

1. [合约列表](#1-合约列表)
2. [数据结构](#2-数据结构)
3. [主要接口](#3-主要接口)

---

## 1. 合约列表

### 1.1 DS Protocol核心合约

| 合约名称 | 职责 | 代码行数 |
|---------|------|---------|
| **DSRegistry.sol** | 投资者注册表 | ~600行 |
| **DSToken.sol** | DS代币合约 | ~800行 |
| **DSService.sol** | 服务合约 | ~500行 |
| **DSCompliance.sol** | 合规合约 | ~700行 |
| **RegulationD.sol** | Reg D规则 | ~400行 |
| **RegulationS.sol** | Reg S规则 | ~350行 |

---

## 2. 数据结构

### 2.1 DSToken结构

```solidity
contract DSToken is ERC20 {
    struct TokenInfo {
        string name;
        string symbol;
        uint256 totalSupply;
        address issuer;
        uint256 issuanceDate;
        TokenType tokenType;
        TokenStatus status;
    }
    
    struct InvestorInfo {
        bool isVerified;
        InvestorType investorType;
        string jurisdiction;
        uint256 lockupEnd;
        uint256 investmentLimit;
        bool isUSPerson;
    }
    
    enum TokenType {
        Equity,
        Debt,
        Fund,
        RealEstate
    }
    
    enum TokenStatus {
        Active,
        Paused,
        Matured
    }
    
    enum InvestorType {
        Retail,
        Accredited,
        QIB
    }
    
    mapping(address => InvestorInfo) public investors;
    TokenInfo public tokenInfo;
}
```

### 2.2 Compliance结构

```solidity
struct ComplianceRule {
    uint256 ruleId;
    RuleType ruleType;
    bool isActive;
    bytes parameters;
}

struct TransferRestriction {
    uint256 lockupEnd;
    uint256 maxTransferAmount;
    bool canTransferToRetail;
    string[] allowedJurisdictions;
}

enum RuleType {
    RegulationD,
    RegulationS,
    RegulationA,
    RegulationCF,
    Custom
}
```

---

## 3. 主要接口

### 3.1 DSToken接口

```solidity
interface IDSToken {
    // ERC20标准
    function transfer(address to, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
    
    // DS扩展
    function canTransfer(
        address from,
        address to,
        uint256 amount
    ) external view returns (bool, string memory);
    
    function isVerified(address investor) external view returns (bool);
    
    function getInvestorType(address investor) 
        external view returns (InvestorType);
    
    // 管理功能
    function verifyInvestor(
        address investor,
        InvestorType investorType,
        string memory jurisdiction
    ) external;
    
    function setLockup(address investor, uint256 lockupEnd) external;
    
    function pause() external;
    function unpause() external;
}
```

### 3.2 DSCompliance接口

```solidity
interface IDSCompliance {
    // 添加合规规则
    function addRule(
        RuleType ruleType,
        bytes memory parameters
    ) external returns (uint256 ruleId);
    
    // 检查合规
    function checkCompliance(
        address from,
        address to,
        uint256 amount
    ) external view returns (bool, string memory);
    
    // 更新规则
    function updateRule(uint256 ruleId, bytes memory parameters) external;
    
    // 禁用规则
    function disableRule(uint256 ruleId) external;
}
```

### 3.3 DSService接口

```solidity
interface IDSService {
    // 分红
    function distributeDividend(
        uint256 totalAmount,
        address currency
    ) external returns (uint256 dividendId);
    
    // 赎回
    function redeemTokens(
        address investor,
        uint256 amount
    ) external returns (uint256 redemptionId);
    
    // 投票
    function createProposal(
        string memory description,
        uint256 votingPeriod
    ) external returns (uint256 proposalId);
    
    function vote(uint256 proposalId, bool support) external;
}
```

### 3.4 事件定义

```solidity
// DSToken事件
event Transfer(address indexed from, address indexed to, uint256 value);
event InvestorVerified(address indexed investor, InvestorType investorType);
event LockupSet(address indexed investor, uint256 lockupEnd);
event TokenPaused();
event TokenUnpaused();

// DSCompliance事件
event RuleAdded(uint256 indexed ruleId, RuleType ruleType);
event RuleUpdated(uint256 indexed ruleId);
event ComplianceCheckFailed(address indexed from, address indexed to, string reason);

// DSService事件
event DividendDistributed(uint256 indexed dividendId, uint256 totalAmount);
event TokensRedeemed(uint256 indexed redemptionId, address indexed investor, uint256 amount);
event ProposalCreated(uint256 indexed proposalId, string description);
event VoteCast(uint256 indexed proposalId, address indexed voter, bool support);
```

---

## 📚 参考资源

- [DS Protocol GitHub](https://github.com/securitize-io/DSProtocol)
- [DS Protocol文档](https://docs.securitize.io/ds-protocol)
- [ERC20标准](https://eips.ethereum.org/EIPS/eip-20)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 10:38:00 CST
