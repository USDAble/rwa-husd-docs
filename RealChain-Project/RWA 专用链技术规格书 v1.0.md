# RWA 专用链技术规格书

**文档编号**: [008]
**文档版本**: v1.0
**创建时间**: 2025-10-16
**文档类型**: 技术规格书 (Technical Specification)
**目标读者**: 第三方区块链开发团队、审计团队、SRE 团队、合规顾问
**所属项目**: USDable Chain RWA 专用链
**参考文档**: [007] USDable Chain 公链开发需求文档

---

## 文档体系说明

本文档是 USDable Chain 文档体系的一部分:

-   **L1 产品愿景**: USDable Chain 白皮书 (待创建)
-   **L2 产品需求**: [007] USDable Chain 公链开发需求文档
-   **L3 技术规格**: 本文档 (RWA 专用链技术规格书)
-   **L4 验收手册**: [012] RWA 专用链验收手册 (待创建)

本文档基于 [007] USDable Chain 公链开发需求文档,提供详细的技术实现规格。

---

## MVP 范围说明

### MVP 交付范围 (P0 - 区块链核心功能)

本技术规格书将功能模块分为 **P0 (区块链核心)** 和 **P1 (应用层协议)** 两个优先级:

**P0 模块 (MVP 必需 - 12 个月内交付)**:

1. **网络与节点** - 节点类型、P2P 网络、数据可用性
2. **共识、执行与费用市场** - PoS、EVM、EIP-1559
3. **合规引擎** - Policy DSL、preTransfer 钩子、错误码
4. **RWA 资产标准** - IRWAToken 接口、生命周期状态机
5. **分红系统** - 调度、计税、分配
6. **预言机与风控** - 价格聚合、NAV 计算、异常检测
7. **跨链分红** - 桥架构、合规、限额
8. **治理与升级** - 提案类型、投票、Timelock
9. **数据、事件、索引** - 事件 Schema、API/SDK、限流
10. **安全、审计、SRE** - 合约安全、运维、隐私

**P1 模块 (应用层协议 - 可选/扩展)**:

1. **DEX/AMM** - 流动性池、TWAP、合规池
2. **借贷/抵押/清算** - 市场、风险参数、清算机制
3. **稳定币** - 超额抵押、铸赎、储备证明
4. **保险池** - 资金来源、赔付上限、优先级

### MVP 验收标准

**P0 模块** 必须在 12 个月内完成并通过验收,包括:

-   所有 P0 模块的功能实现
-   ≥2 家独立审计 + 复审
-   性能指标达标 (TPS≥5,000、出块 3-5s、最终性 ≤2 块)
-   完整的验收脚本和测试报告

**P1 模块** 可在 MVP 交付后逐步实现,或由生态合作伙伴基于 P0 基础设施开发。

---

## 文档目标

实现一条 **EVM 兼容、RWA 原生合规、可审计、可扩展的公链**,平台代币 **ABLE** 用作 Gas/质押/治理/激励。

---

**0. 背景**

USDable
旨在构建"全球真实世界资产（RWA）的金融级公链基础设施"。本项目面向多资产：房地产、艺术品、股票、债券、收藏品等，要求在同一链上以统一标准（XRWA）实现：确权 → 代币化 → 流通 → 分红 → 抵押 → 清算 → 治理
的完整闭环，并兼顾跨链与合规。

业务目标：在 12
个月内交付可运营主网，支持 ≥10 类资产发行，TVL≥1 亿美金，活跃钱包 ≥5 万。

技术目标：EVM 一等公民、PoS+BFT 共识、TPS≥5,000、出块
3--5s、最终性 ≤2 块、Gas\<\$0.01。

合规目标：ERC-3643 全量实现；3 级 KYC；5
大司法辖区（US/EU/UK/SG/HK）最小合规基线。

## 1. 术语与缩写

### 1.1 核心术语

