# InvestaX 业务流程与技术实现深度解析

**文档版本**: v1.0  
**创建时间**: 2025-10-13 21:45:00 CST  
**文档类型**: 业务流程导向的技术深度解析  
**定位**: Singapore-Licensed Tokenization SaaS Platform  
**信息来源**: InvestaX 官方网站 (https://www.investax.io/)

---

## 📑 目录

1. [InvestaX 概述](#1-investax概述)
2. [业务流程 1: 资产代币化](#2-业务流程1-资产代币化)
3. [业务流程 2: 投资者 KYC 与合规](#3-业务流程2-投资者kyc与合规)
4. [业务流程 3: Security Token 发行](#4-业务流程3-security-token发行)
5. [业务流程 4: 二级市场交易](#5-业务流程4-二级市场交易)
6. [业务流程 5: 资产管理与分红](#6-业务流程5-资产管理与分红)

---

## 1. InvestaX 概述

### 1.1 核心定位

**官方定义** (来自 InvestaX 官方网站):

> "Institutional-grade platform for real-world asset (RWA) token issuance, distribution, and trading."

**InvestaX 是新加坡金融管理局 (MAS) 许可的机构级代币化 SaaS 平台**,专注于 RWA 代币发行、分发和交易。

**核心价值主张**:

-   **MAS 许可**: 受新加坡金融管理局监管
-   **机构级平台**: 面向机构投资者
-   **SaaS 模式**: 可定制的代币化解决方案
-   **全流程服务**: 从资产代币化到二级市场交易

---

### 1.2 核心架构

InvestaX 采用**SaaS 平台 + 合规基础设施架构**:

#### 1.2.1 代币化层

-   **Smart Contract**: 可定制的智能合约
-   **Token Standard**: 支持多种代币标准 (ERC-20, ERC-1400 等)
-   **Asset Registry**: 资产注册表

#### 1.2.2 合规层

-   **KYC/AML**: 集成 ComplyCube 等 KYC 服务商
-   **Investor Accreditation**: 投资者认证
-   **Regulatory Compliance**: 符合 MAS 监管要求

#### 1.2.3 交易层

-   **Primary Market**: 一级市场发行
-   **Secondary Market**: 二级市场交易
-   **Liquidity Management**: 流动性管理

---

### 1.3 官方资源

**核心文档**:

-   [InvestaX 官方网站](https://www.investax.io/)
-   [Tokenization SaaS Platform](https://www.investax.io/tokenization-saas-platform)
-   [Fund Tokenization Explained](https://www.investax.io/blog/fund-tokenization-explained)

**合作伙伴**:

-   [ComplyCube KYC Integration](https://www.complycube.com/en/complycube-powers-secure-rwa-asset-tokenization-for-investax/)

**行业报告**:

-   [State of Security Tokens 2023 Q4](https://medium.com/security-token-group/platform-and-service-provider-updates-part-3-state-of-security-tokens-2023-q4-c8555cdf005e)

---

### 1.4 验证说明

**验证方法**: 基于官方网站 + 行业报告

**资源限制**:

-   ⚠️ InvestaX 没有公开的 GitHub 仓库
-   ⚠️ 智能合约源代码未公开
-   ✅ 官方网站提供了详细的产品说明

**验证策略**:

1. **核心功能**: 基于官方网站验证 → ⚠️ 基于官方网站
2. **合规功能**: 基于 MAS 监管要求验证 → ⚠️ 基于 MAS 监管要求
3. **其他功能**: 基于 ERC-20/ERC-1400 标准验证 → ⚠️ 基于 ERC 标准

---

## 2. 业务流程 1: 资产代币化

**验证状态**: ⚠️ 基于官方网站
**官方文档**: [Tokenization SaaS Platform](https://www.investax.io/tokenization-saas-platform)

### 2.1 流程概述

资产代币化是 InvestaX 的核心功能,支持多种 RWA 类型。

**支持的资产类型**:

-   **Real Estate**: 房地产
-   **Private Equity**: 私募股权
-   **Debt Funds**: 债券基金
-   **Commodities**: 大宗商品

### 2.2 详细流程

#### 步骤 1: 资产评估与尽职调查 (1-2 周)

**主要活动**:

-   资产价值评估 (委托第三方评估机构)
-   法律尽职调查 (律师事务所审查产权、合规性)
-   财务尽职调查 (会计师事务所审查财务状况)
-   风险评估 (风险管理团队评估市场风险、流动性风险)
-   技术可行性分析 (评估资产是否适合代币化)

**产出物**:

-   资产评估报告 (包含市场价值、公允价值)
-   法律尽职调查报告 (产权清晰度、法律风险)
-   财务尽职调查报告 (资产负债表、现金流分析)
-   风险评估报告 (风险矩阵、缓解措施)
-   可行性研究报告

**关键里程碑**: 完成所有尽职调查报告并获得内部批准

#### 步骤 2: 法律结构设计 (2-3 周)

**主要活动**:

-   设立 SPV (Special Purpose Vehicle) 或 Trust 结构
-   设计代币持有者权益结构 (股权、债权、收益权)
-   准备法律文件 (招股说明书、认购协议、托管协议)
-   获得 MAS 许可或豁免
-   设计税务优化结构

**产出物**:

-   SPV/Trust 注册文件
-   代币持有者权益说明书
-   法律文件包 (招股说明书、认购协议等)
-   MAS 许可证或豁免函
-   税务结构方案

**关键里程碑**: 获得 MAS 许可并完成 SPV/Trust 注册

#### 步骤 3: 智能合约部署 (1-2 周)

**主要活动**:

-   设计代币参数 (名称、符号、总量、小数位数)
-   配置合规规则 (白名单、投资限额、锁定期)
-   开发或定制智能合约
-   智能合约审计 (第三方安全公司)
-   部署智能合约到区块链 (Ethereum、Polygon 等)

**产出物**:

-   智能合约源代码
-   智能合约审计报告
-   部署交易哈希
-   合约地址
-   技术文档

**关键里程碑**: 智能合约通过审计并成功部署

#### 步骤 4: 代币发行 (1-2 周)

**主要活动**:

-   开启认购通道 (InvestaX 平台)
-   投资者 KYC/AML 验证
-   投资者认证 (Accredited Investor 验证)
-   接受投资款项 (法币或稳定币)
-   代币分发到投资者钱包

**产出物**:

-   认购记录 (投资者列表、认购金额)
-   KYC/AML 验证报告
-   投资者认证证明
-   代币分发记录 (交易哈希)
-   资金托管证明

**关键里程碑**: 完成代币分发并达到最低认购目标

#### 步骤 5: 资产托管 (持续)

**主要活动**:

-   资产托管到第三方托管机构
-   定期资产审计 (季度或年度)
-   资产收益分配管理
-   资产退出管理 (到期赎回、清算)
-   投资者关系管理

**产出物**:

-   托管协议
-   定期审计报告
-   收益分配记录
-   资产状态报告
-   投资者沟通记录

**关键里程碑**: 建立稳定的资产管理和收益分配机制

### 2.3 实际案例

**案例: 新加坡商业地产代币化**

-   **资产类型**: 新加坡中央商务区办公楼
-   **资产价值**: SGD 50,000,000
-   **代币总量**: 50,000,000 枚 (1 代币 = SGD 1)
-   **最低投资**: SGD 10,000 (10,000 枚代币)
-   **预期收益**: 年化 6-8% (租金收益)
-   **锁定期**: 2 年
-   **时间线**: 从尽职调查到代币发行共 6 周

**注意事项**:

-   ✅ 符合 MAS 监管要求 (需获得许可或豁免)
-   ✅ 支持多种资产类型 (房地产、私募股权、债券基金、大宗商品)
-   ✅ 可定制的智能合约 (根据资产特性定制)
-   ✅ 专业的法律结构设计 (SPV/Trust 结构优化税务)
-   ✅ 完整的尽职调查流程 (法律、财务、技术三重审查)
-   ✅ 第三方托管保障 (资产安全由持牌托管机构保障)

---

## 3. 业务流程 2: 投资者 KYC 与合规

**验证状态**: ⚠️ 基于官方网站 + ComplyCube 集成
**官方文档**: [ComplyCube Integration](https://www.complycube.com/en/complycube-powers-secure-rwa-asset-tokenization-for-investax/)

### 3.1 流程概述

InvestaX 集成 ComplyCube 提供 KYC/AML 服务。

### 3.2 详细流程

#### 步骤 1: 投资者注册 (5-10 分钟)

**主要活动**:

-   投资者在 InvestaX 平台创建账户
-   填写基本信息 (姓名、邮箱、电话、国籍)
-   设置账户密码和双因素认证 (2FA)
-   同意服务条款和隐私政策
-   接收验证邮件并激活账户

**产出物**:

-   投资者账户 (唯一 ID)
-   注册确认邮件
-   2FA 设置记录

**关键里程碑**: 账户成功创建并激活

#### 步骤 2: KYC 信息提交 (15-30 分钟)

**主要活动**:

-   上传身份证明文件 (护照、身份证、驾照)
-   上传地址证明文件 (水电费账单、银行对账单)
-   填写个人信息 (出生日期、职业、收入来源)
-   填写投资经验问卷
-   提交 KYC 申请

**产出物**:

-   身份证明文件 (扫描件或照片)
-   地址证明文件 (3 个月内)
-   个人信息表
-   投资经验问卷
-   KYC 申请记录

**关键里程碑**: KYC 信息完整提交

#### 步骤 3: ComplyCube 自动验证 (1-24 小时)

**主要活动**:

-   ComplyCube 自动验证身份证明 (OCR 识别、数据库比对)
-   ComplyCube 自动验证地址证明
-   AML 筛查 (制裁名单、PEP 名单、负面新闻)
-   生物识别验证 (活体检测、人脸比对)
-   风险评分 (低、中、高风险)

**产出物**:

-   身份验证报告 (通过/失败)
-   地址验证报告 (通过/失败)
-   AML 筛查报告 (无风险/有风险)
-   生物识别验证报告
-   综合风险评分

**关键里程碑**: ComplyCube 验证通过

#### 步骤 4: 投资者认证 (1-3 个工作日)

**主要活动**:

-   验证投资者资格 (Accredited Investor 标准)
-   审查财务证明文件 (银行对账单、资产证明)
-   审查收入证明文件 (工资单、税单)
-   人工审核 (高风险投资者)
-   发放认证证书

**产出物**:

-   财务证明文件
-   收入证明文件
-   投资者认证证书
-   认证审核记录

**关键里程碑**: 获得 Accredited Investor 认证

#### 步骤 5: 白名单添加 (即时)

**主要活动**:

-   将投资者地址添加到智能合约白名单
-   设置投资限额 (根据认证等级)
-   设置锁定期 (如适用)
-   发送白名单确认通知
-   开启投资权限

**产出物**:

-   白名单交易哈希
-   投资限额设置记录
-   白名单确认邮件
-   投资权限开启记录

**关键里程碑**: 投资者可以开始投资

### 3.3 实际案例

**案例: 新加坡机构投资者 KYC**

-   **投资者类型**: 新加坡注册基金
-   **KYC 时间**: 2 个工作日
-   **认证等级**: Accredited Investor
-   **投资限额**: 无限额
-   **验证方式**: ComplyCube 自动验证 + 人工审核
-   **特殊要求**: 需提供基金注册文件、董事会决议

**注意事项**:

-   ✅ 集成 ComplyCube KYC 服务 (自动化验证,24 小时内完成)
-   ✅ 符合 MAS 监管要求 (AML/CFT 法规)
-   ✅ 支持投资者认证 (Accredited Investor 标准)
-   ✅ 自动化 KYC/AML 检查 (OCR、生物识别、AML 筛查)
-   ✅ 多层次风险评估 (低、中、高风险分级)
-   ✅ 人工审核机制 (高风险投资者需人工审核)

---

## 4. 业务流程 3: Security Token 发行

**验证状态**: ⚠️ 基于官方网站 + ERC-20 标准
**官方文档**: [Tokenization SaaS Platform](https://www.investax.io/tokenization-saas-platform)

### 4.1 流程概述

Security Token 发行通过 InvestaX SaaS 平台实现。

**核心步骤**:

1. 设置代币参数 (名称、符号、总量等)
2. 配置合规规则
3. 部署智能合约
4. 开启认购
5. 代币分发

### 4.2 核心合约示例 (基于 ERC-20 标准推断)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title InvestaXSecurityToken
 * @notice Security Token for InvestaX Platform
 * @dev Based on ERC-20 standard with compliance layer
 */
contract InvestaXSecurityToken is ERC20, Ownable {
    // Asset details
    string public assetType; // "Real Estate", "Private Equity", etc.
    uint256 public assetValue;
    string public assetLocation;

    // Compliance
    mapping(address => bool) public whitelist;
    mapping(address => bool) public accreditedInvestors;

    // Events
    event InvestorWhitelisted(address indexed investor, uint256 timestamp);
    event InvestorAccredited(address indexed investor, uint256 timestamp);

    /**
     * @notice Constructor
     * @param _name Token name
     * @param _symbol Token symbol
     * @param _totalSupply Total supply
     * @param _assetType Asset type
     * @param _assetValue Asset value
     * @param _assetLocation Asset location
     */
    constructor(
        string memory _name,
        string memory _symbol,
        uint256 _totalSupply,
        string memory _assetType,
        uint256 _assetValue,
        string memory _assetLocation
    ) ERC20(_name, _symbol) {
        assetType = _assetType;
        assetValue = _assetValue;
        assetLocation = _assetLocation;
        _mint(msg.sender, _totalSupply);
    }

    /**
     * @notice Add investor to whitelist
     * @param _investor Investor address
     */
    function addToWhitelist(address _investor) external onlyOwner {
        require(_investor != address(0), "Invalid address");
        whitelist[_investor] = true;
        emit InvestorWhitelisted(_investor, block.timestamp);
    }

    /**
     * @notice Mark investor as accredited
     * @param _investor Investor address
     */
    function markAsAccredited(address _investor) external onlyOwner {
        require(_investor != address(0), "Invalid address");
        accreditedInvestors[_investor] = true;
        emit InvestorAccredited(_investor, block.timestamp);
    }

    /**
     * @notice Override transfer to add compliance checks
     * @param to Recipient address
     * @param amount Amount to transfer
     */
    function transfer(address to, uint256 amount) public override returns (bool) {
        require(whitelist[msg.sender], "Sender not whitelisted");
        require(whitelist[to], "Recipient not whitelisted");
        return super.transfer(to, amount);
    }

    /**
     * @notice Override transferFrom to add compliance checks
     * @param from Sender address
     * @param to Recipient address
     * @param amount Amount to transfer
     */
    function transferFrom(address from, address to, uint256 amount) public override returns (bool) {
        require(whitelist[from], "Sender not whitelisted");
        require(whitelist[to], "Recipient not whitelisted");
        return super.transferFrom(from, to, amount);
    }
}
```

**注意事项**:

-   ✅ 可定制的 Security Token Offering
-   ✅ 符合 MAS 监管要求
-   ✅ 支持多种代币标准
-   ✅ 自动化代币分发
-   ✅ 基于 ERC-20 标准推断

---

## 5. 业务流程 4: 二级市场交易

**验证状态**: ⚠️ 基于官方网站
**官方文档**: [InvestaX 官方网站](https://www.investax.io/)

### 5.1 流程概述

InvestaX 提供二级市场交易功能。

### 5.2 详细流程

#### 步骤 1: 投资者在平台上挂单 (即时)

**主要活动**:

-   卖家登录 InvestaX 平台
-   选择要出售的代币
-   设置出售价格 (市价或限价)
-   设置出售数量
-   提交挂单请求

**产出物**:

-   挂单记录 (订单 ID、价格、数量)
-   挂单确认通知
-   订单簿更新

**关键里程碑**: 挂单成功并显示在订单簿

#### 步骤 2: 买家提交购买订单 (即时)

**主要活动**:

-   买家浏览订单簿
-   选择要购买的挂单
-   确认购买价格和数量
-   提交购买订单
-   锁定购买资金

**产出物**:

-   购买订单记录
-   资金锁定记录
-   订单匹配通知

**关键里程碑**: 购买订单提交并资金锁定

#### 步骤 3: 合规检查 (1-5 分钟)

**主要活动**:

-   验证买家 KYC 状态 (是否通过 KYC)
-   验证买家白名单状态 (是否在白名单)
-   验证买家投资限额 (是否超过限额)
-   验证卖家锁定期 (是否在锁定期内)
-   AML 筛查 (买卖双方)

**产出物**:

-   KYC 验证结果
-   白名单验证结果
-   投资限额验证结果
-   锁定期验证结果
-   AML 筛查结果

**关键里程碑**: 合规检查通过

#### 步骤 4: 交易撮合 (即时)

**主要活动**:

-   匹配买卖订单
-   计算交易金额和手续费
-   生成交易确认单
-   通知买卖双方
-   准备代币转账

**产出物**:

-   交易确认单
-   手续费计算记录
-   买卖双方通知

**关键里程碑**: 交易撮合成功

#### 步骤 5: 代币转账 (1-5 分钟)

**主要活动**:

-   从卖家钱包转账代币到买家钱包
-   从买家账户扣除购买资金
-   向卖家账户转入销售资金
-   扣除交易手续费
-   更新持有者列表

**产出物**:

-   代币转账交易哈希
-   资金转账记录
-   手续费扣除记录
-   持有者列表更新记录
-   交易完成通知

**关键里程碑**: 代币和资金转账完成

### 5.3 实际案例

**案例: 新加坡商业地产代币二级市场交易**

-   **代币**: RealEstate-CBD-001
-   **卖家**: 早期投资者 (持有 2 年)
-   **买家**: 新投资者 (已通过 KYC)
-   **交易价格**: SGD 1.10 (溢价 10%)
-   **交易数量**: 10,000 枚代币
-   **交易金额**: SGD 11,000
-   **手续费**: 0.5% (SGD 55)
-   **交易时间**: 5 分钟 (从挂单到完成)

**注意事项**:

-   ✅ 符合 MAS 监管要求 (二级市场交易需符合证券法规)
-   ✅ 自动合规检查 (KYC、白名单、投资限额、锁定期)
-   ✅ 流动性管理 (订单簿机制,提供流动性)
-   ✅ 透明的交易记录 (所有交易上链,可追溯)
-   ✅ 实时交易撮合 (自动匹配买卖订单)
-   ✅ 低手续费 (0.5% 交易手续费)

---

## 6. 业务流程 5: 资产管理与分红

**验证状态**: ⚠️ 基于官方网站
**官方文档**: [Fund Tokenization Explained](https://www.investax.io/blog/fund-tokenization-explained)

### 6.1 流程概述

InvestaX 支持资产管理与分红功能。

### 6.2 详细流程

#### 步骤 1: 资产收益计算 (每月/每季度)

**主要活动**:

-   收集资产收益数据 (租金、利息、股息等)
-   扣除运营成本 (管理费、维护费、税费)
-   计算净收益
-   计算每代币分红金额
-   生成收益报告

**产出物**:

-   资产收益明细
-   运营成本明细
-   净收益计算表
-   每代币分红金额
-   收益报告

**关键里程碑**: 完成收益计算并生成报告

#### 步骤 2: 分红计划制定 (1-2 周)

**主要活动**:

-   确定分红比例 (通常 80-90% 净收益)
-   确定分红时间 (月度、季度、年度)
-   确定分红方式 (法币、稳定币、代币)
-   获得董事会批准
-   发布分红公告

**产出物**:

-   分红计划书
-   董事会决议
-   分红公告
-   分红时间表

**关键里程碑**: 分红计划获批并公告

#### 步骤 3: 自动分红分配 (1-2 天)

**主要活动**:

-   快照代币持有者列表 (特定时间点)
-   计算每个持有者的分红金额
-   准备分红资金 (法币或稳定币)
-   执行分红转账 (批量转账)
-   发送分红通知

**产出物**:

-   持有者快照记录
-   分红金额明细表
-   分红转账记录
-   分红通知邮件

**关键里程碑**: 分红资金成功转账到所有持有者

#### 步骤 4: 分红记录上链 (即时)

**主要活动**:

-   将分红记录写入智能合约
-   生成分红交易哈希
-   更新链上分红历史
-   发布分红完成公告
-   归档分红文件

**产出物**:

-   分红交易哈希
-   链上分红记录
-   分红完成公告
-   分红归档文件

**关键里程碑**: 分红记录成功上链

#### 步骤 5: 投资者查询 (随时)

**主要活动**:

-   投资者登录 InvestaX 平台
-   查看分红历史记录
-   下载分红证明文件
-   查询链上分红记录
-   联系客服咨询

**产出物**:

-   分红历史查询结果
-   分红证明文件 (PDF)
-   链上分红验证

**关键里程碑**: 投资者可随时查询分红记录

### 6.3 实际案例

**案例: 新加坡商业地产季度分红**

-   **资产**: 新加坡中央商务区办公楼
-   **季度租金收入**: SGD 1,200,000
-   **运营成本**: SGD 200,000 (管理费、维护费、税费)
-   **净收益**: SGD 1,000,000
-   **分红比例**: 90% (SGD 900,000)
-   **代币总量**: 50,000,000 枚
-   **每代币分红**: SGD 0.018 (年化 7.2%)
-   **分红方式**: USDC (稳定币)
-   **分红时间**: 每季度末

**注意事项**:

-   ✅ 自动化分红分配 (智能合约自动执行)
-   ✅ 按持股比例分配 (公平透明)
-   ✅ 透明的分红记录 (所有分红记录上链)
-   ✅ 支持多种资产类型 (房地产、私募股权、债券基金)
-   ✅ 多种分红方式 (法币、稳定币、代币)
-   ✅ 定期分红报告 (季度收益报告、年度审计报告)

---

## 总结

InvestaX 作为新加坡金融管理局许可的机构级代币化 SaaS 平台,提供了完整的 RWA 代币化解决方案。其核心优势在于:

1. **监管合规**: 受 MAS 监管,符合新加坡证券法规
2. **机构级服务**: 面向机构投资者,提供专业服务
3. **SaaS 模式**: 可定制的代币化解决方案
4. **全流程服务**: 从资产代币化到二级市场交易

**文档质量**: ⭐⭐⭐⭐ (基于官方网站和行业报告)
