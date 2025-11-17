# RWA-HUSD 合约架构与 SAAS 层设计映射说明

> 目标：根据 SAAS 层业务流程，分析哪些功能应该上链、哪些只在 SAAS 层处理，并在现有合约架构基础上补充/扩展合约设计。

---

## 0. 总体思路概览

- **上链的是「权利 & 承诺」**：  
  资产确权结果、代币发行与持有、资金托管与分红、赎回权利、KYC 授权状态、机构资质承诺。

- **不上链的是「过程 & 隐私」**：  
  KYC 详细资料、律师/评估具体文书、线下调查记录、租赁合同全文、客户实名信息、具体租约细节。

围绕这个原则，在现有 RWA-HUSD 合约架构基础上，建议重点增加 **3～4 个合约/模块**：

1. `AssetRegistry`（资产登记合约） ✅ 上链  
2. `ComplianceRegistry`（合规 / KYC 状态合约） ✅ 上链  
3. `InstitutionRegistry`（机构身份 + 质押映射，可复用/扩展现有 StakingAuthorization） ✅ 上链  
4. （可选）扩展 `RentCustodyContract` 或新建 `CashflowManager` 管理租金分红逻辑 ✅ 上链  

---

## 1. 按 SAAS 模块划分：上链 / 不上链 对照

### 1.1 资产方端（资产提交 & 审核 & 上链）

**SAAS 功能：**

- 填写房产信息（地址、面积、估值、年化收益、月租金…）  
- 上传各类证照（房产证、土地证、照片、合同、抵押文件等）  
- 查看审核进度（待律师审核、待评估、待确权、待上链…）  
- 被驳回后补件 / 重新提交  

**建议上链的内容：**

- ✅ 资产唯一 ID：`assetId`（例如业务系统 ID 的 `keccak256`）  
- ✅ 资产与 `PropertyToken` 的绑定关系：`assetId => propertyToken`  
- ✅ 资产关键「结果型状态」：  
  - `APPROVED_FOR_TOKENIZATION`（律师 + 评估 + 确权通过，可上链）  
  - `ONCHAIN_ACTIVE`（已发行代币、可交易）  
  - `FROZEN`（被冻结）  
  - `REDEEMED` / `DEFAULTED`（赎回完成 / 违约）  
- ✅ 若干关键文件/流程的 **哈希**：  
  - `legalOpinionHash`（律师最终意见 PDF 的 hash/IPFS CID）  
  - `valuationReportHash`（评估报告 hash）  
  - `titleDeedHash`（核心确权文书 hash）  

这些建议由新合约 **`AssetRegistry`** 统一管理。

**不建议上链的内容：**

- ❌ 房产详细地址（国别/城市可以上链，详细地址建议只存 hash，避免隐私问题）  
- ❌ 全量证照内容、图片、扫描件  
- ❌ 律师审理过程的每一步状态、具体意见文本  
- ❌ 评估过程中的中间数据与模型参数  

这些信息只在 SAAS + 文件存储中保留（可用 IPFS + 加密 + 数据库），链上仅保留 hash 进行关联。

---

### 1.2 评估机构端（案件评估、报告上传）

**SAAS 功能：**

- 案件认领  
- 校验资产资料完整度  
- 填写估值参数与风险评级  
- 上传正式评估报告  

**建议上链的内容：**

- ✅ 最终评估结论的摘要：  
  - `lastValuation`（数值）  
  - `valuationCurrency`  
  - `valuationTime`  
  - `valuationProvider`（指向 on-chain 的机构地址）  
- ✅ `valuationReportHash` 作为不可篡改的凭证  
- ✅ 评估机构 on-chain 身份与资质（通过 `InstitutionRegistry` + `StakingAuthorization` 体现）  

**不建议上链的内容：**

- ❌ 评估方法细节、模型、调研细节  
- ❌ 报告全文内容（仅存 hash / CID）  
- ❌ 评估师真实姓名、联系方式等隐私信息  

---

### 1.3 律师事务所端（线上初审 + 线下确权 + 拍卖执行）

**SAAS 功能：**

- 在线初审：逐个文件打勾（合格 / 不合格）+ 驳回原因  
- 确权任务：预约线下调查、拍照、记录  
- 拍卖执行：上传拍卖合同与成交价格，更新状态  

