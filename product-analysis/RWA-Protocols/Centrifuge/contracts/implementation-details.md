# Centrifuge 实现细节分析

**文档版本**: v2.0
**创建时间**: 2025-10-09 09:44:00 CST
**最后更新**: 2025-10-13 10:05:00 CST
**框架**: Substrate + Solidity

---

## 📑 目录

1. [Hub 模块实现](#1-hub模块实现)
2. [Spoke 模块实现](#2-spoke模块实现)
3. [Vaults 模块实现](#3-vaults模块实现)
4. [Tinlake 实现(历史)](#4-tinlake实现历史)
5. [NAV 计算](#5-nav计算)
6. [收益分配](#6-收益分配)
7. [Gas 优化](#7-gas优化)

---

## 1. Hub 模块实现

### 1.1 Hub 合约核心方法

#### 1.1.1 设置 BalanceSheet 管理器

```solidity
/**
 * @dev 设置BalanceSheet管理器
 * @param centrifugeId Centrifuge链上的ID
 * @param poolId 池ID
 * @param manager 管理器地址(bytes32格式)
 * @param addManager true=添加, false=移除
 */
function updateBalanceSheetManager(
    bytes32 centrifugeId,
    uint256 poolId,
    bytes32 manager,
    bool addManager
) external onlyHubAdmin {
    // 1. 验证池存在
    require(hubRegistry.poolExists(poolId), "Pool does not exist");

    // 2. 获取BalanceSheet合约
    address balanceSheet = hubRegistry.getBalanceSheet(poolId);

    // 3. 更新管理器
    if (addManager) {
        IBalanceSheet(balanceSheet).addManager(centrifugeId, manager);
        emit ManagerAdded(poolId, manager);
    } else {
        IBalanceSheet(balanceSheet).removeManager(centrifugeId, manager);
        emit ManagerRemoved(poolId, manager);
    }
}
```

---

#### 1.1.2 设置请求管理器

```solidity
/**
 * @dev 设置请求管理器
 * @param poolId 池ID
 * @param scId 份额类别ID
 * @param assetId 资产ID
 * @param manager 管理器地址(bytes32格式)
 */
function setRequestManager(
    uint256 poolId,
    uint256 scId,
    uint256 assetId,
    bytes32 manager
) external onlyHubAdmin {
    // 1. 验证池和份额类别存在
    require(hubRegistry.poolExists(poolId), "Pool does not exist");
    require(shareClassManager.scExists(poolId, scId), "Share class does not exist");

    // 2. 设置管理器
    shareClassManager.setRequestManager(poolId, scId, assetId, manager);

    emit RequestManagerSet(poolId, scId, assetId, manager);
}
```

---

### 1.2 ShareClassManager 核心方法

#### 1.2.1 提交投资/赎回请求

```solidity
/**
 * @dev 提交投资或赎回请求
 * @param poolId 池ID
 * @param scId 份额类别ID
 * @param amount 金额
 * @param requestType INVEST或REDEEM
 * @return requestId 请求ID
 */
function submitRequest(
    uint256 poolId,
    uint256 scId,
    uint256 amount,
    RequestType requestType
) external returns (uint256 requestId) {
    // 1. 验证参数
    require(amount > 0, "Amount must be positive");
    require(scExists(poolId, scId), "Share class does not exist");

    // 2. 生成请求ID
    requestId = _generateRequestId(poolId, scId, msg.sender);

    // 3. 创建请求
    Request storage request = requests[requestId];
    request.user = msg.sender;
    request.poolId = poolId;
    request.scId = scId;
    request.amount = amount;
    request.requestType = requestType;
    request.status = RequestStatus.Pending;
    request.timestamp = block.timestamp;

    // 4. 添加到当前Epoch
    uint256 currentEpoch = getCurrentEpoch(poolId, scId);
    epochRequests[poolId][scId][currentEpoch].push(requestId);

    // 5. 触发事件
    emit RequestSubmitted(requestId, poolId, scId, msg.sender, amount, requestType);

    return requestId;
}
```

---

#### 1.2.2 关闭 Epoch

```solidity
/**
 * @dev 关闭当前Epoch
 * @param poolId 池ID
 * @param scId 份额类别ID
 */
function closeEpoch(uint256 poolId, uint256 scId) external onlyAuthorized {
    // 1. 获取当前Epoch
    uint256 currentEpoch = getCurrentEpoch(poolId, scId);

    // 2. 验证Epoch状态
    require(epochStatus[poolId][scId][currentEpoch] == EpochStatus.Open, "Epoch not open");

    // 3. 计算总投资和赎回金额
    uint256[] memory requestIds = epochRequests[poolId][scId][currentEpoch];
    uint256 totalInvest = 0;
    uint256 totalRedeem = 0;

    for (uint256 i = 0; i < requestIds.length; i++) {
        Request storage request = requests[requestIds[i]];
        if (request.requestType == RequestType.INVEST) {
            totalInvest += request.amount;
        } else {
            totalRedeem += request.amount;
        }
    }

    // 4. 更新Epoch状态
    epochStatus[poolId][scId][currentEpoch] = EpochStatus.Closed;
    epochData[poolId][scId][currentEpoch] = EpochData({
        totalInvest: totalInvest,
        totalRedeem: totalRedeem,
        sharePrice: 0,  // 将在executeEpoch中设置
        closeTime: block.timestamp
    });

    // 5. 触发事件
    emit EpochClosed(poolId, scId, currentEpoch, totalInvest, totalRedeem);
}
```

---

#### 1.2.3 执行 Epoch

```solidity
/**
 * @dev 执行Epoch
 * @param poolId 池ID
 * @param scId 份额类别ID
 */
function executeEpoch(uint256 poolId, uint256 scId) external onlyAuthorized {
    // 1. 获取当前Epoch
    uint256 currentEpoch = getCurrentEpoch(poolId, scId);

    // 2. 验证Epoch状态
    require(epochStatus[poolId][scId][currentEpoch] == EpochStatus.Closed, "Epoch not closed");

    // 3. 获取当前份额价格
    uint256 sharePrice = _getSharePrice(poolId, scId);
    epochData[poolId][scId][currentEpoch].sharePrice = sharePrice;

    // 4. 处理所有请求
    uint256[] memory requestIds = epochRequests[poolId][scId][currentEpoch];
    for (uint256 i = 0; i < requestIds.length; i++) {
        _processRequest(requestIds[i], sharePrice);
    }

    // 5. 更新Epoch状态
    epochStatus[poolId][scId][currentEpoch] = EpochStatus.Executed;

    // 6. 开启新Epoch
    _startNewEpoch(poolId, scId);

    // 7. 触发事件
    emit EpochExecuted(poolId, scId, currentEpoch, sharePrice);
}

/**
 * @dev 处理单个请求
 * @param requestId 请求ID
 * @param sharePrice 份额价格
 */
function _processRequest(uint256 requestId, uint256 sharePrice) private {
    Request storage request = requests[requestId];

    if (request.requestType == RequestType.INVEST) {
        // 计算份额数量
        uint256 shares = (request.amount * 1e18) / sharePrice;

        // 铸造份额
        IShareToken(shareTokens[request.poolId][request.scId]).mint(request.user, shares);

        request.status = RequestStatus.Executed;
        request.shares = shares;
    } else {
        // 计算赎回金额
        uint256 amount = (request.amount * sharePrice) / 1e18;

        // 销毁份额
        IShareToken(shareTokens[request.poolId][request.scId]).burn(request.user, request.amount);

        // 转账
        IERC20(assets[request.poolId]).transfer(request.user, amount);

        request.status = RequestStatus.Executed;
        request.amount = amount;
    }
}
```

---

## 2. Spoke 模块实现

### 2.1 BalanceSheet 核心方法

#### 2.1.1 存款实现

```solidity
/**
 * @dev 存款到池
 * @param poolId 池ID
 * @param scId 份额类别ID
 * @param tokenAddress 代币地址
 * @param tokenType 代币类型(0=ERC20)
 * @param amount 存款金额
 */
function deposit(
    uint256 poolId,
    uint256 scId,
    address tokenAddress,
    uint256 tokenType,
    uint256 amount
) external {
    // 1. 验证参数
    require(amount > 0, "Amount must be positive");
    require(tokenType == 0, "Only ERC20 supported");

    // 2. 验证管理器授权
    require(isAuthorizedManager(poolId, msg.sender), "Not authorized");

    // 3. 转入代币
    IERC20(tokenAddress).transferFrom(msg.sender, address(this), amount);

    // 4. 更新余额
    balances[poolId][scId][tokenAddress] += amount;

    // 5. 通知Hub
    IHub(hub).notifyDeposit(poolId, scId, tokenAddress, amount);

    // 6. 触发事件
    emit Deposit(poolId, scId, tokenAddress, amount, msg.sender);
}
```

---

#### 2.1.2 提款实现

```solidity
/**
 * @dev 从池提款
 * @param poolId 池ID
 * @param scId 份额类别ID
 * @param tokenAddress 代币地址
 * @param tokenType 代币类型(0=ERC20)
 * @param receiver 接收地址
 * @param amount 提款金额
 */
function withdraw(
    uint256 poolId,
    uint256 scId,
    address tokenAddress,
    uint256 tokenType,
    address receiver,
    uint256 amount
) external {
    // 1. 验证参数
    require(amount > 0, "Amount must be positive");
    require(tokenType == 0, "Only ERC20 supported");
    require(receiver != address(0), "Invalid receiver");

    // 2. 验证管理器授权
    require(isAuthorizedManager(poolId, msg.sender), "Not authorized");

    // 3. 验证余额
    require(balances[poolId][scId][tokenAddress] >= amount, "Insufficient balance");

    // 4. 更新余额
    balances[poolId][scId][tokenAddress] -= amount;

    // 5. 转出代币
    IERC20(tokenAddress).transfer(receiver, amount);

    // 6. 通知Hub
    IHub(hub).notifyWithdraw(poolId, scId, tokenAddress, amount);

    // 7. 触发事件
    emit Withdraw(poolId, scId, tokenAddress, amount, receiver);
}
```

---

## 3. Vaults 模块实现

### 3.1 AsyncVault 核心方法

#### 3.1.1 请求存款

```solidity
/**
 * @dev 请求存款(ERC-7540)
 * @param assets 资产数量
 * @param receiver 接收地址
 * @return requestId 请求ID
 */
function requestDeposit(
    uint256 assets,
    address receiver
) external returns (uint256 requestId) {
    // 1. 验证参数
    require(assets > 0, "Assets must be positive");
    require(receiver != address(0), "Invalid receiver");

    // 2. 生成请求ID
    requestId = _generateRequestId(msg.sender, receiver);

    // 3. 转入资产到托管
    IERC20(asset).transferFrom(msg.sender, escrow, assets);

    // 4. 创建请求
    depositRequests[requestId] = DepositRequest({
        owner: msg.sender,
        receiver: receiver,
        assets: assets,
        status: RequestStatus.Pending,
        timestamp: block.timestamp
    });

    // 5. 触发事件
    emit DepositRequested(requestId, msg.sender, receiver, assets);

    return requestId;
}
```

---

#### 3.1.2 执行存款

```solidity
/**
 * @dev 执行存款(ERC-7540)
 * @param assets 资产数量
 * @param receiver 接收地址
 * @return shares 份额数量
 */
function deposit(
    uint256 assets,
    address receiver
) external returns (uint256 shares) {
    // 1. 验证请求存在
    uint256 requestId = pendingDepositRequests[msg.sender][receiver];
    require(requestId != 0, "No pending request");

    DepositRequest storage request = depositRequests[requestId];
    require(request.status == RequestStatus.Approved, "Request not approved");
    require(request.assets == assets, "Assets mismatch");

    // 2. 计算份额
    shares = convertToShares(assets);

    // 3. 从托管转入资产
    IERC20(asset).transferFrom(escrow, address(this), assets);

    // 4. 铸造份额
    _mint(receiver, shares);

    // 5. 更新请求状态
    request.status = RequestStatus.Executed;
    delete pendingDepositRequests[msg.sender][receiver];

    // 6. 触发事件
    emit Deposit(msg.sender, receiver, assets, shares);

    return shares;
}
```

---

### 3.2 SyncDepositVault 核心方法

#### 3.2.1 同步存款(ERC-4626)

```solidity
/**
 * @dev 同步存款(ERC-4626)
 * @param assets 资产数量
 * @param receiver 接收地址
 * @return shares 份额数量
 */
function deposit(
    uint256 assets,
    address receiver
) external returns (uint256 shares) {
    // 1. 验证参数
    require(assets > 0, "Assets must be positive");
    require(receiver != address(0), "Invalid receiver");

    // 2. 计算份额
    shares = convertToShares(assets);
    require(shares > 0, "Shares must be positive");

    // 3. 转入资产
    IERC20(asset).transferFrom(msg.sender, address(this), assets);

    // 4. 铸造份额
    _mint(receiver, shares);

    // 5. 触发事件
    emit Deposit(msg.sender, receiver, assets, shares);

    return shares;
}
```

---

## 4. Tinlake 实现(历史)

### 4.1 投资函数

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

## 2. NAV 计算

### 2.1 NAV 计算公式

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

## 4. Gas 优化

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

-   [Centrifuge GitHub](https://github.com/centrifuge)
-   [Tinlake 文档](https://docs.centrifuge.io/getting-started/tinlake)

---

**文档维护**: RWA-HUSD 技术团队  
**最后更新**: 2025-10-09 09:44:00 CST
