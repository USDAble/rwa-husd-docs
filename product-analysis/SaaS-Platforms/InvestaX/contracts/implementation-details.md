# InvestaX 实现细节分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 10:33:00 CST

---

## 📑 目录

1. [关键函数实现](#1-关键函数实现)
2. [合规检查](#2-合规检查)
3. [分红机制](#3-分红机制)
4. [Gas优化](#4-gas优化)

---

## 1. 关键函数实现

### 1.1 转账函数（带合规检查）

```solidity
function transfer(address to, uint256 amount) 
    public 
    override 
    returns (bool) 
{
    // 1. 合规检查
    (bool canTransfer, string memory reason) = _canTransfer(msg.sender, to, amount);
    require(canTransfer, reason);
    
    // 2. 执行转账
    _transfer(msg.sender, to, amount);
    
    // 3. 更新投资者信息
    _updateInvestorInfo(to, amount);
    
    return true;
}

function _canTransfer(address from, address to, uint256 amount) 
    internal 
    view 
    returns (bool, string memory) 
{
    // 检查白名单
    if (!investors[to].isWhitelisted) {
        return (false, "Recipient not whitelisted");
    }
    
    // 检查锁定期
    if (block.timestamp < investors[from].lockupEnd) {
        return (false, "Tokens are locked");
    }
    
    // 检查代币状态
    if (tokenInfo.status != TokenStatus.Active) {
        return (false, "Token not active");
    }
    
    // 检查余额
    if (balanceOf(from) < amount) {
        return (false, "Insufficient balance");
    }
    
    return (true, "");
}
```

### 1.2 白名单管理

```solidity
function addToWhitelist(
    address investor,
    InvestorType investorType,
    string memory jurisdiction
) external onlyAdmin {
    require(investor != address(0), "Invalid address");
    
    investors[investor] = InvestorInfo({
        isWhitelisted: true,
        balance: 0,
        lockupEnd: 0,
        jurisdiction: jurisdiction,
        investorType: investorType
    });
    
    emit WhitelistAdded(investor, investorType);
}

function removeFromWhitelist(address investor) external onlyAdmin {
    require(investors[investor].isWhitelisted, "Not whitelisted");
    investors[investor].isWhitelisted = false;
    
    emit WhitelistRemoved(investor);
}
```

---

## 2. 合规检查

### 2.1 多层合规验证

```solidity
contract ComplianceModule {
    // 地域限制
    mapping(string => bool) public allowedJurisdictions;
    
    // 投资者类型限制
    mapping(InvestorType => uint256) public maxInvestment;
    
    function checkCompliance(
        address investor,
        uint256 amount
    ) external view returns (bool, string memory) {
        InvestorInfo memory info = investors[investor];
        
        // 1. 检查地域
        if (!allowedJurisdictions[info.jurisdiction]) {
            return (false, "Jurisdiction not allowed");
        }
        
        // 2. 检查投资限额
        uint256 newBalance = info.balance + amount;
        if (newBalance > maxInvestment[info.investorType]) {
            return (false, "Exceeds investment limit");
        }
        
        // 3. 检查KYC状态
        if (!_isKYCValid(investor)) {
            return (false, "KYC expired");
        }
        
        return (true, "");
    }
}
```

---

## 3. 分红机制

### 3.1 分红分配

```solidity
function distributeDividends(uint256 paymentId) external onlyAdmin {
    DividendPayment storage payment = dividendPayments[paymentId];
    require(payment.status == PaymentStatus.Pending, "Already distributed");
    
    // 1. 获取快照
    address[] memory holders = _getHoldersAtDate(payment.recordDate);
    
    // 2. 计算每个持有人的分红
    for (uint256 i = 0; i < holders.length; i++) {
        address holder = holders[i];
        uint256 balance = _getBalanceAtDate(holder, payment.recordDate);
        uint256 dividend = (payment.totalAmount * balance) / tokenInfo.totalSupply;
        
        investorDividends[paymentId][holder] = InvestorDividend({
            amount: dividend,
            claimed: false,
            claimDate: 0
        });
    }
    
    payment.status = PaymentStatus.Distributed;
    emit DividendDistributed(paymentId);
}

function claimDividend(uint256 paymentId) external {
    InvestorDividend storage dividend = investorDividends[paymentId][msg.sender];
    require(dividend.amount > 0, "No dividend");
    require(!dividend.claimed, "Already claimed");
    
    dividend.claimed = true;
    dividend.claimDate = block.timestamp;
    
    // 转账分红
    IERC20(dividendPayments[paymentId].currency).transfer(
        msg.sender,
        dividend.amount
    );
    
    emit DividendClaimed(msg.sender, dividend.amount);
}
```

---

## 4. Gas优化

### 4.1 批量操作

```solidity
function batchAddToWhitelist(
    address[] calldata investors,
    InvestorType[] calldata types
) external onlyAdmin {
    require(investors.length == types.length, "Length mismatch");
    
    for (uint256 i = 0; i < investors.length;) {
        _addToWhitelist(investors[i], types[i]);
        unchecked { i++; }
    }
}
```

### 4.2 存储优化

```solidity
// 使用packed storage
struct InvestorInfoPacked {
    uint96 balance;          // 96 bits
    uint32 lockupEnd;        // 32 bits
    uint8 investorType;      // 8 bits
    bool isWhitelisted;      // 8 bits
    // Total: 144 bits (fits in 2 slots)
}
```

---

## 📚 参考资源

- [InvestaX文档](https://docs.investax.io)
- [Solidity优化](https://docs.soliditylang.org/en/latest/internals/optimiser.html)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 10:33:00 CST
