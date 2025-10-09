# RealT 实现细节分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 10:45:00 CST

---

## 📑 目录

1. [关键函数实现](#1-关键函数实现)
2. [租金分配](#2-租金分配)
3. [RMM流动性池](#3-rmm流动性池)
4. [Gas优化](#4-gas优化)

---

## 1. 关键函数实现

### 1.1 每日租金计算

```solidity
function calculateDailyRent(address holder, uint256 date) 
    public 
    view 
    returns (uint256) 
{
    // 1. 获取持有人余额
    uint256 balance = balanceOf(holder);
    if (balance == 0) return 0;
    
    // 2. 计算每日租金
    uint256 annualRent = propertyInfo.annualRent;
    uint256 dailyRent = annualRent / 365;
    
    // 3. 扣除费用
    uint256 expenses = _calculateDailyExpenses();
    uint256 netRent = dailyRent > expenses ? dailyRent - expenses : 0;
    
    // 4. 计算持有人份额
    uint256 holderRent = (netRent * balance) / tokenInfo.totalSupply;
    
    return holderRent;
}

function _calculateDailyExpenses() internal view returns (uint256) {
    uint256 dailyTax = propertyInfo.propertyTax / 365;
    uint256 dailyInsurance = propertyInfo.insurance / 365;
    uint256 dailyMaintenance = propertyInfo.maintenance / 365;
    uint256 dailyManagement = (propertyInfo.annualRent * propertyInfo.managementFee) / 10000 / 365;
    
    return dailyTax + dailyInsurance + dailyMaintenance + dailyManagement;
}
```

### 1.2 自动分红

```solidity
function distributeDailyRent(uint256 date) external onlyAdmin {
    require(!dailyRents[date].finalized, "Already finalized");
    require(block.timestamp >= date + 1 days, "Too early");
    
    // 1. 获取所有持有人
    address[] memory holders = _getAllHolders();
    uint256 totalDistributed = 0;
    
    // 2. 分配给每个持有人
    for (uint256 i = 0; i < holders.length; i++) {
        address holder = holders[i];
        uint256 amount = calculateDailyRent(holder, date);
        
        if (amount > 0) {
            // 转账xDai
            (bool success, ) = payable(holder).call{value: amount}("");
            require(success, "Transfer failed");
            
            dailyRents[date].holderRents[holder] = amount;
            totalDistributed += amount;
            
            emit RentDistributed(holder, amount, date);
        }
    }
    
    // 3. 更新记录
    dailyRents[date].distributed = totalDistributed;
    dailyRents[date].finalized = true;
    
    emit DailyRentFinalized(date, totalDistributed);
}
```

---

## 2. 租金分配

### 2.1 批量分配优化

```solidity
function batchDistributeRent(
    uint256[] calldata dates
) external onlyAdmin {
    for (uint256 i = 0; i < dates.length;) {
        distributeDailyRent(dates[i]);
        unchecked { i++; }
    }
}
```

### 2.2 租金历史查询

```solidity
function getRentHistory(
    address holder,
    uint256 startDate,
    uint256 endDate
) external view returns (uint256[] memory dates, uint256[] memory amounts) {
    uint256 days = (endDate - startDate) / 1 days;
    dates = new uint256[](days);
    amounts = new uint256[](days);
    
    for (uint256 i = 0; i < days; i++) {
        uint256 date = startDate + (i * 1 days);
        dates[i] = date;
        amounts[i] = dailyRents[date].holderRents[holder];
    }
    
    return (dates, amounts);
}
```

---

## 3. RMM流动性池

### 3.1 添加流动性

```solidity
function addLiquidity(
    uint256 tokenAmount,
    uint256 xDaiAmount
) external returns (uint256 liquidity) {
    require(tokenAmount > 0 && xDaiAmount > 0, "Invalid amounts");
    
    // 1. 转入代币
    realToken.transferFrom(msg.sender, address(this), tokenAmount);
    
    // 2. 转入xDai
    require(msg.value == xDaiAmount, "Incorrect xDai amount");
    
    // 3. 计算流动性代币
    if (totalSupply == 0) {
        liquidity = sqrt(tokenAmount * xDaiAmount);
    } else {
        liquidity = min(
            (tokenAmount * totalSupply) / tokenReserve,
            (xDaiAmount * totalSupply) / xDaiReserve
        );
    }
    
    // 4. 铸造LP代币
    _mint(msg.sender, liquidity);
    
    // 5. 更新储备
    tokenReserve += tokenAmount;
    xDaiReserve += xDaiAmount;
    
    emit LiquidityAdded(msg.sender, tokenAmount, xDaiAmount, liquidity);
    
    return liquidity;
}
```

### 3.2 交换代币

```solidity
function swapTokenForxDai(uint256 tokenAmount) 
    external 
    returns (uint256 xDaiAmount) 
{
    require(tokenAmount > 0, "Invalid amount");
    
    // 1. 计算输出金额（含0.3%手续费）
    uint256 tokenAmountWithFee = tokenAmount * 997;
    xDaiAmount = (tokenAmountWithFee * xDaiReserve) / 
                 (tokenReserve * 1000 + tokenAmountWithFee);
    
    require(xDaiAmount > 0, "Insufficient output");
    
    // 2. 转入代币
    realToken.transferFrom(msg.sender, address(this), tokenAmount);
    
    // 3. 转出xDai
    payable(msg.sender).transfer(xDaiAmount);
    
    // 4. 更新储备
    tokenReserve += tokenAmount;
    xDaiReserve -= xDaiAmount;
    
    emit Swap(msg.sender, tokenAmount, 0, 0, xDaiAmount);
    
    return xDaiAmount;
}
```

---

## 4. Gas优化

### 4.1 使用Gnosis Chain

**Gas费对比**：
```solidity
// Ethereum: ~$5-50/交易
// Gnosis Chain: ~$0.001/交易

// 每日分红成本（100个持有人）：
// Ethereum: $500-5000/天
// Gnosis Chain: $0.1/天
```

### 4.2 批量操作

```solidity
function batchTransfer(
    address[] calldata recipients,
    uint256[] calldata amounts
) external {
    require(recipients.length == amounts.length, "Length mismatch");
    
    for (uint256 i = 0; i < recipients.length;) {
        _transfer(msg.sender, recipients[i], amounts[i]);
        unchecked { i++; }
    }
}
```

---

## 📚 参考资源

- [RealT文档](https://docs.realt.co)
- [Gnosis Chain](https://docs.gnosischain.com)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 10:45:00 CST
