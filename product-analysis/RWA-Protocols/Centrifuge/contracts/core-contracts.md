# Centrifuge 核心合约分析

**文档版本**: v2.0
**创建时间**: 2025-10-09 09:43:00 CST
**最后更新**: 2025-10-13 10:00:00 CST
**框架**: Substrate + Solidity
**语言**: Rust (Pallets) + Solidity (EVM)

---

## 📑 目录

1. [Hub 模块合约](#1-hub模块合约)
2. [Spoke 模块合约](#2-spoke模块合约)
3. [Vaults 模块合约](#3-vaults模块合约)
4. [Pallet 列表(历史)](#4-pallet列表历史)
5. [Tinlake 合约(历史)](#5-tinlake合约历史)
6. [数据结构](#6-数据结构)
7. [主要接口](#7-主要接口)

---

## 1. Hub 模块合约

Hub 模块是 Centrifuge 协议的中央管理层,由**6 个核心智能合约**组成。

### 1.1 Hub 合约

| 属性         | 值                                           |
| ------------ | -------------------------------------------- |
| **合约名称** | Hub                                          |
| **职责**     | 中央池管理合约                               |
| **语言**     | Solidity                                     |
| **代理模式** | UUPS                                         |
| **主要功能** | 池创建、份额类别设置、元数据更新、管理员管理 |

**核心方法**:

```solidity
// 设置BalanceSheet管理器
function updateBalanceSheetManager(
    bytes32 centrifugeId,
    uint256 poolId,
    bytes32 manager,
    bool addManager
) external;

// 设置请求管理器
function setRequestManager(
    uint256 poolId,
    uint256 scId,
    uint256 assetId,
    bytes32 manager
) external;
```

---

### 1.2 HubHelpers 合约

| 属性         | 值                 |
| ------------ | ------------------ |
| **合约名称** | HubHelpers         |
| **职责**     | Hub 合约的扩展     |
| **语言**     | Solidity           |
| **主要功能** | 提供额外的辅助功能 |

---

### 1.3 HubRegistry 合约

| 属性         | 值                           |
| ------------ | ---------------------------- |
| **合约名称** | HubRegistry                  |
| **职责**     | 全局注册表                   |
| **语言**     | Solidity                     |
| **主要功能** | 池/资产/货币注册、唯一性保证 |

**数据结构**:

```solidity
// PoolId对象映射
mapping(bytes32 => PoolId) public pools;

// 资产注册
mapping(bytes32 => Asset) public assets;

// 货币注册
mapping(bytes32 => Currency) public currencies;
```

---

### 1.4 Holdings 合约

| 属性         | 值                           |
| ------------ | ---------------------------- |
| **合约名称** | Holdings                     |
| **职责**     | 持仓账本                     |
| **语言**     | Solidity                     |
| **主要功能** | 余额跟踪、头寸管理、可追溯性 |

**数据结构**:

```solidity
struct Holding {
    uint256 balance;        // 余额
    uint256 accountingId;   // 会计ID
    address asset;          // 资产地址
    uint256 timestamp;      // 时间戳
}

mapping(uint256 => Holding) public holdings;  // poolId => Holding
```

---

### 1.5 Accounting 合约

| 属性         | 值                           |
| ------------ | ---------------------------- |
| **合约名称** | Accounting                   |
| **职责**     | 复式记账系统                 |
| **语言**     | Solidity                     |
| **主要功能** | 借贷管理、账本平衡、审计追踪 |

**核心原理**:

```
借方(Debit) = 贷方(Credit)
```

**核心方法**:

```solidity
// 记录交易
function recordTransaction(
    uint256 debitAccountId,   // 借方账户ID
    uint256 creditAccountId,  // 贷方账户ID
    uint256 amount,           // 金额
    string memory description // 交易描述
) external;
```

---

### 1.6 ShareClassManager 合约

| 属性         | 值                         |
| ------------ | -------------------------- |
| **合约名称** | ShareClassManager          |
| **职责**     | 份额类别管理               |
| **语言**     | Solidity                   |
| **主要功能** | Epoch 工作流、跨链状态管理 |

**核心方法**:

```solidity
// 提交投资请求
function submitRequest(
    uint256 poolId,
    uint256 scId,
    uint256 amount,
    RequestType requestType  // INVEST or REDEEM
) external returns (uint256 requestId);

// 关闭Epoch
function closeEpoch(uint256 poolId, uint256 scId) external;

// 执行Epoch
function executeEpoch(uint256 poolId, uint256 scId) external;
```

---

## 2. Spoke 模块合约

Spoke 模块是 Centrifuge 协议的资产管理和金库操作层,由**5 个核心组件**组成。

### 2.1 BalanceSheet 合约

| 属性         | 值                            |
| ------------ | ----------------------------- |
| **合约名称** | BalanceSheet                  |
| **职责**     | 余额跟踪器                    |
| **语言**     | Solidity                      |
| **主要功能** | 资产/份额余额跟踪、管理器授权 |

**核心方法**:

```solidity
// 存款
function deposit(
    uint256 poolId,
    uint256 scId,
    address tokenAddress,
    uint256 tokenType,
    uint256 amount
) external;

// 提款
function withdraw(
    uint256 poolId,
    uint256 scId,
    address tokenAddress,
    uint256 tokenType,
    address receiver,
    uint256 amount
) external;
```

---

### 2.2 ShareToken 合约

| 属性         | 值                     |
| ------------ | ---------------------- |
| **合约名称** | ShareToken             |
| **职责**     | 份额代币               |
| **语言**     | Solidity               |
| **标准**     | ERC20 + ERC1404        |
| **主要功能** | 份额代币发行、转账限制 |

**转账限制类型**:

-   **FreezeOnly**: 仅冻结功能
-   **RedemptionRestrictions**: 赎回限制
-   **FullRestrictions**: 完整限制

---

### 2.3 Escrow 合约

| 属性         | 值                        |
| ------------ | ------------------------- |
| **合约名称** | Escrow                    |
| **职责**     | 托管机制                  |
| **语言**     | Solidity                  |
| **类型**     | Global Escrow, PoolEscrow |

**Global Escrow**:

-   持有待处理请求
-   系统范围的安全结算

**PoolEscrow**:

-   持有池的资产负债表
-   通过 PoolEscrowFactory 部署

---

## 3. Vaults 模块合约

Vaults 模块实现了灵活、安全和符合标准的代币化资产管理。

### 3.1 AsyncVault 合约

| 属性         | 值                       |
| ------------ | ------------------------ |
| **合约名称** | AsyncVault               |
| **职责**     | 完全异步金库             |
| **语言**     | Solidity                 |
| **标准**     | ERC-7540                 |
| **适用场景** | RWA 等需要延迟履行的资产 |

**核心方法**:

```solidity
// 请求存款
function requestDeposit(
    uint256 assets,
    address receiver
) external returns (uint256 requestId);

// 请求赎回
function requestRedeem(
    uint256 shares,
    address receiver
) external returns (uint256 requestId);
```

---

### 3.2 SyncDepositVault 合约

| 属性         | 值                  |
| ------------ | ------------------- |
| **合约名称** | SyncDepositVault    |
| **职责**     | 混合金库            |
| **语言**     | Solidity            |
| **标准**     | ERC-4626 + ERC-7540 |
| **特点**     | 同步存款 + 异步赎回 |

---

### 3.3 Manager 合约

#### 3.3.1 AsyncRequestManager

| 属性         | 值                           |
| ------------ | ---------------------------- |
| **合约名称** | AsyncRequestManager          |
| **职责**     | 异步操作引擎                 |
| **语言**     | Solidity                     |
| **主要功能** | 队列管理、执行管理、状态管理 |

---

#### 3.3.2 SyncManager

| 属性         | 值                 |
| ------------ | ------------------ |
| **合约名称** | SyncManager        |
| **职责**     | 同步操作管理       |
| **语言**     | Solidity           |
| **标准**     | ERC-4626           |
| **主要功能** | 汇率计算、会计逻辑 |

---

## 4. Pallet 列表(历史)

### 4.1 核心 Pallets

| Pallet 名称            | 职责       | 代码行数 |
| ---------------------- | ---------- | -------- |
| **pallet-pool**        | 资产池管理 | ~800 行  |
| **pallet-nft**         | 资产 NFT   | ~600 行  |
| **pallet-tranche**     | 分层投资   | ~700 行  |
| **pallet-pricing**     | 定价预言机 | ~500 行  |
| **pallet-permissions** | 权限管理   | ~400 行  |

---

## 2. EVM 合约列表

### 2.1 Tinlake 合约（Ethereum）

| 合约名称            | 职责       | 代码行数 |
| ------------------- | ---------- | -------- |
| **Assessor.sol**    | 池评估合约 | ~600 行  |
| **Tranche.sol**     | 分层合约   | ~500 行  |
| **Reserve.sol**     | 储备金合约 | ~400 行  |
| **Coordinator.sol** | 协调器合约 | ~700 行  |
| **MemberList.sol**  | 成员列表   | ~300 行  |

---

## 3. 数据结构

### 3.1 Pool 结构

```rust
// Substrate Pallet结构
pub struct Pool<T: Config> {
    pub pool_id: PoolId,
    pub admin: T::AccountId,
    pub reserve: Balance,
    pub max_reserve: Balance,
    pub tranches: Vec<TrancheId>,
    pub assets: Vec<AssetId>,
    pub status: PoolStatus,
}

pub enum PoolStatus {
    Active,
    Paused,
    Closed,
}
```

### 3.2 Asset NFT 结构

```rust
pub struct Asset<T: Config> {
    pub asset_id: AssetId,
    pub pool_id: PoolId,
    pub value: Balance,
    pub maturity_date: Moment,
    pub interest_rate: Rate,
    pub status: AssetStatus,
    pub metadata: BoundedVec<u8, T::MaxMetadataLength>,
}

pub enum AssetStatus {
    Active,
    Repaid,
    Defaulted,
    WrittenOff,
}
```

### 3.3 Tranche 结构

```solidity
// EVM合约结构
struct Tranche {
    uint256 trancheId;
    string name;              // "Junior" or "Senior"
    uint256 totalSupply;      // 总供应量
    uint256 totalDebt;        // 总债务
    uint256 interestRate;     // 利率（基点）
    uint256 minRatio;         // 最小比例
    uint256 maxRatio;         // 最大比例
    TrancheStatus status;
}

enum TrancheStatus {
    Active,
    Paused,
    Closed
}
```

---

## 4. 主要接口

### 4.1 Pool Pallet 接口

```rust
#[pallet::call]
impl<T: Config> Pallet<T> {
    /// 创建资产池
    #[pallet::weight(10_000)]
    pub fn create_pool(
        origin: OriginFor<T>,
        pool_id: PoolId,
        tranches: Vec<TrancheInput>,
    ) -> DispatchResult {
        // 实现逻辑
    }

    /// 添加资产到池
    #[pallet::weight(10_000)]
    pub fn add_asset(
        origin: OriginFor<T>,
        pool_id: PoolId,
        asset: AssetInput,
    ) -> DispatchResult {
        // 实现逻辑
    }

    /// 更新池状态
    #[pallet::weight(5_000)]
    pub fn update_pool_status(
        origin: OriginFor<T>,
        pool_id: PoolId,
        status: PoolStatus,
    ) -> DispatchResult {
        // 实现逻辑
    }
}
```

### 4.2 Tranche 合约接口

```solidity
interface ITranche {
    // 投资
    function supply(uint256 amount) external returns (uint256 shares);

    // 赎回
    function redeem(uint256 shares) external returns (uint256 amount);

    // 计算份额价格
    function sharePrice() external view returns (uint256);

    // 获取用户余额
    function balanceOf(address user) external view returns (uint256);
}
```

### 4.3 事件定义

```rust
// Substrate事件
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
    PoolCreated(PoolId, T::AccountId),
    AssetAdded(PoolId, AssetId),
    TrancheInvested(PoolId, TrancheId, T::AccountId, Balance),
    AssetRepaid(PoolId, AssetId, Balance),
}
```

```solidity
// EVM事件
event Supply(address indexed user, uint256 amount, uint256 shares);
event Redeem(address indexed user, uint256 shares, uint256 amount);
event PoolUpdated(uint256 indexed poolId, uint256 nav);
```

---

## 📚 参考资源

-   [Centrifuge GitHub](https://github.com/centrifuge)
-   [Substrate 文档](https://docs.substrate.io)
-   [Tinlake 文档](https://docs.centrifuge.io/getting-started/tinlake)

---

**文档维护**: RWA-HUSD 技术团队  
**最后更新**: 2025-10-09 09:43:00 CST