| 术语     | 英文全称                | 中文说明                                              |
| -------- | ----------------------- | ----------------------------------------------------- |
| **RWA**  | Real-World Assets       | 真实世界资产,包括房地产、艺术品、股票、债券、收藏品等 |
| **SPV**  | Special Purpose Vehicle | 特殊目的实体,用于资产隔离和风险管理                   |
| **ABLE** | -                       | 平台代币,用作 Gas/治理/质押                           |
| **pUSD** | -                       | 生态稳定币 (示例)                                     |
| **NAV**  | Net Asset Value         | 资产净值,用于 RWA 资产估值                            |
| **TVL**  | Total Value Locked      | 总锁仓价值                                            |

### 1.2 区块链技术术语

| 术语    | 英文全称                        | 中文说明                                   |
| ------- | ------------------------------- | ------------------------------------------ |
| **EVM** | Ethereum Virtual Machine        | 以太坊虚拟机,本链 100% 兼容                |
| **PoS** | Proof of Stake                  | 权益证明共识机制                           |
| **BFT** | Byzantine Fault Tolerance       | 拜占庭容错,用于最终性保证                  |
| **DA**  | Data Availability               | 数据可用性 (EigenDA/Celestia/ETH calldata) |
| **PBS** | Proposer-Builder Separation     | 提议者-构建者分离                          |
| **IL**  | Inclusion List                  | 强制包含列表,用于抗审查                    |
| **MEV** | Miner/Maximal Extractable Value | 矿工/最大可提取价值                        |
| **FN**  | Full Node                       | 全节点,存储最近状态                        |
| **AN**  | Archive Node                    | 归档节点,存储完整历史                      |

### 1.3 合规与身份术语

| 术语         | 英文全称                           | 中文说明                        |
| ------------ | ---------------------------------- | ------------------------------- |
| **DID**      | Decentralized Identifier           | 去中心化身份标识符              |
| **VC**       | Verifiable Credential              | 可验证凭证,用于 KYC/AML         |
| **KYC**      | Know Your Customer                 | 了解你的客户,身份验证流程       |
| **AML**      | Anti-Money Laundering              | 反洗钱                          |
| **OFAC**     | Office of Foreign Assets Control   | 美国外国资产控制办公室,制裁名单 |
| **GDPR**     | General Data Protection Regulation | 欧盟通用数据保护条例            |
| **ERC-3643** | -                                  | 以太坊 RWA 代币标准             |

### 1.4 金融协议术语

| 术语      | 英文全称                     | 中文说明         |
| --------- | ---------------------------- | ---------------- |
| **DEX**   | Decentralized Exchange       | 去中心化交易所   |
| **AMM**   | Automated Market Maker       | 自动做市商       |
| **TWAP**  | Time-Weighted Average Price  | 时间加权平均价格 |
| **CLAMM** | Concentrated Liquidity AMM   | 集中流动性 AMM   |
| **LTV**   | Loan-to-Value                | 贷款价值比       |
| **CDP**   | Collateralized Debt Position | 抵押债务头寸     |

### 1.5 技术实现术语

| 术语     | 英文全称                             | 中文说明                      |
| -------- | ------------------------------------ | ----------------------------- |
| **DSL**  | Domain-Specific Language             | 领域特定语言,用于合规策略定义 |
| **UUPS** | Universal Upgradeable Proxy Standard | 通用可升级代理标准            |
| **HSM**  | Hardware Security Module             | 硬件安全模块,用于密钥管理     |
| **KMS**  | Key Management Service               | 密钥管理服务                  |
| **TSS**  | Threshold Signature Scheme           | 门限签名方案                  |
| **MPC**  | Multi-Party Computation              | 多方计算                      |
| **RTO**  | Recovery Time Objective              | 恢复时间目标                  |
| **RPO**  | Recovery Point Objective             | 恢复点目标                    |
| **SLO**  | Service Level Objective              | 服务级别目标                  |
| **SRE**  | Site Reliability Engineering         | 站点可靠性工程                |