**建议上链的内容：**

- ✅ 最终「合规结论」布尔状态 + 时间：  
  - `legalApproved = true/false`  
  - `legalApprovedAt`  
- ✅ 关键确权/拍卖文书的 hash：  
  - `titleDeedHash`  
  - `auctionContractHash`  
- ✅ 拍卖成交价（如果需要用来指导赎回 / 分红）  
- ✅ 负责律师事务所的链上身份（通过 `InstitutionRegistry`）  

**不建议上链的内容：**

- ❌ 每一次退回/补件的详细理由  
- ❌ 律师笔记、调查细节  
- ❌ 拍卖过程中所有中间出价记录（可只上链最终成交价）  

---

### 1.4 节点运营端（上链执行）

**SAAS 功能：**

- 查看已确权且待上链的资产列表  
- 执行上链：调用合约完成资产登记、代币发行  
- 在必要时执行冻结 / 解冻  

**建议上链的内容：**

- ✅ `AssetRegistry` 中 `status` 从 `APPROVED_FOR_TOKENIZATION` → `ONCHAIN_ACTIVE` 的状态切换  
- ✅ 调用 `PropertyTokenFactory.createPropertyToken(...)` 时：  
  - 新增参数 `assetId`  
  - 工厂合约内部校验 `AssetRegistry.status(assetId)`  
- ✅ 在 `InstitutionRegistry` 中登记节点地址/机构，并通过 `onlyNodeOperator` 之类的修饰器控制权限  
- ✅ 冻结/解冻操作 + 事件（`Frozen`, `Unfrozen`）  

---

### 1.5 资产运营 & 租金托管 / 分红

**SAAS 功能：**

- 资产运营状态看板（应收租金、待缴房产数、违约资产数等）  
- 租金缴纳记录（按房源/租户维度）  
- 租金托管列表 & 对账  

**合约建议上链的内容：**

- ✅ 租金/收益在链上托管和分配的部分：  
  - 按 `assetId` 维度管理的资金池余额  
  - 每次分红批次记录（`distributionId`、总金额、时间）  
  - 向 token holder 分发的实际金额（批量分发事件）  
- ✅ 若租金为链上支付（HUSD/USDC），则付款交易、付款地址天然上链  
- ✅ 资产违约 / 中断分红这类会影响投资者权利的“结果型状态”  

**更适合留在 SAAS 的内容：**

- ❌ 每个月的「账单细节」：租户姓名、租约开始结束日期、合同条款等  
- ❌ 线下/法币支付部分的明细（在链上只体现为一笔「充值/补足资金」即可）  
- ❌ 催收流程记录、逾期天数详细过程（链上只需体现 `DEFAULTED` 或「停止分红」的结果即可）  

---

### 1.6 用户 & 机构 & KYC/KYB

**目标：** 给交易、赎回、分红等所有关键操作一个统一的合规校验入口。

**建议新增 on-chain：`ComplianceRegistry` + `InstitutionRegistry`**

链上只存：

- ✅ 地址是否通过 KYC/KYB（标记为可投资/禁止）  
- ✅ 简单风险等级/区域（例如枚举：`0 Normal / 1 Medium / 2 High`）  
- ✅ 机构类型：`LAW_FIRM` / `APPRAISAL_FIRM` / `NODE_OPERATOR` / `PROPERTY_MANAGER` 等（通过 `InstitutionRegistry`）  

业务合约如 `TradeContract` / `RedemptionManager` / `RentCustodyContract` / `PropertyTokenFactory` 等在关键入口调用：

```solidity
require(complianceRegistry.isAllowedToInvest(msg.sender), "KYC_REQUIRED");
```

**不上链的内容：**

- ❌ 用户实名信息（姓名、证件号、地址、银行账户等）  
- ❌ 详细 KYC 文档与审查过程  
- ❌ AML 引擎的详细判定逻辑（在 SAAS + 合规服务中实现）  

---

## 2. 建议新增 / 扩展的合约设计

### 2.1 新合约：`AssetRegistry`

**目标：** 在链上管理「资产生命周期的关键结果」，并与 `PropertyToken` 绑定。

**核心数据结构示例：**

