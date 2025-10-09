# Centrifuge 核心合约分析

**文档版本**: v1.0  
**创建时间**: 2025-10-09 09:43:00 CST  
**框架**: Substrate + Solidity  
**语言**: Rust (Pallets) + Solidity (EVM)

---

## 📑 目录

1. [Pallet列表](#1-pallet列表)
2. [EVM合约列表](#2-evm合约列表)
3. [数据结构](#3-数据结构)
4. [主要接口](#4-主要接口)

---

## 1. Pallet列表

### 1.1 核心Pallets

| Pallet名称 | 职责 | 代码行数 |
|-----------|------|---------|
| **pallet-pool** | 资产池管理 | ~800行 |
| **pallet-nft** | 资产NFT | ~600行 |
| **pallet-tranche** | 分层投资 | ~700行 |
| **pallet-pricing** | 定价预言机 | ~500行 |
| **pallet-permissions** | 权限管理 | ~400行 |

---

## 2. EVM合约列表

### 2.1 Tinlake合约（Ethereum）

| 合约名称 | 职责 | 代码行数 |
|---------|------|---------|
| **Assessor.sol** | 池评估合约 | ~600行 |
| **Tranche.sol** | 分层合约 | ~500行 |
| **Reserve.sol** | 储备金合约 | ~400行 |
| **Coordinator.sol** | 协调器合约 | ~700行 |
| **MemberList.sol** | 成员列表 | ~300行 |

---

## 3. 数据结构

### 3.1 Pool结构

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

### 3.2 Asset NFT结构

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

### 3.3 Tranche结构

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

### 4.1 Pool Pallet接口

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

### 4.2 Tranche合约接口

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

- [Centrifuge GitHub](https://github.com/centrifuge)
- [Substrate文档](https://docs.substrate.io)
- [Tinlake文档](https://docs.centrifuge.io/getting-started/tinlake)

---

**文档维护**: RWA-HUSD技术团队  
**最后更新**: 2025-10-09 09:43:00 CST