链总体设计目标与边界

**EVM 100% 兼容**：Solidity/Vyper、Hardhat/Foundry/Remix 全兼容。

**高性能**：区块时间 1--2s；最终性 ≤10s；目标 TPS（批量）≥5,000。

**强合规**：链内核提供**转账前校验**（KYC/司法/限售/额度/制裁名单），所有受限资产**无法绕过**。

**模块化**：合规、预言机、桥、费用市场、治理、清算、分红为**可插拔模块**，热升级可回滚。

**可审计**：状态机与策略版本化；关键操作全事件化；历史可追溯（AN +
索引）。

**经济闭环**：ABLE 作为
Gas、质押奖励与治理权，支持费用折扣与销毁策略（链级）。

**业务 ⇄ 链边界**：

业务侧：KYC/披露/法务文档签发与存储、税表生成；

链侧：**仅存哈希/指纹与策略引用**，在转账/发行/分红等路径强制执行。

## 第一部分: 区块链核心功能 (P0 - MVP 必需)

---

### 2. 网络与节点 (Network & Nodes) **[P0]**

**2.1 节点类型与职责**

![](media/image1.png)

**点击图片可查看完整电子表格**

**部署交付**：docker-compose/Helm/Terraform
一键化；出厂仪表盘（Prometheus/Grafana）；告警规则（出块/落盘/DA
失败/积压/背压/丢块）。

**2.2 P2P 网络与抗审查**

Gossip + Req/Resp；DoS 限流；对等连接上限、同源速率。

**Inclusion List / 延迟收件箱**：用户可向 L1/DA 提交 tx，Sequencer
必须在窗口 N 内纳入。

**验收脚本**：提交延迟收件箱交易 → N 个区块内出现
ForcedInclusion(txHash) 事件。

**2.3 数据可用性（DA）**

**混合 DA**：高价值交易 →ETH calldata；常规 →EigenDA/Celestia。

**切换策略**：治理参数可改；失败回退（DA 写入失败 → 重试/降级 ETH）。

**指标**：DA 写入成功率 ≥99.95%，延迟 P95≤2s。

### 3. 共识、执行与费用市场 **[P0]**

**3.1 共识与最终性**

PoS 验证 + Sequencer 出块；最终性协议要求 ≤10s。

Fork 处理：最长链 + 状态根一致性；Reorg 上限 ≤2 块。

**3.2 EVM 执行**

**ChainID**：固定提供；**Gas
Token**：ABLE；**预编译**：ECDSA/secp256k1、Keccak、BN128。

**日志与事件**：ABI 兼容；Bloom 过滤。

**存储策略**：快照+修剪（FN），全量（AN）。

**3.3 Gas & Fee**

**EIP-1559 模式**：BaseFee + Priority Fee；

**治理参数**：块 Gas 上限、BaseFee 最大增幅、最低 Priority。

**折扣**：ABLE 质押等级 → Gas 折扣（链级模块）。

**销毁**：BaseFee 部分销毁；可治理。

**验收**：高峰期 BaseFee 动态升降；折扣组生效；销毁事件计量准确。

### 4. 合规引擎 (Compliance Engine) **[P0]**

**4.1 策略 DSL (链上引用)**

---

YAML\
 version: 1.1.0\
 kyc:\
 provider: ACME_KYC\
 status: VERIFIED\
 expiry_days: 365\
 jurisdiction:\
 allow: \[UK, DE, FR, JP\]\
 deny: \[US, IR\]\
 investor_class_allow: \[PROFESSIONAL, INSTITUTIONAL\]\
 sanctions: \[OFAC, UN\]\
 transfer_rules:\
 lockup_days: 360\
 per_investor_cap: 1_000_000_USD\
 allowlist: merkleRoot_0xabc\...\
 revocation_list: merkleRoot_0xdef\...

---

**4.2 执行点与接口**