```solidity
struct AssetInfo {
    bytes32 assetId;            // 业务侧 assetId 的 keccak
    address propertyToken;      // 对应的 PropertyToken 合约
    uint8 status;               // 枚举: 0 Draft, 1 ApprovedForToken, 2 OnchainActive, 3 Frozen, 4 Redeemed, 5 Defaulted
    bytes32 legalOpinionHash;   // 律师意见文书 hash/IPFS CID
    bytes32 valuationReportHash;// 评估报告 hash
    bytes32 titleDeedHash;      // 核心确权文件 hash
    uint256 lastValuation;      // 最近一次估值数值
    uint64 valuationTime;       // 估值时间
    address valuationProvider;  // 评估机构地址
    address legalProvider;      // 律师事务所地址
}
```

**关键函数：**

- `registerAsset(bytes32 assetId, address legalProvider)`  
  - 只能由 SAAS 后端 / 节点角色调用（`onlySystemOrNode`）  
- `setLegalApproved(bytes32 assetId, bytes32 legalOpinionHash, bool approved)`  
- `setValuation(bytes32 assetId, bytes32 valuationReportHash, uint256 value, address provider)`  
- `setStatus(bytes32 assetId, Status newStatus)`  
- `bindPropertyToken(bytes32 assetId, address propertyToken)`  

**与现有合约的关系：**

- `PropertyTokenFactory.createPropertyToken(...)` 新增参数 `bytes32 assetId`：  
  - 内部逻辑：  
    - `AssetRegistry.status(assetId)` 必须为 `APPROVED_FOR_TOKENIZATION`  
    - 创建成功后调用 `AssetRegistry.bindPropertyToken(assetId, tokenAddress)`  
    - 将 `status` 改为 `ONCHAIN_ACTIVE`  
- `PropertyToken` 内部保存 `bytes32 public assetId;`，便于其他合约查询  
- `RedemptionManager`、`RentCustodyContract`、`Treasury` 可以基于 `assetId` 做资产粒度管理，而不只依赖 token 地址。

---

### 2.2 新合约：`ComplianceRegistry`（KYC/合规状态）

**目标：** 为交易、赎回、分红等需要合规的操作提供统一 KYC 校验点。

**数据结构示例：**

```solidity
struct ComplianceInfo {
    bool kycPassed;       // 是否通过KYC/KYB
    uint8 riskLevel;      // 0 Normal, 1 Medium, 2 High
    uint64 lastUpdated;   // 更新时间
}
```

**核心函数：**

- `setKycStatus(address user, bool passed, uint8 riskLevel)`  
  - 由后台多签 / 合规专用 Role 调用  
- `isAllowedToInvest(address user) returns (bool)`  
- `isAllowedToRedeem(address user) returns (bool)`  

**对现有合约的影响：**

- `TradeContract`：  
  在 `createSellOrder / createBuyOrder / buyOrder / sellOrder` 等入口增加校验：

  ```solidity
  require(complianceRegistry.isAllowedToInvest(msg.sender), "KYC_REQUIRED");
  ```

- `RedemptionManager`：  
  在 `redeem(...)` 中增加：

  ```solidity
  require(complianceRegistry.isAllowedToRedeem(msg.sender), "KYC_REQUIRED");
  ```

- 其他如领取大额分红等操作也可按需接入该校验。

---

### 2.3 扩展 / 抽象：`StakingAuthorization` → `InstitutionRegistry`

现有架构中已经有 `AbleToken` + `StakingAuthorization` 做机构质押与授权。建议：

**新增机构维度信息：**

```solidity
enum InstitutionType { Unknown, LawFirm, AppraisalFirm, NodeOperator, PropertyManager }

struct Institution {
    InstitutionType institutionType;
    uint256 stakedAmount;      // ABLE 质押数量
    bool active;
    bytes32 metadataHash;      // 对应 SAAS 层机构表的 hash
}
```

**提供查询接口：**

- `isActiveInstitution(address)`  
- `isLawFirm(address)` / `isAppraisalFirm(address)` / `isNodeOperator(address)` 等便捷方法  

**对现有合约的影响：**

- 在 `AssetRegistry` 中设置 `legalProvider` / `valuationProvider` / `nodeOperator` 时：  
  - 要求对应地址在 `InstitutionRegistry` 中 `active == true`，且类型匹配。  
