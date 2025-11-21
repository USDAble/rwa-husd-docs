# RWA-HUSD 合约架构与 SAAS 层设计映射说明

> 目标：根据 SAAS 层业务流程，分析哪些功能应该上链、哪些只在 SAAS 层处理，并在现有合约架构基础上补充/扩展合约设计。

---

## 0. 总体思路概览

- **上链的是「权利 & 承诺」**：  
  资产确权结果、代币发行与持有、资金托管与分红、赎回权利、KYC 授权状态、机构资质承诺。

- **不上链的是「过程 & 隐私」**：  
  KYC 详细资料、律师/评估具体文书、线下调查记录、租赁合同全文、客户实名信息、具体租约细节。

围绕这个原则，在现有 RWA-HUSD 合约架构基础上，建议重点增加 **3～4 个合约/模块**：

- **使用现有 `serverId` 直接作为 `assetId`**：`bytes32` 传入链上即可，无需新增资产登记合约或额外 ID 体系。  
- **核心合约是合规入口**：`ComplianceRegistry`（等同于 UserRegistry V2 + 模块化合规框架），统一承载 KYC/KYB 校验、风险分级与合规模块。  
- **资产登记与机构管理系统作为未来可选扩展**：`AssetRegistry` / `InstitutionRegistry` 仅在后续需要更强资产生命周期或机构质押管理时再接入，详见 Section 6。  
- **租金/分红扩展保持可选**：在现有 `RentCustodyContract` 基础上按需增强，保持与核心合规设计解耦。  

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

## 2. 现有合约功能总结与缺口分析

### 2.1 现有合约架构梳理
- ✅ `batch-token-transfer`：`RentCustodyContract`（租金托管批量分发）
- ✅ `redemption-contracts`：`RedemptionManager`（赎回资金池管理）
- ✅ `rwa-husd-contracts`：`SystemConfig` / `UserRegistry` / `PropertyTokenFactory` / `PropertyToken`
- ✅ `rwa-husd-trading`：`TradeContract`（OTC 订单撮合）
- ✅ `treasury-contracts`：`Treasury` / `RedemptionStrategy`

### 2.2 现有合约已覆盖的功能
- ✅ Token 发行与购买：`PropertyToken.purchaseTokens`，分账 33.33% 至国库
- ✅ 赎回流程：`RedemptionManager`（含状态 6/7 校验）
- ✅ 二级交易：`TradeContract`（含 KYC 校验）
- ✅ 租金分红：`RentCustodyContract` 按 `houseId` 批量分发
- ✅ 国库资金管理：`Treasury` + `RedemptionStrategy`
- ⚠️ 基础 KYC：`UserRegistry` 布尔校验

### 2.3 SAAS 六步流程的链上缺口
1. 资产提交 → ❌ 缺少 `assetId` 链上登记  
2. 律师初审 → ❌ 缺少 `legalOpinionHash` 和审核结果记录  
3. 评估审核 → ❌ 缺少 `valuationReportHash`、估值数据、评估机构身份  
4. 线下确权 → ❌ 缺少 `titleDeedHash` 和确权凭证  
5. 节点上链 → ❌ 缺少节点运营商角色验证和上链审批流程  
6. 投资认购 → ✅ 已有，但缺少与 `assetId` 的绑定

### 2.4 关键缺失的链上能力
1. **资产生命周期管理**：无 `assetId → PropertyToken` 映射和状态机  
2. **审核流程记录**：无律师/评估/确权的链上哈希与时间戳  
3. **机构身份体系**：无 `LawFirm` / `AppraisalFirm` / `NodeOperator` 的链上角色与质押验证  
4. **合规统一入口**：`UserRegistry` 仅布尔 KYC，缺少风险等级与统一 `isAllowedToInvest / Redeem` 接口