**preTransfer Hook**（强制）：canTransfer(from, to, amount) returns
(bool ok, uint16 code)

**策略版本化**：setPolicy(token, policyId, version, hash)（治理限权）。

**撤销**：VC 失效/被撤销 → 自动冻结（错误码 KYC_EXPIRED/REVOKED）。

**错误码**（示例）：

1001 KYC_REQUIRED / 1002 KYC_EXPIRED / 1003 JURISDICTION_DENIED

1004 LOCKUP_ACTIVE / 1005 CAP_EXCEEDED / 1006 SANCTIONED

**事件**：PolicyUpdated(token, policyId, version, diffHash)。

**验收**：修改 VC 为过期 → 任意转账 revert(1002)；加入制裁名单 →
revert(1006)；法域不在 allow → revert(1003)；仍保留可读 reason(code)。

### 5. RWA 资产标准与生命周期 **[P0]**

**5.1 标准接口 (Solidity 片段)**

---

Solidity\
 interface IRWAToken is IERC20 {\
 function state() external view returns (uint8); // 0..6\
 function metadata() external view returns (bytes32 ipfsHash); //
disclosures\
 function policy() external view returns (bytes32 policyId);\
 \
 event StateChanged(uint8 fromState, uint8 toState);\
 event DisclosureRegistered(bytes32 uriHash);\
 }

---

**状态机**：Draft→Offering→Active→Suspended→Defaulted→Redeeming→Closed

**Offering**：仅白名单/额度内认购；

**Active**：允许合规转让；

**Suspended**：冻结新转让；

**Defaulted**：违约触发（NAV/现金流/审计失败）；

**Redeeming**：回购/结清；

**Closed**：归档。

**验收脚本**：逐状态调用对应函数 → 触发 StateChanged；非法跳转 revert.

**5.2 认购与配售**

超额处理：比例配售/先到先得；超募退款自动回款。

白名单与额度：Merkle 证明 + per-investor cap。

事件：Subscribed(investor, amount, txHash) / Refunded(investor,
amount)。

### 6. 收益分红与现金流 **[P0]**

**6.1 调度与计算**

**周期**：/日/周/月/季（Business Day Convention 可配置）。

**快照**：recordDate 基于块高；

**税务**：按法域/类别预扣；税表哈希上链。

**6.2 发放与复投**

资产方选择：稳定币（pUSD/USDC）/ 复投份额。

**事件**：

DistributionScheduled(token, period, recordDate)

DistributionExecuted(token, batchId, gross, net, tax, merkleRoot)

TaxReportPublished(token, period, taxURIHash)

**验收**：对比持仓 × 利率 = 税前；税后=税前 ×(1-税率)；抽样核对 merkle
证明；失败重试批次全部到账。

## 第二部分: 应用层协议 (P1 - 可选/扩展)

---

### 7. 金融协议 (链内置约束) **[P1]**

**7.1 DEX / AMM**

池型：x\*y=k 与 CLAMM（可选）；合规池对受限资产开启白名单。

TWAP 输出供借贷使用；费率档与激励可治理。

**验收**：受限资产向"非合规池"加流动性 → revert；TWAP 与链上价格一致。

**7.2 借贷/抵押/清算**

市场：RWA / pUSD / ABLE；

风险参数：抵押因子、清算阈值、折扣、借款上限（治理）。

清算：直清 / 拍卖（荷兰式）；清算人需通过合规校验。

预言机异常 → 熔断（暂停借款/清算）。

**验收**：价格跌破 → 清算触发；非合规清算人 →
revert；熔断时相关操作拒绝并事件记录。

**7.3 稳定币（pUSD 类）**

超额抵押（RWA+ABLE+主流资产）；铸赎费率治理；储备证明上链。

偏差回正工具：费率/回购/激励。

**验收**：铸赎路径与费用正确；储备披露哈希一致；偏差\>阈值触发回正措施。