- `PropertyTokenFactory`：  
  - 创建资产类 token 时要求调用方为某类机构（如 NodeOperator）或平台管理员。  
- 与现有 `StakingAuthorization` 的兼容：  
  - 可通过 UUPS 升级扩展存储，不破坏现有接口；  
  - 或新增独立 `InstitutionRegistry` 合约，由老合约引用。  

---

### 2.4 租金 / 分红：扩展 `RentCustodyContract` 或新建 `CashflowManager`

结合 SAAS 中的「租金缴纳」「资产运营看板」「托管列表」，以及现有 `Treasury` + 租金托管合约的设计，建议：

**合约层保留 / 增加：**

- 按 `assetId` 维度管理余额，而不仅仅是 token：

  ```solidity
  mapping(bytes32 => HouseBalance) public houseBalances;
  ```

- 分发时增加事件：

  ```solidity
  event RentDistributed(bytes32 indexed assetId, uint256 totalAmount, uint256 timestamp);
  event RentClaimed(bytes32 indexed assetId, address indexed user, uint256 amount);
  ```

- 具体采用：  
  - 「主动领取模型」（pull）→ 为每个用户记录可领取金额；  
  - 或「批量分发模型」（push）→ 延续当前批量结构即可。  

**不上链但要在 SAAS 做的：**

- 租户层合同与账单（租户姓名、租期、违约条款等）  
- 线下/法币部分租金与手续费的精细账务（链上只需一个总额充值映射）  

**对现有合约的影响：**

- 尽量通过新增函数 + 新事件实现，不破坏已发布接口。  
- 如已有 `houseId` 等字段，可通过 `assetId = keccak256(houseId)` 映射，保持兼容。  

---

## 3. 从合约视角再看 SAAS 层

### 3.1 现有合约已覆盖的链上职责

- `PropertyTokenFactory`：房产代币创建  
- `PropertyToken`：持仓、分红、部分赎回逻辑  
- `Treasury`：资金池、策略管理、提款  
- `RentCustodyContract`：租金托管与分发  
- `RedemptionManager`：赎回流程资金管理  
- `TradeContract`：OTC/订单撮合交易  
- `AbleToken + StakingAuthorization`：机构质押与授权  

这些主要覆盖了 **「投资端 + 资金端」** 的链上逻辑。

### 3.2 结合 SAAS 层，现在缺的链上“桥梁”

1. **资产生命周期与 PropertyToken 之间的桥** → `AssetRegistry`  
2. **合规 / KYC/KYB 的统一校验点** → `ComplianceRegistry`  
3. **机构身份与 Staking 的显式绑定** → 扩展 `StakingAuthorization` 为 `InstitutionRegistry`  

这三块补上之后，SAAS 的「资产全流程 + 机构协同」就能在链上形成可验证的最小状态集。

---

## 4. 落地实施建议

1. **更新架构图**：  
   在原有合约架构图上增加 `AssetRegistry` / `ComplianceRegistry` / `InstitutionRegistry`，并标注这些合约与 SAAS 模块（资产端 / 评估 / 律师 / 节点 / 账户中心）的对应关系。

2. **从最小改动版本开始实现**：  
   - v1：优先上线 `AssetRegistry + ComplianceRegistry`，在 `Factory / Trade / RedemptionManager` 中通过 `require` 方式接入，不改原接口签名。  
   - v2：进一步拓展 `StakingAuthorization` → `InstitutionRegistry`，再精细化租金托管部分。  

3. **配合 SAAS API 设计**：  
   - 为每个核心按钮（提交资产、通过审核、确认上链、发起分红等）明确：  
     - 是否调用链上合约  
     - 调用哪个新函数  
     - 需要同步哪些 `assetId` / hash / 状态到链上  
   - 形成一份「SAAS API ↔ 合约调用点对照表」，便于前后端与合约开发统一。

---

> 本文档可以作为「RWA-HUSD 合约架构与 SAAS 层结合设计说明」，后续可在此基础上进一步细化为：  
> - `AssetRegistry` / `ComplianceRegistry` / `InstitutionRegistry` 的完整 Solidity 接口与实现草案；  
> - SAAS REST / GraphQL API 与合约交互的映射清单。