### 2.5 必须上链的操作清单（新增）
- **影响投资者权益的状态变更**：资产审批通过/冻结/违约；`PropertyToken` 发行启用/停用；赎回快照值调整
- **金融操作**：`PropertyToken` 销售收款分账；租金分红按 `assetId` 归集与分发；赎回资金调拨
- **不可篡改凭证**：`legalOpinionHash`、`valuationReportHash`、`titleDeedHash`；估值数值与评估机构身份；节点上链交易哈希
- **权限校验**：节点/机构角色验证；交易/赎回的统一合规检查

---

## 3. 统一合规入口详细设计（基于 ERC3643）

### 3.1 设计原则与分阶段路线图
- 使用现有 `serverId` 直接作为 `assetId`（`bytes32`）传入链上，无需新增资产登记合约或额外 ID 体系。  
- 法律 / 估值 / 确权流程均在 SAAS 层完成，仅将结果型状态或哈希透出；合约只做校验与授权。  
- 暂不引入节点运营商与质押流程，接口预留扩展。  
- 核心是单一入口的合规校验（类似 ERC3643 的 Compliance 层），先做“可选”，后续一键切换为“强制”。  
- 三阶段渐进式落地：  
  - Phase 1：增强型 `UserRegistry`（最小合规，兼容旧布尔 KYC）。  
  - Phase 2：模块化合规框架 `ComplianceRegistry` + 插件式模块。  
  - Phase 3：声明/凭证体系（可选），用于更精细的资格验证。

### 3.2 Phase 1: 增强型 UserRegistry
目标：在保持布尔 KYC 兼容的前提下，增加合规维度，并提供统一查询入口。

**数据结构与接口示例（Solidity）：**
```solidity
struct UserProfile {
    bool kycPassed;          // 兼容旧字段
    uint8 riskLevel;         // 0 低 / 1 中 / 2 高
    uint16 countryCode;      // ISO-3166 数字码，节省 gas
    uint8 investorType;      // 0 Retail / 1 Accredited / 2 Institutional
    uint64 expiry;           // KYC 失效时间戳
}

interface IUserRegistryV2 {
    function setUserProfile(
        address user,
        bool kycPassed,
        uint8 riskLevel,
        uint16 countryCode,
        uint8 investorType,
        uint64 expiry
    ) external;

    function isKycPassed(address user) external view returns (bool); // 保留旧接口

    function isEligibleForInvestment(address user, bytes32 assetId)
        external
        view
        returns (bool allowed, string memory reason);
}
```

**兼容与迁移：**
- 旧的布尔 KYC 存储保留；`isKycPassed` 继续返回旧布尔值。  
- 新增 `UserProfile` 映射，布尔写入时默认填入 `riskLevel=0, investorType=0, expiry=type(uint64).max`。  
- 对存量用户，可批量脚本调用 `setUserProfile(user, oldFlag, 0, 0, 0, max)` 迁移。  
- `assetId` 贯穿 `isEligibleForInvestment` 以便后续按资产差异化规则。

**集成点（Phase 1 先行可选）：**
- `TradeContract` 下单前：若启用合规开关则调用 `isEligibleForInvestment(msg.sender, assetId)`。  
- `PropertyToken` 购买/转账：可通过 modifier 包裹；开启开关时校验。  
- `RedemptionManager`：赎回前检查 `isEligibleForInvestment(msg.sender, assetId)` 或单独 `isEligibleForRedemption`（可别名同实现）。

**可选强制开关（全局或资产级）：**
- `bool public complianceOptional = true;` 默认可选。  
- 可扩展 `mapping(bytes32 => bool) assetEnforcement;` 支持 per-asset 强制。

**Gas 优化要点：**
- 字段打包到单个 `UserProfile`，使用 `uint8/uint16/uint64`。  
- 只在必要时写入 storage，读操作可标记 `view` 并使用 `memory` 返回原因字符串。

### 3.3 Phase 2: 模块化合规框架
目标：引入类似 ERC3643 的可插拔模块，但简化适配 RWA 需求。