**7.4 保险池**

资金来源：发行费抽成/质押抽成等；

赔付上限、优先级（Senior/Junior）治理。

**验收**：违约触发 → 赔付 ≤ 上限，顺位正确。

## 第三部分: RWA 原生功能 (P0 - MVP 必需)

---

### 8. 预言机与风险控制 **[P0]**

**8.1 价格与 NAV**

多源聚合（中位数/加权）；异动阈值与冷却期；手动兜底（多签）。

**事件**：OracleUpdated(symbol, price, sources, median, sigHash)。

**验收**：单源异常 → 聚合剔除；高波动 → 触发冷静期；熔断事件生效。

### 9. 跨链收益 (Bridge / Skylink-like) **[P0]**

**9.1 架构**

资产留在本链；收益或凭证跨链：**消息通道 + 轻客户端/多签中继**。

目标链铸造 ReceiptToken 或直接发放稳定币。

合规：目标链地址需白名单快照或 ZK-KYC 证明。

**9.2 风险与限额**

跨链限额日/周上限；大额多签阈值提升；异常熔断。

**验收**：目标链领取成功；不合规地址拒绝；中继异常 → 回滚与赔付上限生效。

### 10. 治理与升级 (On-chain Governance) **[P0]**

提案类型：参数变更/模块升级/名单更新/桥限额/预言机权重。

流程：提案 → 投票（ABLE 权重）→ 时锁 ≥48h→ 执行；紧急开关多签+时锁。

事件：ProposalCreated/Queued/Executed/Cancelled。

**验收**：Gas 上限变更提案全流程跑通；回滚剧本通过演练。

### 11. 数据、事件、索引与开发者支持 **[P0]**

**11.1 事件 Schema (核心)**

![](media/image2.png)

**点击图片可查看完整电子表格**

**11.2 API / SDK / 限流**

**JSON-RPC/WS/GraphQL**；批量接口、分页、过滤。

速率限制（API Key 级别）；签名校验与幂等键。

SDK：JS/TS/Python/Go（含发行/分红/借贷/跨链示例 dApp）。

**验收**：QPS≥2000；P95≤200ms；错误码齐全（含合规模块码）。

### 12. 安全、审计、SRE 与隐私 **[P0]**

**12.1 合约安全**

上线前 ≥2 家独立审计 +
复审；关键模块（合规/桥/借贷/清算）建议形式化验证。

权限最小化：多签/时锁/Owner 可撤销；Upgrade 安全清单。

自动化：CI 安全扫描、静/动态分析、模糊测试。

**12.2 SRE 运维**

指标：出块延迟、Fork 率、交易积压、DA
失败、预言机延迟、桥积压、节点健康。

告警：阈值 + 突变；P95/P99；多渠道（邮件/Slack/Telegram）。

灾备：多地域多活、每日快照、季度恢复演练；RTO/RPO 目标。

密钥：HSM/KMS/TSS(MPC)；轮换流程 SOP。

**12.3 隐私与合规**

DID/VC：只存哈希与状态；GDPR 兼容（链下删除/更正）；披露/税表 URI+Hash
存证。

可选 Travel Rule/VASP 接口（对接合规机构）。

## 第四部分: 非功能需求 (P0 - MVP 必需)

---

### 13. 非功能性指标 (NFR) **[P0]**

![](media/image3.png)

**点击图片可查看完整电子表格**

### 14. 交付清单与环境 **[P0]**

**14.1 交付物**

源码（含注释/架构图）与 CI/CD（含安全扫描）

部署脚本：docker-compose/Helm/Terraform（devnet/testnet/mainnet）

API/SDK 文档、错误码表、事件字典、策略 DSL 规范

浏览器与索引服务、公共 RPC、faucet

监控与告警仪表盘、SRE Runbook

审计报告（≥2）+ 修复报告 + 攻防演练报告

**14.2 环境分层**

**Devnet**：每日自动重置；开发验证。

