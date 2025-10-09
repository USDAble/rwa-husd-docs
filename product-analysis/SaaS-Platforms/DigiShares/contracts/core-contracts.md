# DigiShares 核心合约分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 10:50:00 CST  
**框架**: Multi-Chain (Ethereum, Polygon, BSC, Fantom, Polymesh)

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
| **AssetToken.sol** | 资产代币 | ~600行 |
| **WhitelistManager.sol** | 白名单管理 | ~400行 |
| **PaymentProcessor.sol** | 支付处理 | ~500行 |
| **DividendDistributor.sol** | 分红分配 | ~450行 |
| **ComplianceModule.sol** | 合规模块 | ~550行 |

---

## 2. 数据结构

### 2.1 AssetToken结构

```solidity
contract AssetToken is ERC20 {
    struct TokenInfo {
        string assetName;
        AssetType assetType;
        uint256 totalSupply;
        uint256 tokenPrice;
        address issuer;
        uint256 issuanceDate;
        TokenStatus status;
    }
    
    struct InvestorInfo {
        bool isWhitelisted;
        InvestorType investorType;
        string jurisdiction;
        uint256 investmentAmount;
        uint256 lockupEnd;
        PaymentMethod preferredPayment;
    }
    
    enum AssetType {
        RealEstate,
        PrivateEquity,
        Art,
        Bonds,
        Funds,
        Commodities
    }
    
    enum TokenStatus {
        Active,
        Paused,
        Matured
    }
    
    enum InvestorType {
        Retail,
        Accredited,
        Institutional
    }
    
    enum PaymentMethod {
        Crypto,
        CreditCard,
        BankTransfer,
        PayPal
    }
    
    mapping(address => InvestorInfo) public investors;
    TokenInfo public tokenInfo;
}
```

### 2.2 Payment结构

```solidity
struct Payment {
    uint256 paymentId;
    address investor;
    uint256 amount;
    PaymentMethod method;
    PaymentStatus status;
    uint256 timestamp;
}

struct Dividend {
    uint256 dividendId;
    uint256 totalAmount;
    uint256 paymentDate;
    address currency;
    DividendStatus status;
}

enum PaymentStatus {
    Pending,
    Completed,
    Failed,
    Refunded
}

enum DividendStatus {
    Pending,
    Distributed,
    Completed
}
```

---

## 3. 主要接口

### 3.1 AssetToken接口

```solidity
interface IAssetToken {
    // ERC20标准
    function transfer(address to, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
    
    // 白名单管理
    function addToWhitelist(
        address investor,
        InvestorType investorType,
        string memory jurisdiction
    ) external;
    
    function removeFromWhitelist(address investor) external;
    
    // 转账检查
    function canTransfer(address from, address to, uint256 amount) 
        external view returns (bool, string memory);
    
    // 资产信息
    function getTokenInfo() external view returns (TokenInfo memory);
    function getInvestorInfo(address investor) 
        external view returns (InvestorInfo memory);
}
```

### 3.2 PaymentProcessor接口

```solidity
interface IPaymentProcessor {
    // 处理支付
    function processPayment(
        address investor,
        uint256 amount,
        PaymentMethod method
    ) external returns (uint256 paymentId);
    
    // 查询支付
    function getPayment(uint256 paymentId) 
        external view returns (Payment memory);
    
    // 退款
    function refundPayment(uint256 paymentId) external;
}
```

### 3.3 DividendDistributor接口

```solidity
interface IDividendDistributor {
    // 创建分红
    function createDividend(
        uint256 totalAmount,
        address currency
    ) external returns (uint256 dividendId);
    
    // 分配分红
    function distributeDividend(uint256 dividendId) external;
    
    // 领取分红
    function claimDividend(uint256 dividendId) external;
    
    // 查询分红
    function getDividend(address investor, uint256 dividendId) 
        external view returns (uint256);
}
```

### 3.4 事件定义

```solidity
// AssetToken事件
event Transfer(address indexed from, address indexed to, uint256 value);
event WhitelistAdded(address indexed investor, InvestorType investorType);
event WhitelistRemoved(address indexed investor);
event TokenPaused();
event TokenUnpaused();

// PaymentProcessor事件
event PaymentProcessed(uint256 indexed paymentId, address indexed investor, uint256 amount);
event PaymentFailed(uint256 indexed paymentId, string reason);
event PaymentRefunded(uint256 indexed paymentId);

// DividendDistributor事件
event DividendCreated(uint256 indexed dividendId, uint256 totalAmount);
event DividendDistributed(uint256 indexed dividendId);
event DividendClaimed(address indexed investor, uint256 amount);
```

---

## 📚 参考资源

- [DigiShares GitHub](https://github.com/digishares)
- [ERC20标准](https://eips.ethereum.org/EIPS/eip-20)
- [Polymesh](https://polymesh.network)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 10:50:00 CST