**核心接口与数据结构：**
```solidity
interface IComplianceModule {
    function isTransferAllowed(address from, address to, bytes32 assetId, uint256 amount)
        external
        view
        returns (bool allowed, string memory reason);
}

struct ComplianceRule {
    bool enabled;
    address module;
}

contract ComplianceRegistry {
    // 全局或资产级开关
    bool public globalOptional = true;
    mapping(bytes32 => bool) public assetEnforcement;

    // 资产级规则链表（小数组存地址即可）
    mapping(bytes32 => ComplianceRule[]) public assetRules;

    // 模块模板注册
    mapping(bytes32 => address) public ruleTemplates; // keccak256("CountryRestriction") => module addr

    function check(address from, address to, bytes32 assetId, uint256 amount)
        external
        view
        returns (bool, string memory);
}
```

**基础规则模块（示例）：**
- `CountryRestriction`: 允许/禁止国家列表，对 `UserRegistry` 的 `countryCode` 进行查询。  
- `InvestorTypeRestriction`: 仅允许特定 `investorType`。  
- `MaxHoldersLimit`: 读取 `PropertyToken.totalHolders(assetId)`，限制持有人数量。  
- `TransferLimit`: 限制单笔或周期内转账额度（简单实现：日上限累加，映射 `user => day => amount`）。

**集成位置：**
- `PropertyToken` 内部 `_beforeTokenTransfer` 调用 `complianceRegistry.check(from, to, assetId, amount)`。  
- `TradeContract` 撮合成交前检查 `check(buyer, seller, assetId, amount)`。  
- `RedemptionManager` 在赎回与转账阶段调用 `check(msg.sender, treasury, assetId, redeemAmount)`。

**可选/强制策略：**
- 默认 `globalOptional=true`：失败返回 `_check` 结果若为 false 但 optional，则记录事件 `ComplianceBypassed(assetId, user, reason)` 继续流程。  
- 当 `assetEnforcement[assetId]=true` 或 `globalOptional=false` 时，`require(allowed, reason)` 强制阻断。

**资产差异化配置：**
- `setAssetRules(assetId, [modules])` 将资产绑定特定规则集。  
- 轻量资产（低风险）可仅用 `CountryRestriction`，高风险资产启用全部模块。

**Gas 优化：**
- 将规则集合用小数组存地址，不用动态可变大数组；限制最多 4 个模块避免循环过长。  
- `reason` 文本使用短字符串或错误码（`bytes4`）以减小返回 gas，前端映射文案。  
- `assetId` 直接使用现有 `serverId` hash，避免重复存储。

### 3.4 Phase 3: 声明系统（可选）
目标：在不引入 ERC734/735 全量开销的前提下，提供简化版可信声明，用于更细粒度合规。

**核心概念：**
- `ClaimTypes`：`KYC_VERIFIED`, `ACCREDITED_INVESTOR`, `COUNTRY_APPROVED`, `AML_CLEARED`（枚举/`bytes32`）。  
- `TrustedIssuers`：可信的 KYC/AML 服务商地址列表，由多签/管理员维护。  
- 声明存储按 `user => claimType => Claim`。

**接口示例：**
```solidity
struct Claim {
    address issuer;
    uint64 issuedAt;
    uint64 validUntil;
    bytes32 dataHash; // 可选，用于引用 off-chain 证明
}

interface IClaimRegistry {
    function setTrustedIssuer(address issuer, bool allowed) external;
    function issueClaim(address user, bytes32 claimType, uint64 validUntil, bytes32 dataHash) external;
    function hasValidClaim(address user, bytes32 claimType) external view returns (bool);
}
```

**在合规检查中的使用：**
- `ComplianceRegistry` 的模块可查询 `IClaimRegistry.hasValidClaim(user, CLAIM_TYPE)`；若无声明则 fallback 到 `UserRegistry` 数据。  
- 适用于需要“合规即插即用”的场景（例如美国合格投资人、特定国家批准）。