**Testnet**（公开）：外部可接；文档门户。

**Mainnet**：灰度发布；参数审慎。

### 15. 验收矩阵 (场景 × 检查 × 方式)

---

每项需附**脚本/命令/ABI 调用**与**预期返回** (第三方须交付《验收手册》)。

本验收矩阵分为 **P0 (MVP 必需)** 和 **P1 (可选/扩展)** 两部分。

---

#### 15.1 P0 验收矩阵 (MVP 必需)

**A. 合规与发行 [P0]**

KYC 未通过地址认购 → revert(1001)

VC 过期地址转账 → revert(1002)

US 地址二级转让 → revert(1003)

锁定期内转让 → revert(1004)

超额度认购 → revert(1005)

策略升级 v1→v2：旧持仓不溯及，新交易按 v2；触发 PolicyUpdated(diffHash)

**B. 生命周期与披露 [P0]**

Draft→Offering→Active→Suspended→Defaulted→Redeeming→Closed
正向通过；非法跳转 revert

披露 URI 下载文件哈希与 DisclosureRegistered 一致

**C. 分红与税务 [P0]**

设月度分红 → 生成快照 → 税前/税后匹配公式；DistributionExecuted 包含
merkleRoot

重试机制：模拟部分失败 → 重试到账

#### 15.2 P1 验收矩阵 (可选/扩展)

**D. 借贷/清算 [P1]**

预言机正常：价格下跌触发清算成功

预言机异常：熔断事件触发，借款与清算暂停

非合规清算人 → revert

**E. 稳定币与保险 [P1]**

pUSD 铸赎流程与费率正确;储备披露哈希一致

违约 → 保险赔付 ≤ 上限,顺位正确

**F. DEX 合规池 [P1]**

受限资产尝试进普通池 → revert

TWAP 与链上价格源一致 (误差阈值<ε)

#### 15.3 P0 验收矩阵 (续)

**G. 跨链收益 [P0]**

目标链领取凭证成功;不合格地址领取失败

中继异常 → 回滚与限额熔断记录

**H. 网络与 DA [P0]**

延迟收件箱交易 N 块内强制纳入,ForcedInclusion 事件出现

DA 写入失败 → 自动重试/降级 ETH;记录事件

**I. 治理与升级 [P0]**

Gas 上限参数变更提案 → 投票 → 时锁 → 执行成功

升级失败回滚脚本可用;紧急开关多签+时锁验证

**J. 性能与可靠性 [P0]**

压测 TPS≥5,000;P95 延迟指标达标

节点故障注入:自动恢复、SLO 不退化或在阈值内

### 16. 经济与费用 (链级约束) **[P0]**

**Gas**: ABLE;EIP-1559 动态;折扣等级 (质押 ABLE)。

**销毁**: BaseFee 比例销毁;治理可调。

**验证者奖励**: 出块/验证/挑战奖励;Slash 规则 (双签/作恶/长离线)。

### 17. 迁移、升级与应急 **[P0]**

**升级策略**: UUPS/Transparent Proxy;版本与兼容矩阵;回滚剧本 (含数据恢复)。

**应急**: 熔断 (桥/借贷/DEX);只读模式;社区公告与审计跟进。

**演练**: 季度升级与回滚演练记录。

**补充说明（避免遗漏）**

**没有省略**的关键链功能：

节点分类/DA/抗审查/费用市场/EVM 兼容细节/合规策略
DSL/转账钩子/生命周期状态机/分红税务/DEX
合规池/借贷清算/稳定币与回正/保险池/预言机聚合与熔断/跨链收益与限额/治理与时锁/日志事件与错误码/指标与告警/灾备/性能指标/交付与验收脚本/升级回滚与应急。

**业务与链配合**：KYC/披露/税表在业务端完成与存储，**链只存哈希与规则引用**，在执行路径**强制校验**，确保法律文本与链上策略
1:1 映射与可审计。
