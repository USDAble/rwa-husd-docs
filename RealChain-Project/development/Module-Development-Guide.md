# RealChain 模块开发指南

**版本**: 1.0
**创建日期**: 2025 年 10 月 15 日
**最后更新**: 2025 年 10 月 15 日

> ⚠️ **重要声明**: 本文档为概念设计示例,用于学习和参考目的。
>
> -   **RealChain 不是一个真实存在的区块链项目**
> -   文档基于真实的技术栈(Cosmos SDK, Tendermint)进行概念设计
> -   所有代码示例和配置均为概念设计示例
> -   文档创建时间: 2025-10-15
> -   文档用途: 展示如何开发基于 Cosmos SDK 的公链及其扩展模块

---

## 📋 目录

1. [概述](#概述)
2. [自定义模块开发流程](#自定义模块开发流程)
3. [DApp 开发和集成流程](#dapp-开发和集成流程)
4. [附录](#附录)

---

## 📖 概述

### 1.1 文档说明

#### 文档目的

本文档旨在为 RealChain 模块开发者和 DApp 开发者提供完整的模块开发指南。文档涵盖:

-   **模块开发**: 自定义模块开发流程 (以 RWA 模块为例,包含资产管理、合规检查等业务逻辑)
-   **DApp 集成**: DApp 开发和 SDK 集成指南 (CosmJS、Keplr 钱包集成)

**重点**: 本文档专注于**业务模块开发**,包含 RWA 相关内容。如需了解公链开发基础 (链初始化、节点部署等),请参考 [公链开发指南](./Blockchain-Development-Guide.md)。

#### 适用对象

| 角色            | 适用章节 | 技能要求                    |
| --------------- | -------- | --------------------------- |
| **模块开发者**  | 第 2 章  | Go, Cosmos SDK, 业务逻辑    |
| **DApp 开发者** | 第 3 章  | JavaScript/TypeScript, Web3 |

#### 阅读指南

-   **初学者**: 建议先阅读 [公链开发指南](./Blockchain-Development-Guide.md) 了解基础,再学习模块开发
-   **有经验的开发者**: 可直接跳转到相关章节
-   **模块开发者**: 重点阅读第 2 章 (自定义模块开发)
-   **DApp 开发者**: 重点阅读第 3 章 (DApp 开发和集成)

#### 版本信息

-   **当前版本**: v1.0
-   **创建日期**: 2025 年 10 月 15 日
-   **最后更新**: 2025 年 10 月 15 日
-   **维护者**: RealChain 核心开发团队

---

### 1.2 RealChain 公链架构

#### 架构概览

RealChain 是一个基于 **Cosmos SDK** 构建的模块化通用公链,支持通过扩展模块实现 RWA (Real World Assets) 资产代币化等业务功能。

```
RealChain 架构
├── Cosmos SDK (核心框架)
│   ├── Tendermint Consensus (共识层)
│   ├── ABCI (应用层接口)
│   └── Core Modules (核心模块)
│       ├── x/auth (账户管理)
│       ├── x/bank (代币转账)
│       ├── x/staking (质押)
│       ├── x/governance (治理)
│       └── x/distribution (分配)
│
├── Custom Modules (自定义模块)
│   ├── x/rwa (RWA 资产管理) ⭐
│   ├── x/compliance (合规检查) ⭐
│   └── x/valuation (资产估值) ⭐
│
└── External Integrations (外部集成)
    ├── IBC (跨链协议)
    ├── IPFS (分布式存储)
    └── Oracles (预言机)
```

**说明**: ⭐ 标记的模块为 RealChain 特有的扩展模块

#### 核心技术栈

| 组件           | 技术       | 版本   | 用途               |
| -------------- | ---------- | ------ | ------------------ |
| **区块链框架** | Cosmos SDK | v0.50+ | 公链开发框架       |
| **共识机制**   | Tendermint | v0.38+ | BFT 共识           |
| **编程语言**   | Go         | 1.21+  | 智能合约和模块开发 |
| **跨链协议**   | IBC        | v8.0+  | 跨链通信           |
| **分布式存储** | IPFS       | -      | 资产元数据存储     |
| **开发工具**   | Ignite CLI | v28+   | 链开发脚手架       |

#### RWA 扩展模块定位

**重要**: RealChain 是一个**通用公链**,RWA 功能通过**扩展模块**实现,而非核心功能。

**设计理念**:

-   **模块化**: RWA 模块可插拔,不影响核心功能
-   **可扩展**: 可以添加其他业务模块 (DeFi, NFT, etc.)
-   **互操作**: 通过 IBC 与其他 Cosmos 链互操作

**RWA 模块功能**:

-   资产代币化 (Property, Commodity, Bond)
-   合规检查 (KYC/AML)
-   资产估值 (Valuation Oracle)
-   生命周期管理 (Issuance, Transfer, Redemption)

---

### 3.1 RWA 模块开发流程 (作为示例)

#### 流程概述

RWA (Real World Assets) 模块是 RealChain 的扩展模块示例,用于管理实物资产的代币化。本节展示如何开发一个完整的 RWA 模块。

**目标**:

-   实现资产代币化功能
-   集成合规检查
-   实现资产生命周期管理
-   参考 T-REX (ERC-3643) 标准

**预计时间**: 1-2 天

#### 模块设计

##### 资产类型定义

```go
// proto/realchain/rwa/asset.proto
syntax = "proto3";
package realchain.rwa;

option go_package = "realchain/x/rwa/types";

enum AssetType {
  ASSET_TYPE_UNSPECIFIED = 0;
  ASSET_TYPE_PROPERTY = 1;    // 房地产
  ASSET_TYPE_COMMODITY = 2;   // 商品
  ASSET_TYPE_BOND = 3;        // 债券
}

enum AssetStatus {
  ASSET_STATUS_PENDING = 0;    // 待审核
  ASSET_STATUS_ACTIVE = 1;     // 活跃
  ASSET_STATUS_SUSPENDED = 2;  // 暂停
  ASSET_STATUS_RETIRED = 3;    // 退市
}

message RWAAsset {
  string id = 1;
  AssetType asset_type = 2;
  string name = 3;
  string description = 4;
  string issuer = 5;
  string total_supply = 6;  // 使用 string 存储大数
  AssetStatus status = 7;
  string ipfs_hash = 8;     // 元数据存储在 IPFS
  int64 created_at = 9;
  int64 updated_at = 10;
}
```

##### 消息类型定义

```go
// proto/realchain/rwa/tx.proto
syntax = "proto3";
package realchain.rwa;

import "cosmos/msg/v1/msg.proto";
import "realchain/rwa/asset.proto";

option go_package = "realchain/x/rwa/types";

service Msg {
  option (cosmos.msg.v1.service) = true;

  rpc IssueAsset(MsgIssueAsset) returns (MsgIssueAssetResponse);
  rpc TransferAsset(MsgTransferAsset) returns (MsgTransferAssetResponse);
  rpc UpdateAssetStatus(MsgUpdateAssetStatus) returns (MsgUpdateAssetStatusResponse);
}

message MsgIssueAsset {
  option (cosmos.msg.v1.signer) = "issuer";

  string issuer = 1;
  AssetType asset_type = 2;
  string name = 3;
  string description = 4;
  string total_supply = 5;
  string ipfs_hash = 6;
}

message MsgIssueAssetResponse {
  string asset_id = 1;
}

message MsgTransferAsset {
  option (cosmos.msg.v1.signer) = "from";

  string from = 1;
  string to = 2;
  string asset_id = 3;
  string amount = 4;
}

message MsgTransferAssetResponse {}

message MsgUpdateAssetStatus {
  option (cosmos.msg.v1.signer) = "issuer";

  string issuer = 1;
  string asset_id = 2;
  AssetStatus new_status = 3;
}

message MsgUpdateAssetStatusResponse {}
```

#### 详细步骤

##### 步骤 1: 创建 RWA 模块骨架

```bash
# 创建 RWA 模块
ignite scaffold module rwa

# 创建资产发行消息
ignite scaffold message issue-asset \
  asset-type:uint \
  name \
  description \
  total-supply \
  ipfs-hash \
  --module rwa

# 创建资产转移消息
ignite scaffold message transfer-asset \
  to \
  asset-id \
  amount \
  --module rwa
```

##### 步骤 2: 实现 Keeper 逻辑

```go
// x/rwa/keeper/keeper.go
package keeper

import (
    "context"
    "fmt"

    "cosmossdk.io/store/prefix"
    storetypes "cosmossdk.io/store/types"
    "github.com/cosmos/cosmos-sdk/codec"
    sdk "github.com/cosmos/cosmos-sdk/types"

    "realchain/x/rwa/types"
)

type Keeper struct {
    cdc      codec.BinaryCodec
    storeKey storetypes.StoreKey

    // 依赖其他模块的 Keeper
    bankKeeper    types.BankKeeper
    accountKeeper types.AccountKeeper
}

func NewKeeper(
    cdc codec.BinaryCodec,
    storeKey storetypes.StoreKey,
    bankKeeper types.BankKeeper,
    accountKeeper types.AccountKeeper,
) Keeper {
    return Keeper{
        cdc:           cdc,
        storeKey:      storeKey,
        bankKeeper:    bankKeeper,
        accountKeeper: accountKeeper,
    }
}

// IssueAsset 发行资产
func (k Keeper) IssueAsset(ctx context.Context, msg *types.MsgIssueAsset) (string, error) {
    sdkCtx := sdk.UnwrapSDKContext(ctx)

    // 生成资产 ID
    assetID := fmt.Sprintf("rwa-%d", sdkCtx.BlockHeight())

    // 创建资产对象
    asset := types.RWAAsset{
        Id:          assetID,
        AssetType:   msg.AssetType,
        Name:        msg.Name,
        Description: msg.Description,
        Issuer:      msg.Issuer,
        TotalSupply: msg.TotalSupply,
        Status:      types.AssetStatus_ASSET_STATUS_ACTIVE,
        IpfsHash:    msg.IpfsHash,
        CreatedAt:   sdkCtx.BlockTime().Unix(),
        UpdatedAt:   sdkCtx.BlockTime().Unix(),
    }

    // 存储资产
    k.SetAsset(ctx, asset)

    // 铸造代币到发行者账户
    // (这里简化处理,实际应该使用 bank 模块)

    return assetID, nil
}

// SetAsset 存储资产
func (k Keeper) SetAsset(ctx context.Context, asset types.RWAAsset) {
    store := prefix.NewStore(runtime.KVStoreAdapter(k.storeService.OpenKVStore(ctx)), types.AssetKey)
    b := k.cdc.MustMarshal(&asset)
    store.Set([]byte(asset.Id), b)
}

// GetAsset 获取资产
func (k Keeper) GetAsset(ctx context.Context, id string) (types.RWAAsset, bool) {
    store := prefix.NewStore(runtime.KVStoreAdapter(k.storeService.OpenKVStore(ctx)), types.AssetKey)
    b := store.Get([]byte(id))
    if b == nil {
        return types.RWAAsset{}, false
    }

    var asset types.RWAAsset
    k.cdc.MustUnmarshal(b, &asset)
    return asset, true
}
```

##### 步骤 3: 实现消息处理器

```go
// x/rwa/keeper/msg_server.go
package keeper

import (
    "context"

    sdk "github.com/cosmos/cosmos-sdk/types"
    "realchain/x/rwa/types"
)

type msgServer struct {
    Keeper
}

func NewMsgServerImpl(keeper Keeper) types.MsgServer {
    return &msgServer{Keeper: keeper}
}

func (k msgServer) IssueAsset(goCtx context.Context, msg *types.MsgIssueAsset) (*types.MsgIssueAssetResponse, error) {
    ctx := sdk.UnwrapSDKContext(goCtx)

    // 验证发行者权限 (简化处理)
    // 实际应该检查发行者是否有权限发行资产

    // 发行资产
    assetID, err := k.Keeper.IssueAsset(goCtx, msg)
    if err != nil {
        return nil, err
    }

    // 发出事件
    ctx.EventManager().EmitEvent(
        sdk.NewEvent(
            "asset_issued",
            sdk.NewAttribute("asset_id", assetID),
            sdk.NewAttribute("issuer", msg.Issuer),
            sdk.NewAttribute("asset_type", msg.AssetType.String()),
        ),
    )

    return &types.MsgIssueAssetResponse{AssetId: assetID}, nil
}
```

#### Mermaid 流程图

##### 资产发行流程 (Sequence Diagram)

```mermaid
sequenceDiagram
    participant Issuer
    participant CLI
    participant MsgServer
    participant Keeper
    participant Store
    participant IPFS

    Issuer->>IPFS: 上传资产元数据
    IPFS-->>Issuer: 返回 IPFS Hash
    Issuer->>CLI: realchaind tx rwa issue-asset
    CLI->>MsgServer: MsgIssueAsset
    MsgServer->>MsgServer: 验证发行者权限
    MsgServer->>Keeper: IssueAsset()
    Keeper->>Keeper: 生成资产 ID
    Keeper->>Keeper: 创建资产对象
    Keeper->>Store: 存储资产
    Store-->>Keeper: 确认
    Keeper->>Keeper: 铸造代币
    Keeper-->>MsgServer: 资产 ID
    MsgServer->>MsgServer: 发出事件
    MsgServer-->>CLI: MsgIssueAssetResponse
    CLI-->>Issuer: 交易哈希 + 资产 ID
```

##### 资产状态转换 (State Diagram)

```mermaid
stateDiagram-v2
    [*] --> PENDING: 创建资产
    PENDING --> ACTIVE: 审核通过
    PENDING --> [*]: 审核拒绝
    ACTIVE --> SUSPENDED: 暂停交易
    SUSPENDED --> ACTIVE: 恢复交易
    ACTIVE --> RETIRED: 资产退市
    RETIRED --> [*]
```

#### 参考资源

-   **T-REX (ERC-3643)**: https://github.com/TokenySolutions/T-REX
-   **ERC-3643 标准**: https://eips.ethereum.org/EIPS/eip-3643
-   **Cosmos SDK 模块开发**: https://docs.cosmos.network/main/build/building-modules/intro

---

### 3.2 合规模块开发流程

#### 流程概述

合规模块用于实现 KYC/AML 检查,确保资产转移符合监管要求。参考 T-REX 的 ONCHAINID 标准。

**目标**:

-   实现链上身份验证
-   集成 KYC/AML 检查
-   实现合规规则引擎
-   参考 ONCHAINID 标准

**预计时间**: 1-2 天

#### 模块设计

##### 身份类型定义

```go
// proto/realchain/compliance/identity.proto
syntax = "proto3";
package realchain.compliance;

option go_package = "realchain/x/compliance/types";

enum IdentityStatus {
  IDENTITY_STATUS_UNVERIFIED = 0;
  IDENTITY_STATUS_PENDING = 1;
  IDENTITY_STATUS_VERIFIED = 2;
  IDENTITY_STATUS_REJECTED = 3;
}

message Identity {
  string address = 1;
  IdentityStatus status = 2;
  string kyc_provider = 3;
  repeated Claim claims = 4;
  int64 verified_at = 5;
  int64 expires_at = 6;
}

message Claim {
  string claim_type = 1;  // e.g., "KYC", "AML", "ACCREDITED_INVESTOR"
  string issuer = 2;
  string data = 3;
  int64 issued_at = 4;
}
```

##### 合规规则定义

```go
// proto/realchain/compliance/rule.proto
syntax = "proto3";
package realchain.compliance;

option go_package = "realchain/x/compliance/types";

message ComplianceRule {
  string asset_id = 1;
  repeated string required_claims = 2;  // 必需的 Claim 类型
  string min_holding_period = 3;        // 最小持有期 (秒)
  string max_transfer_amount = 4;       // 最大转移金额
  bool whitelist_enabled = 5;           // 是否启用白名单
  repeated string whitelist = 6;        // 白名单地址
}
```

#### 详细步骤

##### 步骤 1: 创建合规模块

```bash
# 创建合规模块
ignite scaffold module compliance

# 创建身份注册消息
ignite scaffold message register-identity \
  kyc-provider \
  --module compliance

# 创建 Claim 添加消息
ignite scaffold message add-claim \
  address \
  claim-type \
  data \
  --module compliance
```

##### 步骤 2: 实现 Keeper 逻辑

```go
// x/compliance/keeper/keeper.go
package keeper

import (
    "context"
    "fmt"

    "cosmossdk.io/store/prefix"
    storetypes "cosmossdk.io/store/types"
    "github.com/cosmos/cosmos-sdk/codec"
    sdk "github.com/cosmos/cosmos-sdk/types"

    "realchain/x/compliance/types"
)

type Keeper struct {
    cdc      codec.BinaryCodec
    storeKey storetypes.StoreKey
}

func NewKeeper(
    cdc codec.BinaryCodec,
    storeKey storetypes.StoreKey,
) Keeper {
    return Keeper{
        cdc:      cdc,
        storeKey: storeKey,
    }
}

// RegisterIdentity 注册身份
func (k Keeper) RegisterIdentity(ctx context.Context, address string, kycProvider string) error {
    identity := types.Identity{
        Address:     address,
        Status:      types.IdentityStatus_IDENTITY_STATUS_PENDING,
        KycProvider: kycProvider,
        Claims:      []types.Claim{},
    }

    k.SetIdentity(ctx, identity)
    return nil
}

// AddClaim 添加 Claim
func (k Keeper) AddClaim(ctx context.Context, address string, claim types.Claim) error {
    identity, found := k.GetIdentity(ctx, address)
    if !found {
        return fmt.Errorf("identity not found: %s", address)
    }

    identity.Claims = append(identity.Claims, claim)
    k.SetIdentity(ctx, identity)
    return nil
}

// CheckCompliance 检查合规性
func (k Keeper) CheckCompliance(ctx context.Context, from string, to string, assetID string, amount string) error {
    // 检查发送方身份
    fromIdentity, found := k.GetIdentity(ctx, from)
    if !found || fromIdentity.Status != types.IdentityStatus_IDENTITY_STATUS_VERIFIED {
        return fmt.Errorf("sender identity not verified")
    }

    // 检查接收方身份
    toIdentity, found := k.GetIdentity(ctx, to)
    if !found || toIdentity.Status != types.IdentityStatus_IDENTITY_STATUS_VERIFIED {
        return fmt.Errorf("receiver identity not verified")
    }

    // 检查合规规则
    rule, found := k.GetComplianceRule(ctx, assetID)
    if found {
        // 检查必需的 Claims
        for _, requiredClaim := range rule.RequiredClaims {
            if !k.hasClaim(fromIdentity, requiredClaim) {
                return fmt.Errorf("sender missing required claim: %s", requiredClaim)
            }
            if !k.hasClaim(toIdentity, requiredClaim) {
                return fmt.Errorf("receiver missing required claim: %s", requiredClaim)
            }
        }

        // 检查白名单
        if rule.WhitelistEnabled {
            if !k.isWhitelisted(rule, to) {
                return fmt.Errorf("receiver not in whitelist")
            }
        }
    }

    return nil
}

func (k Keeper) hasClaim(identity types.Identity, claimType string) bool {
    for _, claim := range identity.Claims {
        if claim.ClaimType == claimType {
            return true
        }
    }
    return false
}

func (k Keeper) isWhitelisted(rule types.ComplianceRule, address string) bool {
    for _, addr := range rule.Whitelist {
        if addr == address {
            return true
        }
    }
    return false
}

// SetIdentity 存储身份
func (k Keeper) SetIdentity(ctx context.Context, identity types.Identity) {
    store := prefix.NewStore(runtime.KVStoreAdapter(k.storeService.OpenKVStore(ctx)), types.IdentityKey)
    b := k.cdc.MustMarshal(&identity)
    store.Set([]byte(identity.Address), b)
}

// GetIdentity 获取身份
func (k Keeper) GetIdentity(ctx context.Context, address string) (types.Identity, bool) {
    store := prefix.NewStore(runtime.KVStoreAdapter(k.storeService.OpenKVStore(ctx)), types.IdentityKey)
    b := store.Get([]byte(address))
    if b == nil {
        return types.Identity{}, false
    }

    var identity types.Identity
    k.cdc.MustUnmarshal(b, &identity)
    return identity, true
}

// SetComplianceRule 存储合规规则
func (k Keeper) SetComplianceRule(ctx context.Context, rule types.ComplianceRule) {
    store := prefix.NewStore(runtime.KVStoreAdapter(k.storeService.OpenKVStore(ctx)), types.RuleKey)
    b := k.cdc.MustMarshal(&rule)
    store.Set([]byte(rule.AssetId), b)
}

// GetComplianceRule 获取合规规则
func (k Keeper) GetComplianceRule(ctx context.Context, assetID string) (types.ComplianceRule, bool) {
    store := prefix.NewStore(runtime.KVStoreAdapter(k.storeService.OpenKVStore(ctx)), types.RuleKey)
    b := store.Get([]byte(assetID))
    if b == nil {
        return types.ComplianceRule{}, false
    }

    var rule types.ComplianceRule
    k.cdc.MustUnmarshal(b, &rule)
    return rule, true
}
```

#### Mermaid 流程图

##### KYC 验证流程 (Sequence Diagram)

```mermaid
sequenceDiagram
    participant User
    participant KYCProvider
    participant CLI
    participant ComplianceModule
    participant IdentityStore

    User->>KYCProvider: 提交 KYC 资料
    KYCProvider->>KYCProvider: 验证身份
    KYCProvider-->>User: 验证通过
    User->>CLI: realchaind tx compliance register-identity
    CLI->>ComplianceModule: MsgRegisterIdentity
    ComplianceModule->>IdentityStore: 创建身份 (PENDING)
    IdentityStore-->>ComplianceModule: 确认
    KYCProvider->>CLI: realchaind tx compliance add-claim
    CLI->>ComplianceModule: MsgAddClaim
    ComplianceModule->>IdentityStore: 添加 KYC Claim
    ComplianceModule->>IdentityStore: 更新状态 (VERIFIED)
    IdentityStore-->>ComplianceModule: 确认
    ComplianceModule-->>CLI: 成功
    CLI-->>User: 身份验证完成
```

#### 参考资源

-   **ONCHAINID**: https://github.com/onchain-id/solidity
-   **T-REX Compliance**: https://github.com/TokenySolutions/T-REX/tree/main/contracts/compliance
-   **ERC-735**: https://github.com/ethereum/EIPs/issues/735

---

### 3.3 模块集成和测试

#### 流程概述

将 RWA 模块和合规模块集成到主链,并进行端到端测试。

**目标**:

-   集成多个模块
-   实现模块间通信
-   编写集成测试
-   进行端到端测试

**预计时间**: 4-8 小时

#### 详细步骤

##### 步骤 1: 集成模块到主链

编辑 `app/app.go`:

```go
import (
    rwakeeper "realchain/x/rwa/keeper"
    rwamodule "realchain/x/rwa"
    rwatypes "realchain/x/rwa/types"

    compliancekeeper "realchain/x/compliance/keeper"
    compliancemodule "realchain/x/compliance"
    compliancetypes "realchain/x/compliance/types"
)

type App struct {
    // ... 其他 Keeper
    RWAKeeper        rwakeeper.Keeper
    ComplianceKeeper compliancekeeper.Keeper
}

func NewApp(...) *App {
    // ... 初始化其他 Keeper

    // 初始化 Compliance Keeper
    app.ComplianceKeeper = compliancekeeper.NewKeeper(
        appCodec,
        keys[compliancetypes.StoreKey],
    )

    // 初始化 RWA Keeper (依赖 Compliance Keeper)
    app.RWAKeeper = rwakeeper.NewKeeper(
        appCodec,
        keys[rwatypes.StoreKey],
        app.BankKeeper,
        app.AccountKeeper,
        app.ComplianceKeeper,  // 注入 Compliance Keeper
    )

    // 注册模块
    app.ModuleManager = module.NewManager(
        // ... 其他模块
        compliancemodule.NewAppModule(appCodec, app.ComplianceKeeper),
        rwamodule.NewAppModule(appCodec, app.RWAKeeper),
    )

    return app
}
```

##### 步骤 2: 修改 RWA 模块集成合规检查

编辑 `x/rwa/keeper/msg_server.go`:

```go
func (k msgServer) TransferAsset(goCtx context.Context, msg *types.MsgTransferAsset) (*types.MsgTransferAssetResponse, error) {
    ctx := sdk.UnwrapSDKContext(goCtx)

    // 检查合规性
    err := k.complianceKeeper.CheckCompliance(goCtx, msg.From, msg.To, msg.AssetId, msg.Amount)
    if err != nil {
        return nil, fmt.Errorf("compliance check failed: %w", err)
    }

    // 执行转移
    // ...

    return &types.MsgTransferAssetResponse{}, nil
}
```

##### 步骤 3: 编写集成测试

创建 `x/rwa/keeper/integration_test.go`:

```go
package keeper_test

import (
    "testing"

    "github.com/stretchr/testify/require"

    keepertest "realchain/testutil/keeper"
    "realchain/x/rwa/keeper"
    "realchain/x/rwa/types"
    compliancetypes "realchain/x/compliance/types"
)

func TestAssetTransferWithCompliance(t *testing.T) {
    k, complianceK, ctx := keepertest.RWAKeeperWithCompliance(t)
    srv := keeper.NewMsgServerImpl(k)

    // 1. 注册身份
    alice := "real1alice..."
    bob := "real1bob..."

    err := complianceK.RegisterIdentity(ctx, alice, "kyc-provider-1")
    require.NoError(t, err)

    err = complianceK.RegisterIdentity(ctx, bob, "kyc-provider-1")
    require.NoError(t, err)

    // 2. 添加 KYC Claim
    aliceClaim := compliancetypes.Claim{
        ClaimType: "KYC",
        Issuer:    "kyc-provider-1",
        Data:      "verified",
    }
    err = complianceK.AddClaim(ctx, alice, aliceClaim)
    require.NoError(t, err)

    bobClaim := compliancetypes.Claim{
        ClaimType: "KYC",
        Issuer:    "kyc-provider-1",
        Data:      "verified",
    }
    err = complianceK.AddClaim(ctx, bob, bobClaim)
    require.NoError(t, err)

    // 3. 更新身份状态为已验证
    aliceIdentity, _ := complianceK.GetIdentity(ctx, alice)
    aliceIdentity.Status = compliancetypes.IdentityStatus_IDENTITY_STATUS_VERIFIED
    complianceK.SetIdentity(ctx, aliceIdentity)

    bobIdentity, _ := complianceK.GetIdentity(ctx, bob)
    bobIdentity.Status = compliancetypes.IdentityStatus_IDENTITY_STATUS_VERIFIED
    complianceK.SetIdentity(ctx, bobIdentity)

    // 4. 发行资产
    issueMsg := &types.MsgIssueAsset{
        Issuer:      alice,
        AssetType:   types.AssetType_ASSET_TYPE_PROPERTY,
        Name:        "Test Property",
        Description: "Test",
        TotalSupply: "1000000",
        IpfsHash:    "QmTest...",
    }
    issueRes, err := srv.IssueAsset(ctx, issueMsg)
    require.NoError(t, err)

    // 5. 设置合规规则
    rule := compliancetypes.ComplianceRule{
        AssetId:        issueRes.AssetId,
        RequiredClaims: []string{"KYC"},
    }
    complianceK.SetComplianceRule(ctx, rule)

    // 6. 转移资产 (应该成功)
    transferMsg := &types.MsgTransferAsset{
        From:    alice,
        To:      bob,
        AssetId: issueRes.AssetId,
        Amount:  "1000",
    }
    _, err = srv.TransferAsset(ctx, transferMsg)
    require.NoError(t, err)

    // 7. 尝试转移给未验证的用户 (应该失败)
    charlie := "real1charlie..."
    transferMsg2 := &types.MsgTransferAsset{
        From:    alice,
        To:      charlie,
        AssetId: issueRes.AssetId,
        Amount:  "1000",
    }
    _, err = srv.TransferAsset(ctx, transferMsg2)
    require.Error(t, err)
    require.Contains(t, err.Error(), "identity not verified")
}
```

运行测试:

```bash
go test ./x/rwa/keeper/... -v
```

#### 参考资源

-   **Cosmos SDK 模块集成**: https://docs.cosmos.network/main/build/building-apps/app-go
-   **集成测试**: https://docs.cosmos.network/main/build/building-modules/testing

---

## 📱 DApp 开发和集成流程

### 5.1 SDK 集成流程

#### 流程概述

使用 CosmJS 集成 RealChain SDK,实现与链的交互。

**目标**:

-   安装和配置 CosmJS
-   连接到 RealChain 节点
-   实现钱包集成
-   发送交易和查询数据

**预计时间**: 2-4 小时

#### 详细步骤

##### 步骤 1: 安装 CosmJS

```bash
# 创建新项目
mkdir realchain-dapp
cd realchain-dapp
npm init -y

# 安装 CosmJS
npm install @cosmjs/stargate @cosmjs/proto-signing @cosmjs/encoding

# 安装 TypeScript (可选)
npm install --save-dev typescript @types/node
```

##### 步骤 2: 配置 RealChain 客户端

创建 `src/client.ts`:

```typescript
import { SigningStargateClient, StargateClient } from "@cosmjs/stargate";
import { DirectSecp256k1HdWallet } from "@cosmjs/proto-signing";

// RealChain 配置
const RPC_ENDPOINT = "https://rpc.realchain.network:443";
const CHAIN_ID = "realchain-1";
const PREFIX = "real";

// 创建只读客户端
export async function createReadOnlyClient(): Promise<StargateClient> {
    const client = await StargateClient.connect(RPC_ENDPOINT);
    return client;
}

// 创建签名客户端
export async function createSigningClient(mnemonic: string): Promise<SigningStargateClient> {
    // 从助记词创建钱包
    const wallet = await DirectSecp256k1HdWallet.fromMnemonic(mnemonic, {
        prefix: PREFIX,
    });

    // 获取账户
    const [account] = await wallet.getAccounts();
    console.log("Account address:", account.address);

    // 创建签名客户端
    const client = await SigningStargateClient.connectWithSigner(RPC_ENDPOINT, wallet, {
        gasPrice: {
            amount: "0.001",
            denom: "able",
        },
    });

    return client;
}
```

##### 步骤 3: 实现基本操作

创建 `src/operations.ts`:

```typescript
import { SigningStargateClient, StargateClient } from "@cosmjs/stargate";
import { coins } from "@cosmjs/proto-signing";

// 查询余额
export async function getBalance(
    client: StargateClient,
    address: string,
    denom: string = "able"
): Promise<string> {
    const balance = await client.getBalance(address, denom);
    return balance.amount;
}

// 发送代币
export async function sendTokens(
    client: SigningStargateClient,
    senderAddress: string,
    recipientAddress: string,
    amount: string,
    denom: string = "able"
): Promise<string> {
    const result = await client.sendTokens(
        senderAddress,
        recipientAddress,
        coins(amount, denom),
        "auto"
    );

    return result.transactionHash;
}

// 查询交易
export async function getTransaction(client: StargateClient, txHash: string): Promise<any> {
    const tx = await client.getTx(txHash);
    return tx;
}

// 查询区块高度
export async function getBlockHeight(client: StargateClient): Promise<number> {
    const height = await client.getHeight();
    return height;
}
```

##### 步骤 4: 实现 RWA 模块交互

创建 `src/rwa.ts`:

```typescript
import { SigningStargateClient } from "@cosmjs/stargate";
import { MsgIssueAsset, MsgTransferAsset } from "./proto/realchain/rwa/tx";

// 发行资产
export async function issueAsset(
    client: SigningStargateClient,
    issuerAddress: string,
    assetType: number,
    name: string,
    description: string,
    totalSupply: string,
    ipfsHash: string
): Promise<string> {
    const msg = {
        typeUrl: "/realchain.rwa.MsgIssueAsset",
        value: MsgIssueAsset.fromPartial({
            issuer: issuerAddress,
            assetType: assetType,
            name: name,
            description: description,
            totalSupply: totalSupply,
            ipfsHash: ipfsHash,
        }),
    };

    const result = await client.signAndBroadcast(issuerAddress, [msg], "auto");

    if (result.code !== 0) {
        throw new Error(`Transaction failed: ${result.rawLog}`);
    }

    return result.transactionHash;
}

// 转移资产
export async function transferAsset(
    client: SigningStargateClient,
    fromAddress: string,
    toAddress: string,
    assetId: string,
    amount: string
): Promise<string> {
    const msg = {
        typeUrl: "/realchain.rwa.MsgTransferAsset",
        value: MsgTransferAsset.fromPartial({
            from: fromAddress,
            to: toAddress,
            assetId: assetId,
            amount: amount,
        }),
    };

    const result = await client.signAndBroadcast(fromAddress, [msg], "auto");

    if (result.code !== 0) {
        throw new Error(`Transaction failed: ${result.rawLog}`);
    }

    return result.transactionHash;
}

// 查询资产
export async function queryAsset(client: StargateClient, assetId: string): Promise<any> {
    const queryClient = client.forceGetQueryClient();
    const response = await queryClient.queryAbci(`/realchain.rwa.Query/Asset`, {
        id: assetId,
    });

    return response;
}
```

##### 步骤 5: 实现钱包集成 (Keplr)

创建 `src/wallet.ts`:

```typescript
// 检查 Keplr 是否安装
export function isKeplrInstalled(): boolean {
    return typeof window !== "undefined" && !!window.keplr;
}

// 连接 Keplr 钱包
export async function connectKeplr(chainId: string): Promise<string> {
    if (!isKeplrInstalled()) {
        throw new Error("Keplr wallet is not installed");
    }

    // 请求连接
    await window.keplr.enable(chainId);

    // 获取账户
    const offlineSigner = window.keplr.getOfflineSigner(chainId);
    const accounts = await offlineSigner.getAccounts();

    return accounts[0].address;
}

// 添加 RealChain 到 Keplr
export async function addRealChainToKeplr(): Promise<void> {
    if (!isKeplrInstalled()) {
        throw new Error("Keplr wallet is not installed");
    }

    await window.keplr.experimentalSuggestChain({
        chainId: "realchain-1",
        chainName: "RealChain",
        rpc: "https://rpc.realchain.network:443",
        rest: "https://api.realchain.network:443",
        bip44: {
            coinType: 118,
        },
        bech32Config: {
            bech32PrefixAccAddr: "real",
            bech32PrefixAccPub: "realpub",
            bech32PrefixValAddr: "realvaloper",
            bech32PrefixValPub: "realvaloperpub",
            bech32PrefixConsAddr: "realvalcons",
            bech32PrefixConsPub: "realvalconspub",
        },
        currencies: [
            {
                coinDenom: "ABLE",
                coinMinimalDenom: "able",
                coinDecimals: 6,
            },
        ],
        feeCurrencies: [
            {
                coinDenom: "ABLE",
                coinMinimalDenom: "able",
                coinDecimals: 6,
            },
        ],
        stakeCurrency: {
            coinDenom: "ABLE",
            coinMinimalDenom: "able",
            coinDecimals: 6,
        },
    });
}
```

#### 参考资源

-   **CosmJS 文档**: https://cosmos.github.io/cosmjs/
-   **Keplr 集成**: https://docs.keplr.app/

---

### 5.2 DApp 开发流程

#### 流程概述

开发一个完整的 RWA DApp,包括前端界面和智能合约交互。

**目标**:

-   搭建前端框架
-   实现资产发行界面
-   实现资产交易界面
-   集成钱包和签名

**预计时间**: 1-2 天

#### 技术栈

| 组件           | 技术                     | 用途     |
| -------------- | ------------------------ | -------- |
| **前端框架**   | React + TypeScript       | UI 开发  |
| **状态管理**   | Redux / Zustand          | 状态管理 |
| **UI 库**      | Material-UI / Ant Design | UI 组件  |
| **区块链交互** | CosmJS                   | 链交互   |
| **钱包集成**   | Keplr                    | 钱包连接 |

#### 详细步骤

##### 步骤 1: 创建 React 项目

```bash
# 使用 Vite 创建项目
npm create vite@latest realchain-dapp -- --template react-ts
cd realchain-dapp
npm install

# 安装依赖
npm install @cosmjs/stargate @cosmjs/proto-signing
npm install @mui/material @emotion/react @emotion/styled
npm install zustand
```

##### 步骤 2: 创建钱包连接组件

创建 `src/components/WalletConnect.tsx`:

```typescript
import React, { useState } from "react";
import { Button, Box, Typography } from "@mui/material";
import { connectKeplr, addRealChainToKeplr } from "../utils/wallet";

export const WalletConnect: React.FC = () => {
    const [address, setAddress] = useState<string>("");
    const [connected, setConnected] = useState<boolean>(false);

    const handleConnect = async () => {
        try {
            // 添加 RealChain 到 Keplr
            await addRealChainToKeplr();

            // 连接钱包
            const addr = await connectKeplr("realchain-1");
            setAddress(addr);
            setConnected(true);
        } catch (error) {
            console.error("Failed to connect wallet:", error);
        }
    };

    return (
        <Box>
            {!connected ? (
                <Button
                    variant="contained"
                    onClick={handleConnect}
                >
                    Connect Wallet
                </Button>
            ) : (
                <Box>
                    <Typography>Connected: {address}</Typography>
                </Box>
            )}
        </Box>
    );
};
```

##### 步骤 3: 创建资产发行组件

创建 `src/components/IssueAsset.tsx`:

```typescript
import React, { useState } from "react";
import { TextField, Button, Box, Typography } from "@mui/material";
import { issueAsset } from "../utils/rwa";
import { useWallet } from "../hooks/useWallet";

export const IssueAsset: React.FC = () => {
    const { client, address } = useWallet();
    const [name, setName] = useState("");
    const [description, setDescription] = useState("");
    const [totalSupply, setTotalSupply] = useState("");
    const [ipfsHash, setIpfsHash] = useState("");
    const [txHash, setTxHash] = useState("");

    const handleIssue = async () => {
        if (!client || !address) {
            alert("Please connect wallet first");
            return;
        }

        try {
            const hash = await issueAsset(
                client,
                address,
                1, // ASSET_TYPE_PROPERTY
                name,
                description,
                totalSupply,
                ipfsHash
            );

            setTxHash(hash);
            alert(`Asset issued! TX: ${hash}`);
        } catch (error) {
            console.error("Failed to issue asset:", error);
            alert("Failed to issue asset");
        }
    };

    return (
        <Box>
            <Typography variant="h5">Issue Asset</Typography>
            <TextField
                label="Name"
                value={name}
                onChange={(e) => setName(e.target.value)}
                fullWidth
                margin="normal"
            />
            <TextField
                label="Description"
                value={description}
                onChange={(e) => setDescription(e.target.value)}
                fullWidth
                margin="normal"
            />
            <TextField
                label="Total Supply"
                value={totalSupply}
                onChange={(e) => setTotalSupply(e.target.value)}
                fullWidth
                margin="normal"
            />
            <TextField
                label="IPFS Hash"
                value={ipfsHash}
                onChange={(e) => setIpfsHash(e.target.value)}
                fullWidth
                margin="normal"
            />
            <Button
                variant="contained"
                onClick={handleIssue}
            >
                Issue Asset
            </Button>
            {txHash && <Typography>Transaction Hash: {txHash}</Typography>}
        </Box>
    );
};
```

#### Mermaid 流程图

##### DApp 交互流程 (Sequence Diagram)

```mermaid
sequenceDiagram
    participant User
    participant DApp
    participant Keplr
    participant RealChain

    User->>DApp: 访问 DApp
    User->>DApp: 点击 "Connect Wallet"
    DApp->>Keplr: 请求连接
    Keplr-->>User: 请求授权
    User->>Keplr: 授权
    Keplr-->>DApp: 返回地址
    DApp-->>User: 显示已连接

    User->>DApp: 填写资产信息
    User->>DApp: 点击 "Issue Asset"
    DApp->>Keplr: 请求签名
    Keplr-->>User: 请求确认
    User->>Keplr: 确认签名
    Keplr-->>DApp: 返回签名
    DApp->>RealChain: 广播交易
    RealChain-->>DApp: 返回交易哈希
    DApp-->>User: 显示成功
```

#### 参考资源

-   **React 文档**: https://react.dev/
-   **Material-UI**: https://mui.com/
-   **CosmJS 示例**: https://github.com/cosmos/cosmjs/tree/main/packages/stargate

---

## 📚 附录

### 7.1 API 参考

#### REST API

RealChain 提供标准的 Cosmos SDK REST API:

| 端点                                          | 方法 | 说明           |
| --------------------------------------------- | ---- | -------------- |
| `/cosmos/bank/v1beta1/balances/{address}`     | GET  | 查询账户余额   |
| `/cosmos/tx/v1beta1/txs`                      | POST | 广播交易       |
| `/cosmos/tx/v1beta1/txs/{hash}`               | GET  | 查询交易       |
| `/cosmos/staking/v1beta1/validators`          | GET  | 查询验证者列表 |
| `/realchain/rwa/v1/assets/{id}`               | GET  | 查询 RWA 资产  |
| `/realchain/compliance/v1/identity/{address}` | GET  | 查询身份信息   |

**示例**:

```bash
# 查询余额
curl https://api.realchain.network/cosmos/bank/v1beta1/balances/real1abc...

# 查询 RWA 资产
curl https://api.realchain.network/realchain/rwa/v1/assets/rwa-12345
```

#### gRPC API

RealChain 提供 gRPC API (端口 9090):

```bash
# 使用 grpcurl 查询
grpcurl -plaintext localhost:9090 list

# 查询余额
grpcurl -plaintext \
  -d '{"address":"real1abc..."}' \
  localhost:9090 \
  cosmos.bank.v1beta1.Query/Balance
```

#### 参考资源

-   **Cosmos SDK REST API**: https://docs.cosmos.network/main/build/modules
-   **gRPC 文档**: https://grpc.io/docs/

---

### 7.2 常见问题 (FAQ)

#### 公链开发相关

**Q: 如何修改区块时间?**

A: 编辑 `~/.realchain/config/config.toml`:

```toml
[consensus]
timeout_commit = "5s"  # 修改为你想要的区块时间
```

**Q: 如何增加最大 Gas 限制?**

A: 编辑 `~/.realchain/config/app.toml`:

```toml
[api]
max-gas-wanted = 10000000  # 修改最大 Gas
```

**Q: 如何启用 CORS?**

A: 编辑 `~/.realchain/config/config.toml`:

```toml
[rpc]
cors_allowed_origins = ["*"]
```

#### 模块开发相关

**Q: 如何在模块间共享数据?**

A: 通过 Keeper 依赖注入:

```go
type Keeper struct {
    bankKeeper    types.BankKeeper
    stakingKeeper types.StakingKeeper
}
```

**Q: 如何处理大数?**

A: 使用 `sdk.Int` 或 `sdk.Dec`:

```go
amount := sdk.NewInt(1000000)
price := sdk.NewDec(100)
```

#### 节点运维相关

**Q: 如何备份节点数据?**

A: 备份 `~/.realchain` 目录:

```bash
tar -czf realchain-backup.tar.gz ~/.realchain
```

**Q: 如何恢复节点?**

A: 解压备份文件:

```bash
tar -xzf realchain-backup.tar.gz -C ~/
```

**Q: 验证者被 Jail 怎么办?**

A: 运行 unjail 命令:

```bash
realchaind tx slashing unjail \
  --from validator \
  --chain-id realchain-1
```

#### DApp 开发相关

**Q: 如何处理交易失败?**

A: 检查 `result.code` 和 `result.rawLog`:

```typescript
if (result.code !== 0) {
    console.error("Transaction failed:", result.rawLog);
}
```

**Q: 如何监听链上事件?**

A: 使用 WebSocket 订阅:

```typescript
const ws = new WebSocket("wss://rpc.realchain.network/websocket");
ws.send(
    JSON.stringify({
        jsonrpc: "2.0",
        method: "subscribe",
        params: ["tm.event='NewBlock'"],
        id: 1,
    })
);
```

---

### 7.3 故障排查指南

#### 节点无法启动

**症状**: 运行 `realchaind start` 后立即退出

**可能原因**:

1. 端口被占用
2. 创世文件损坏
3. 数据库损坏

**解决方案**:

```bash
# 检查端口
lsof -i :26656
lsof -i :26657

# 验证创世文件
realchaind validate-genesis

# 重置数据库 (警告: 会删除所有数据)
realchaind tendermint unsafe-reset-all
```

#### 节点不同步

**症状**: `catching_up` 一直为 `true`

**可能原因**:

1. 网络连接问题
2. Peers 配置错误
3. 磁盘 I/O 过慢

**解决方案**:

```bash
# 检查 Peers
realchaind status | jq '.SyncInfo.peers'

# 添加更多 Peers
# 编辑 ~/.realchain/config/config.toml
persistent_peers = "peer1@ip1:26656,peer2@ip2:26656"

# 启用状态同步
[statesync]
enable = true
```

#### 交易失败

**症状**: 交易返回错误码

**常见错误**:

| 错误码 | 说明       | 解决方案       |
| ------ | ---------- | -------------- |
| `5`    | 余额不足   | 检查账户余额   |
| `11`   | Out of gas | 增加 Gas 限制  |
| `19`   | 交易已存在 | 等待交易确认   |
| `32`   | 未授权     | 检查签名者权限 |

**解决方案**:

```bash
# 查询余额
realchaind query bank balances <address>

# 增加 Gas
realchaind tx ... --gas 200000

# 查询交易状态
realchaind query tx <tx-hash>
```

#### 内存不足

**症状**: 节点 OOM (Out of Memory)

**解决方案**:

```bash
# 增加 swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 优化配置
# 编辑 ~/.realchain/config/config.toml
[mempool]
size = 5000  # 减小 mempool 大小
cache_size = 10000  # 减小缓存大小
```

---

### 7.4 参考资源汇总

#### 官方文档

| 资源             | 链接                             | 说明           |
| ---------------- | -------------------------------- | -------------- |
| **Cosmos SDK**   | https://docs.cosmos.network/     | 核心框架文档   |
| **Tendermint**   | https://docs.tendermint.com/     | 共识引擎文档   |
| **IBC Protocol** | https://ibc.cosmos.network/      | 跨链协议文档   |
| **CosmJS**       | https://cosmos.github.io/cosmjs/ | JavaScript SDK |
| **Ignite CLI**   | https://docs.ignite.com/         | 开发工具文档   |

#### RWA 相关标准

| 资源                 | 链接                                        | 说明              |
| -------------------- | ------------------------------------------- | ----------------- |
| **T-REX (ERC-3643)** | https://github.com/TokenySolutions/T-REX    | 证券代币标准      |
| **ONCHAINID**        | https://github.com/onchain-id/solidity      | 链上身份标准      |
| **ERC-735**          | https://github.com/ethereum/EIPs/issues/735 | Claim Holder 标准 |

#### 成功案例

| 项目           | 链接                    | 说明                  |
| -------------- | ----------------------- | --------------------- |
| **Centrifuge** | https://centrifuge.io/  | RWA 代币化平台        |
| **Osmosis**    | https://osmosis.zone/   | Cosmos 生态 DEX       |
| **Juno**       | https://junonetwork.io/ | CosmWasm 智能合约平台 |

#### 社区资源

| 资源               | 链接                             | 说明         |
| ------------------ | -------------------------------- | ------------ |
| **Cosmos Discord** | https://discord.gg/cosmosnetwork | 官方 Discord |
| **Cosmos Forum**   | https://forum.cosmos.network/    | 官方论坛     |
| **Cosmos GitHub**  | https://github.com/cosmos        | 官方 GitHub  |

#### 开发工具

| 工具                     | 链接                              | 说明        |
| ------------------------ | --------------------------------- | ----------- |
| **Keplr Wallet**         | https://www.keplr.app/            | Cosmos 钱包 |
| **Mintscan**             | https://www.mintscan.io/          | 区块浏览器  |
| **Cosmos SDK Tutorials** | https://tutorials.cosmos.network/ | 官方教程    |

---

## 📝 文档结束

**版本**: 1.0
**创建日期**: 2025 年 10 月 15 日
**最后更新**: 2025 年 10 月 15 日

> ⚠️ **再次提醒**: 本文档为概念设计示例,用于学习和参考目的。RealChain 不是一个真实存在的区块链项目。所有代码示例和配置均为概念设计示例。

**文档统计**:

-   总章节: 7 章
-   总小节: 14 节
-   代码示例: 50+ 个
-   Mermaid 图表: 6 个
-   参考资源: 30+ 个

**适用对象**:

-   公链开发者
-   模块开发者
-   节点运维人员
-   DApp 开发者
-   治理参与者

**下一步建议**:

1. 从第 1 章开始,按顺序学习
2. 动手实践每个代码示例
3. 参考官方文档深入学习
4. 加入社区交流讨论

**反馈和贡献**:

-   如有问题或建议,请提交 Issue
-   欢迎贡献代码和文档改进

---

**感谢使用 RealChain 业务开发流程技术文档!** 🚀