**Gas 处理：**
- `Claim` 结构紧凑（2*uint64 + address + bytes32），可压缩到两个 storage slot。  
- 仅受信发行人可写，减少滥用。

### 3.5 与现有合约的集成点
- `UserRegistry`：增加 V2 存储与接口，保留旧 ABI，不影响现有布尔校验调用。  
- `PropertyToken`：新增 `bytes32 assetId` 成员；在 `_beforeTokenTransfer` 中调用 `ComplianceRegistry.check`；保留老逻辑可通过开关跳过。  
- `TradeContract`：下单/成交入口添加合规钩子；对于未开启强制的资产，失败时仅记录事件。  
- `RedemptionManager`：赎回、退款、资金划转前调用合规入口；同样支持可选跳过。  
- **数据纽带**：所有调用统一使用 `assetId`（= `serverId` 的 keccak），无需新建 `AssetRegistry`。  
- **SAAS 配置**：由后端面板管理“资产开关 + 规则集 + 可信发行人”，链上函数提供 `onlyAdmin`/`onlyRole` 控制。

### 3.6 可选强制机制设计
- **全局级别**：`ComplianceRegistry.globalOptional`；关闭后所有资产强制执行。  
- **资产级别**：`assetEnforcement[assetId]`；可针对高风险资产单独开启强制。  
- **过渡模式**：  
  1) 默认可选，记录 `ComplianceBypassed` 事件以便观测影响。  
  2) 观察期后按资产逐步开启强制。  
  3) 最终可关闭全局 Optional，进入全面合规模式。  
- **回滚预案**：保留管理员紧急开关，在出现误封时可快速恢复为可选模式。

### 3.7 实施建议与权衡
- **渐进上线**：Phase 1 先升级 `UserRegistry`，其余合约仅新增可选钩子，不改变既有接口签名。  
- **升级方式**：若采用 UUPS/Proxy，新增存储布局时在末尾添加 `UserProfile` 映射和配置开关，避免存储冲突；若不可升级，可部署 V2 注册表并在业务合约中优先读取 V2，兼容旧布尔接口。  
- **成本与收益**：Phase 1 改动最小，立即提供基础画像；Phase 2 引入模块化使高风险资产可定制；Phase 3 仅在需要“可携带证明”时开启，避免无谓存储。  
- **性能**：重点放在读路径轻量化（小数组模块、短错误码、位宽压缩）；写路径由运营后端批量执行，可接受更高成本。  
- **安全**：所有管理入口保留 `onlyAdmin/onlyRole` 与事件日志；模块合约需经过审计或复用模板，避免自定义模块带来的安全风险。  

---

## 4. 从合约视角再看 SAAS 层

### 4.1 现有合约已覆盖的链上职责

- `PropertyTokenFactory`：房产代币创建  
- `PropertyToken`：持仓、分红、部分赎回逻辑  
- `Treasury`：资金池、策略管理、提款  
- `RentCustodyContract`：租金托管与分发  
- `RedemptionManager`：赎回流程资金管理  
- `TradeContract`：OTC/订单撮合交易  
- `AbleToken + StakingAuthorization`：机构质押与授权  

这些主要覆盖了 **「投资端 + 资金端」** 的链上逻辑。

### 4.2 结合 SAAS 层，现在的核心与扩展路径

- 当前核心：`ComplianceRegistry`（含 UserRegistry V2 + 模块化合规）作为唯一合规 / KYC/KYB 校验点，交易、发行、赎回统一走这一入口。  
- 资产生命周期与机构身份管理：作为未来扩展（见 Section 6），在 Phase 1-3 合规落稳后按需要引入 `AssetRegistry` / `InstitutionRegistry`，补足资产状态与机构质押的链上证明。  
- 租金/分红扩展：保持可选，优先保证合规入口落地与运行数据，再迭代现金流托管与分发逻辑。

