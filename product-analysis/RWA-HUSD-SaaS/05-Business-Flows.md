# RWA-HUSD SaaS 平台业务流程

**文档版本**: v1.0
**创建时间**: 2025-10-11 09:20:00 CST
**文档类型**: 业务流程设计

---

## 📑 目录

1. [机构入驻流程](#1-机构入驻流程)
2. [资产上链流程](#2-资产上链流程)
3. [投资者购买流程](#3-投资者购买流程)
4. [租金分红流程](#4-租金分红流程)
5. [二级市场交易流程](#5-二级市场交易流程)
6. [代币赎回流程](#6-代币赎回流程)

---

## 1. 机构入驻流程

### 1.1 流程图

```mermaid
sequenceDiagram
    participant Admin as 机构管理员
    participant Frontend as 前端
    participant Backend as 后端
    participant KYC as KYC提供商
    participant Blockchain as 区块链
    participant Email as 邮件服务

    Admin->>Frontend: 1. 提交机构注册信息
    Frontend->>Backend: 2. POST /institutions/register
    Backend->>Backend: 3. 创建机构记录（status: pending_kyc）
    Backend->>KYC: 4. 发起企业KYC验证
    Backend-->>Frontend: 5. 返回机构ID
    Frontend-->>Admin: 6. 显示"等待KYC审核"

    KYC->>Backend: 7. KYC审核结果（webhook）
    Backend->>Backend: 8. 更新机构状态（approved/rejected）
    Backend->>Email: 9. 发送审核结果邮件
    Email-->>Admin: 10. 收到邮件通知

    Admin->>Frontend: 11. 质押ABLE代币
    Frontend->>Blockchain: 12. 调用StakingAuthorization.stake()
    Blockchain-->>Frontend: 13. 返回交易哈希
    Frontend->>Backend: 14. POST /institutions/:id/stake
    Backend->>Blockchain: 15. 验证交易
    Backend->>Backend: 16. 计算授权额度（1 ABLE = 1 USD）
    Backend-->>Frontend: 17. 返回授权额度
    Frontend-->>Admin: 18. 显示授权额度
```

### 1.2 关键步骤

1. **机构注册**: 提交企业信息（名称、注册号、法人代表等）
2. **KYC 验证**: 集成 Onfido/Jumio 进行企业资质审核
3. **质押 ABLE**: 质押 ABLE 代币获得资产上链授权
4. **授权额度**: 1 ABLE = 1 USD 授权额度
5. **邮件通知**: 审核结果通过邮件通知

---

## 2. 资产上链流程

### 2.1 流程图

```mermaid
sequenceDiagram
    participant Operator as 机构操作员
    participant Frontend as 前端
    participant Backend as 后端
    participant Storage as 对象存储
    participant Blockchain as 区块链
    participant TheGraph as The Graph

    Operator->>Frontend: 1. 创建资产（填写信息）
    Operator->>Frontend: 2. 上传资产文档
    Frontend->>Storage: 3. 上传文档到S3
    Storage-->>Frontend: 4. 返回文档URL
    Frontend->>Backend: 5. POST /assets/create
    Backend->>Backend: 6. 检查授权额度
    Backend->>Backend: 7. 创建资产记录（status: draft）
    Backend-->>Frontend: 8. 返回资产ID

    Operator->>Frontend: 9. 提交审核
    Frontend->>Backend: 10. POST /assets/:id/submit
    Backend->>Backend: 11. 更新状态（pending_review）
    Backend-->>Frontend: 12. 返回成功

    Admin->>Frontend: 13. 审核资产
    Frontend->>Backend: 14. POST /assets/:id/approve
    Backend->>Backend: 15. 更新状态（approved）

    Operator->>Frontend: 16. 部署资产上链
    Frontend->>Backend: 17. POST /assets/:id/deploy
    Backend->>Blockchain: 18. 调用PropertyTokenFactory.createPropertyToken()
    Blockchain-->>Backend: 19. 返回交易哈希
    Backend->>Backend: 20. 更新状态（deploying）
    Backend-->>Frontend: 21. 返回交易哈希

    Blockchain->>TheGraph: 22. 发出PropertyTokenCreated事件
    TheGraph->>Backend: 23. 推送事件
    Backend->>Backend: 24. 更新合约地址和状态（deployed）
    Backend->>Backend: 25. 扣除授权额度
```

### 2.2 关键步骤

1. **创建资产**: 填写资产信息（名称、符号、总量、价格等）
2. **上传文档**: 上传产权证明、估值报告、法律意见书
3. **检查额度**: 验证机构是否有足够的授权额度
4. **审核流程**: 平台审核资产信息和文档
5. **部署上链**: 调用 PropertyTokenFactory 创建代币
6. **扣除额度**: 扣除对应的授权额度

---

## 3. 投资者购买流程

### 3.1 首发购买流程

```mermaid
sequenceDiagram
    participant Investor as 投资者
    participant Frontend as 前端
    participant Backend as 后端
    participant Blockchain as 区块链
    participant TheGraph as The Graph

    Investor->>Frontend: 1. 选择资产，输入购买数量
    Frontend->>Backend: 2. GET /assets/:id
    Backend-->>Frontend: 3. 返回资产详情（价格、状态）
    Frontend->>Frontend: 4. 计算总价

    Investor->>Frontend: 5. 确认购买
    Frontend->>Blockchain: 6. 调用PropertyToken.buyTokens()
    Blockchain-->>Frontend: 7. 返回交易哈希
    Frontend->>Backend: 8. POST /trades/buy
    Backend->>Backend: 9. 创建交易记录（status: pending）
    Backend-->>Frontend: 10. 返回交易ID

    Blockchain->>TheGraph: 11. 发出TokensPurchased事件
    TheGraph->>Backend: 12. 推送事件
    Backend->>Backend: 13. 更新交易状态（confirmed）
    Backend->>Backend: 14. 更新用户持仓
    Backend->>Email: 15. 发送购买成功邮件
```

### 3.2 二级市场购买流程

见 [5. 二级市场交易流程](#5-二级市场交易流程)

---

## 4. 租金分红流程

### 4.1 流程图

```mermaid
sequenceDiagram
    participant Owner as 房产所有者
    participant Frontend as 前端
    participant Backend as 后端
    participant Blockchain as 区块链
    participant TheGraph as The Graph
    participant Investors as 投资者

    Owner->>Frontend: 1. 充值租金
    Frontend->>Blockchain: 2. 调用RentCustodyContract.depositRent()
    Blockchain-->>Frontend: 3. 返回交易哈希
    Frontend->>Backend: 4. POST /dividends/deposit
    Backend->>Backend: 5. 创建充值记录

    Blockchain->>TheGraph: 6. 发出RentDeposited事件
    TheGraph->>Backend: 7. 推送事件
    Backend->>Backend: 8. 更新充值状态

    Owner->>Frontend: 9. 触发分红
    Frontend->>Blockchain: 10. 调用RentCustodyContract.distributeRent()
    Blockchain->>Blockchain: 11. 批量分红（最多1000人）
    Blockchain-->>Frontend: 12. 返回交易哈希

    Blockchain->>TheGraph: 13. 发出RentDistributed事件
    TheGraph->>Backend: 14. 推送事件
    Backend->>Backend: 15. 创建分红记录
    Backend->>Email: 16. 发送分红通知邮件
    Email-->>Investors: 17. 收到分红通知
```

### 4.2 关键步骤

1. **充值租金**: 房产所有者充值租金到 RentCustodyContract
2. **批量分红**: 调用 distributeRent()批量分红（最多 1000 人）
3. **事件监听**: 监听 RentDistributed 事件，更新数据库
4. **邮件通知**: 发送分红通知邮件给所有投资者

---

## 5. 二级市场交易流程

### 5.1 流程图

```mermaid
sequenceDiagram
    participant Seller as 卖家
    participant Buyer as 买家
    participant Frontend as 前端
    participant Backend as 后端
    participant Blockchain as 区块链
    participant TheGraph as The Graph

    Seller->>Frontend: 1. 创建卖单
    Frontend->>Blockchain: 2. 调用TradeContract.createSellOrder()
    Blockchain-->>Frontend: 3. 返回交易哈希
    Frontend->>Backend: 4. POST /market/orders/sell
    Backend->>Backend: 5. 创建订单记录（status: pending）

    Blockchain->>TheGraph: 6. 发出OrderCreated事件
    TheGraph->>Backend: 7. 推送事件
    Backend->>Backend: 8. 更新订单状态（active）

    Buyer->>Frontend: 9. 查看订单簿
    Frontend->>Backend: 10. GET /market/orderbook/:assetId
    Backend-->>Frontend: 11. 返回买卖订单列表

    Buyer->>Frontend: 12. 撮合卖单
    Frontend->>Blockchain: 13. 调用TradeContract.fillSellOrder()
    Blockchain-->>Frontend: 14. 返回交易哈希
    Frontend->>Backend: 15. POST /market/orders/:id/buy
    Backend->>Backend: 16. 创建交易记录（status: pending）

    Blockchain->>TheGraph: 17. 发出OrderFilled事件
    TheGraph->>Backend: 18. 推送事件
    Backend->>Backend: 19. 更新订单状态（filled）
    Backend->>Backend: 20. 更新交易状态（confirmed）
    Backend->>Backend: 21. 更新买卖双方持仓
```

### 5.2 关键步骤

1. **创建卖单**: 卖家创建卖单，锁定代币
2. **订单簿**: 买家查看订单簿，选择合适的卖单
3. **撮合交易**: 买家撮合卖单，完成交易
4. **手续费**: 平台收取 0.1%手续费
5. **更新持仓**: 更新买卖双方的持仓

---

## 6. 代币赎回流程

### 6.1 流程图

```mermaid
sequenceDiagram
    participant Investor as 投资者
    participant Frontend as 前端
    participant Backend as 后端
    participant Blockchain as 区块链
    participant TheGraph as The Graph

    Investor->>Frontend: 1. 发起赎回请求
    Frontend->>Backend: 2. GET /assets/:id/redemption-info
    Backend->>Blockchain: 3. 查询赎回价格和手续费
    Backend-->>Frontend: 4. 返回赎回信息

    Investor->>Frontend: 5. 确认赎回
    Frontend->>Blockchain: 6. 调用RedemptionManager.requestRedemption()
    Blockchain-->>Frontend: 7. 返回交易哈希
    Frontend->>Backend: 8. POST /trades/redeem
    Backend->>Backend: 9. 创建赎回记录（status: pending）

    Blockchain->>TheGraph: 10. 发出RedemptionRequested事件
    TheGraph->>Backend: 11. 推送事件
    Backend->>Backend: 12. 更新赎回状态（confirmed）
    Backend->>Backend: 13. 更新用户持仓
    Backend->>Email: 14. 发送赎回成功邮件
```

### 6.2 关键步骤

1. **查询信息**: 查询赎回价格和手续费（0.1%）
2. **发起赎回**: 调用 RedemptionManager.requestRedemption()
3. **公平机制**: 使用全局快照确保公平赎回
4. **更新持仓**: 扣除用户持仓，返还资金
5. **邮件通知**: 发送赎回成功邮件

---

**文档维护**: RWA-HUSD 技术团队
**联系方式**: tech@rwa-husd.com
**最后更新**: 2025-10-11 09:20:00 CST
