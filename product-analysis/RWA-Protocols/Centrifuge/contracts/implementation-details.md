# Centrifuge 实现细节分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 09:44:00 CST  
**框架**: Substrate + Solidity

---

## 📑 目录

1. [关键函数实现](#1-关键函数实现)
2. [NAV计算](#2-nav计算)
3. [收益分配](#3-收益分配)
4. [Gas优化](#4-gas优化)

---

## 1. 关键函数实现

### 1.1 投资函数

```solidity
/**
 * @dev 投资到Tranche
 * @param amount 投资金额
 * @return shares 获得的份额
 */
function supply(uint256 amount) external returns (uint256 shares) {
    require(amount > 0, "Amount must be positive");
    require(status == TrancheStatus.Active, "Tranche not active");
    
    // 1. 转入DAI
    currency.transferFrom(msg.sender, address(this), amount);
    
    // 2. 计算份额
    uint256 price = sharePrice();
    shares = (amount * 10**27) / price;
    
    // 3. 铸造份额代币
    _mint(msg.sender, shares);
    
    // 4. 更新总供应
    totalSupply += shares;
    
    // 5. 触发事件
    emit Supply(msg.sender, amount, shares);
    
    return shares;
}
```

### 1.2 赎回函数

```solidity
/**
 * @dev 赎回Tranche份额
 * @param shares 赎回份额
 * @return amount 获得的金额
 */
function redeem(uint256 shares) external returns (uint256 amount) {
    require(shares > 0, "Shares must be positive");
    require(balanceOf(msg.sender) >= shares, "Insufficient shares");
    
    // 1. 计算赎回金额
    uint256 price = sharePrice();
    amount = (shares * price) / 10**27;
    
    // 2. 检查流动性
    require(reserve.balance() >= amount, "Insufficient liquidity");
    
    // 3. 销毁份额代币
    _burn(msg.sender, shares);
    
    // 4. 转出DAI
    reserve.payout(msg.sender, amount);
    
    // 5. 更新总供应
    totalSupply -= shares;
    
    // 6. 触发事件
    emit Redeem(msg.sender, shares, amount);
    
    return amount;
}
```

---

## 2. NAV计算

### 2.1 NAV计算公式

```solidity
/**
 * @dev 计算池的NAV（净资产价值）
 * @return nav 净资产价值
 */
function calcNAV() public view returns (uint256 nav) {
    uint256 totalAssetValue = 0;
    
    // 1. 计算所有资产价值
    for (uint256 i = 0; i < assets.length; i++) {
        Asset memory asset = assets[i];
        
        if (asset.status == AssetStatus.Active) {
            // 本金 + 应计利息
            uint256 interest = calcAccruedInterest(asset);
            totalAssetValue += asset.principal + interest;
        }
    }
    
    // 2. 加上储备金
    totalAssetValue += reserve.balance();
    
    // 3. 减去负债
    uint256 totalDebt = seniorTranche.debt() + juniorTranche.debt();
    
    // 4. NAV = 资产 - 负债
    nav = totalAssetValue - totalDebt;
    
    return nav;
}

/**
 * @dev 计算应计利息
 * @param asset 资产
 * @return interest 应计利息
 */
function calcAccruedInterest(Asset memory asset) 
    private 
    view 
    returns (uint256 interest) 
{
    uint256 timeDelta = block.timestamp - asset.lastUpdateTime;
    uint256 annualRate = asset.interestRate;
    
    // 利息 = 本金 * 年利率 * 时间 / 365天
    interest = (asset.principal * annualRate * timeDelta) / (10000 * 365 days);
    
    return interest;
}
```

---

## 3. 收益分配

### 3.1 分配顺序

```solidity
/**
 * @dev 分配收益
 * @param income 总收益
 */
function distributeIncome(uint256 income) external onlyAdmin {
    // 1. 支付费用
    uint256 fees = (income * feeRate) / 10000;
    reserve.payout(feeRecipient, fees);
    income -= fees;
    
    // 2. 支付Senior Tranche
    uint256 seniorDebt = seniorTranche.debt();
    uint256 seniorInterest = calcTrancheInterest(seniorTranche);
    uint256 seniorPayment = seniorDebt + seniorInterest;
    
    if (income >= seniorPayment) {
        seniorTranche.repay(seniorPayment);
        income -= seniorPayment;
    } else {
        seniorTranche.repay(income);
        income = 0;
    }
    
    // 3. 剩余归Junior Tranche
    if (income > 0) {
        juniorTranche.distribute(income);
    }
    
    emit IncomeDistributed(income, fees, seniorPayment);
}
```

---

## 4. Gas优化

### 4.1 批量操作

```solidity
/**
 * @dev 批量添加资产
 * @param assets 资产列表
 */
function batchAddAssets(Asset[] calldata assets) external onlyAdmin {
    uint256 length = assets.length;
    
    for (uint256 i = 0; i < length;) {
        _addAsset(assets[i]);
        unchecked { i++; }
    }
}
```

---

## 📚 参考资源

- [Centrifuge GitHub](https://github.com/centrifuge)
- [Tinlake文档](https://docs.centrifuge.io/getting-started/tinlake)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 09:44:00 CST