---

## 5. 落地实施建议

基于 Section 3 的三阶段合规设计，建议按以下路径推进：

- **Phase 1：UserRegistry 升级为 V2（最小合规）**  
  - 保留布尔接口，新增画像字段与 `isEligibleForInvestment/Redemption`。  
  - 在 `TradeContract` / `PropertyToken` / `RedemptionManager` 入口增加可选合规开关。  
- **Phase 2：上线 `ComplianceRegistry` + 模块化框架**  
  - 实现最小规则模块（如 CountryRestriction、InvestorTypeRestriction），提供资产级配置与全局/资产强制开关。  
  - 在 `_beforeTokenTransfer`、撮合、赎回路径接入 `check` 并支持可选跳过事件。  
- **Phase 3：声明系统（可选）**  
  - 引入 `ClaimRegistry` 与可信发行人管理，模块查询声明提升合规精度。  
  - 仅在需要可携带证明或分区市场时开启，避免不必要存储。  
- **配套工作**  
  - 更新架构图与 SAAS API 对照表，标注各按钮对应的合约调用与开关。  
  - 先以可选模式观察 `ComplianceBypassed` 事件，再按资产逐步开启强制并保留紧急回滚开关。

---

## 6. 未来扩展：资产登记与机构管理系统

> 这些设计作为未来扩展选项，依赖 Phase 1-3 的合规体系稳定运行后再考虑接入，当前阶段暂不实施。

### 6.1 新合约：`AssetRegistry`

**目标：** 在需要更强链上资产生命周期管理时，引入与 `PropertyToken` 绑定的登记合约。

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

**与现有合约的关系（如后续接入）：**

- `PropertyTokenFactory.createPropertyToken(...)` 新增参数 `bytes32 assetId`：  
  - 内部逻辑：  
    - `AssetRegistry.status(assetId)` 必须为 `APPROVED_FOR_TOKENIZATION`  
    - 创建成功后调用 `AssetRegistry.bindPropertyToken(assetId, tokenAddress)`  
    - 将 `status` 改为 `ONCHAIN_ACTIVE`  
- `PropertyToken` 内部保存 `bytes32 public assetId;`，便于其他合约查询  
- `RedemptionManager`、`RentCustodyContract`、`Treasury` 可以基于 `assetId` 做资产粒度管理，而不只依赖 token 地址。

补充说明：`AssetRegistry` 将 `assetId` 与 `PropertyToken` 以及 ApprovedForTokenization → OnchainActive → Frozen → Redeemed/Defaulted 的状态机强绑定，上链记录每个状态的结果与时间戳，便于在合规体系之上进一步完善资产生命周期的链上可验证性。

---

### 6.2 扩展 / 抽象：`StakingAuthorization` → `InstitutionRegistry`

现有架构中已经有 `AbleToken` + `StakingAuthorization` 做机构质押与授权，未来若需要显式机构类型与元数据映射，可扩展为独立注册表。

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

**对现有合约的影响（如接入）：**

- 在 `AssetRegistry` 中设置 `legalProvider` / `valuationProvider` / `nodeOperator` 时：  
  - 要求对应地址在 `InstitutionRegistry` 中 `active == true`，且类型匹配。  
- `PropertyTokenFactory`：  
  - 创建资产类 token 时要求调用方为某类机构（如 NodeOperator）或平台管理员。  
- 与现有 `StakingAuthorization` 的兼容：  
  - 可通过 UUPS 升级扩展存储，不破坏现有接口；  
  - 或新增独立 `InstitutionRegistry` 合约，由老合约引用。  

---

### 6.3 租金 / 分红：扩展 `RentCustodyContract` 或新建 `CashflowManager`

结合 SAAS 中的「租金缴纳」「资产运营看板」「托管列表」，以及现有 `Treasury` + 租金托管合约的设计，未来可按需增强：

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
