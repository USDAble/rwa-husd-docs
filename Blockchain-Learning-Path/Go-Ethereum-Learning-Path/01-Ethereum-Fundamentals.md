# 01 - 以太坊基础 (Ethereum Fundamentals)

**难度**: ⭐⭐☆☆☆ | **预估学习时间**: 8-12 小时

---

## 📋 本章概述

本章将介绍以太坊协议的核心概念,包括账户模型、交易机制、Gas 费用系统和 PoS 共识机制。这些基础知识是理解 Geth 客户端和以太坊网络运作的前提。

### 学习目标

完成本章学习后,您将能够:

- 理解以太坊的账户模型和状态机制
- 掌握交易的结构和生命周期
- 理解 Gas 机制和费用市场
- 了解 PoS 共识机制的基本原理
- 理解以太坊网络的分层架构

---

## 1. 以太坊协议概述

### 1.1 什么是以太坊?

以太坊 (Ethereum) 是一个去中心化的区块链平台,支持智能合约的执行。与比特币不同,以太坊不仅是一个加密货币网络,更是一个全球性的分布式计算平台。

**核心特性**:
- **图灵完备**: 支持任意复杂的计算逻辑
- **智能合约**: 自动执行的代码合约
- **去中心化应用 (DApp)**: 运行在区块链上的应用程序
- **EVM (以太坊虚拟机)**: 执行智能合约的运行环境

### 1.2 以太坊的演进历史

| 阶段 | 时间 | 共识机制 | 主要特性 |
|------|------|----------|----------|
| **Frontier** | 2015.07 | PoW (Ethash) | 以太坊主网启动 |
| **Homestead** | 2016.03 | PoW | 首个稳定版本 |
| **Byzantium** | 2017.10 | PoW | 引入 zk-SNARKs |
| **Constantinople** | 2019.02 | PoW | 优化 Gas 费用 |
| **Istanbul** | 2019.12 | PoW | 提升安全性 |
| **Berlin** | 2021.04 | PoW | Gas 费用优化 |
| **London** | 2021.08 | PoW | EIP-1559 (费用市场改革) |
| **The Merge** | 2022.09 | **PoS** | 从 PoW 转向 PoS |
| **Shanghai** | 2023.04 | PoS | 支持质押提款 |

**参考资料**: [以太坊硬分叉历史](https://github.com/ethereum/execution-specs/tree/master/network-upgrades/mainnet-upgrades)

---

## 2. 账户模型与状态

### 2.1 账户类型

以太坊有两种账户类型:

#### 外部账户 (EOA - Externally Owned Account)

- **控制方式**: 由私钥控制
- **特性**:
  - 可以发起交易
  - 没有代码
  - 由用户直接控制
- **组成**:
  - 地址 (Address): 20 字节,由公钥派生
  - 余额 (Balance): 账户持有的 ETH 数量
  - 随机数 (Nonce): 交易计数器,防止重放攻击

#### 合约账户 (Contract Account)

- **控制方式**: 由智能合约代码控制
- **特性**:
  - 不能主动发起交易
  - 包含可执行代码
  - 由外部账户或其他合约触发
- **组成**:
  - 地址 (Address): 20 字节,由创建者地址和 nonce 派生
  - 余额 (Balance): 账户持有的 ETH 数量
  - 代码 (Code): 智能合约字节码
  - 存储 (Storage): 持久化的键值存储

### 2.2 账户状态

每个账户的状态包含以下字段:

```go
type Account struct {
    Nonce    uint64      // 交易计数器 (EOA) 或创建的合约数量 (Contract)
    Balance  *big.Int    // Wei 单位的余额
    Root     common.Hash // 存储树的根哈希 (仅合约账户)
    CodeHash []byte      // 合约代码的哈希 (仅合约账户)
}
```

### 2.3 地址生成

**外部账户地址生成**:

```plaintext
1. 生成私钥 (256 位随机数)
   Private Key = random(256 bits)

2. 从私钥派生公钥 (使用 secp256k1 椭圆曲线)
   Public Key = secp256k1(Private Key)

3. 对公钥进行 Keccak-256 哈希,取后 20 字节
   Address = Keccak256(Public Key)[12:]
```

**合约账户地址生成**:

```plaintext
Address = Keccak256(RLP(creator_address, nonce))[12:]
```

---

## 3. 交易机制

### 3.1 交易结构

以太坊交易是状态转换的触发器。一个交易包含以下字段:

```javascript
{
  "type": "0x2",                    // 交易类型 (0x0=Legacy, 0x1=EIP-2930, 0x2=EIP-1559)
  "nonce": "0x3",                   // 发送者的交易计数器
  "from": "0xca57f3b40b...",        // 发送者地址
  "to": "0xce8dba5e4157...",        // 接收者地址 (null 表示合约创建)
  "value": "0x2386f26fc10000",     // 转账金额 (Wei)
  "gas": "0x5208",                  // Gas 限制 (21000)
  "maxFeePerGas": "0x908a9079",    // 最大 Gas 费用 (EIP-1559)
  "maxPriorityFeePerGas": "0x908a901f", // 最大优先费用 (EIP-1559)
  "input": "0x",                    // 交易数据 (合约调用或创建)
  "chainId": "0xaa36a7",           // 链 ID (防止跨链重放攻击)
  "v": "0x0",                       // 签名 v 值
  "r": "0x66e5d23ad156...",        // 签名 r 值
  "s": "0x647ff82be943..."         // 签名 s 值
}
```

### 3.2 交易类型

| 类型 | EIP | 描述 | 特性 |
|------|-----|------|------|
| **Legacy** | - | 传统交易 | 固定 `gasPrice` |
| **EIP-2930** | EIP-2930 | 访问列表交易 | 预声明访问的地址和存储槽 |
| **EIP-1559** | EIP-1559 | 动态费用交易 | `baseFee` + `priorityFee` |

### 3.3 交易生命周期

```mermaid
graph LR
    A[创建交易] --> B[签名交易]
    B --> C[广播到网络]
    C --> D[进入交易池]
    D --> E[被打包进区块]
    E --> F[执行交易]
    F --> G[状态更新]
    G --> H[交易完成]
```

### 3.4 实践: 发送交易

**使用 Geth JavaScript 控制台**:

```javascript
// 1. 查看账户列表 (需要 Clef 批准)
eth.accounts;

// 2. 查询账户余额
eth.getBalance(eth.accounts[0]);

// 3. 构造交易对象
var tx = {
  from: eth.accounts[0],
  to: eth.accounts[1],
  value: web3.toWei(0.1, "ether")
};

// 4. 发送交易 (需要 Clef 批准)
eth.sendTransaction(tx);
// 返回: "0x99d489d0bd984915fd370b307c2d39320860950666aac3f261921113ae4f95bb"

// 5. 查询交易详情
eth.getTransaction("0x99d489d0bd984915fd370b307c2d39320860950666aac3f261921113ae4f95bb");
```

**使用 JSON-RPC API**:

```bash
# 查询账户余额
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc":"2.0",
    "method":"eth_getBalance",
    "params":["0xca57f3b40b42fcce3c37b8d18adbca5260ca72ec", "latest"],
    "id":1
  }'

# 发送交易 (需要 Clef 批准)
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc":"2.0",
    "method":"eth_sendTransaction",
    "params":[{
      "from": "0xca57f3b40b42fcce3c37b8d18adbca5260ca72ec",
      "to": "0xce8dba5e4157c2b284d8853afeeea259344c1653",
      "value": "0x16345785d8a0000"
    }],
    "id":1
  }'
```

---

## 4. Gas 机制与费用市场

### 4.1 什么是 Gas?

**Gas** 是以太坊中衡量计算工作量的单位。每个 EVM 操作都消耗一定数量的 Gas。

**为什么需要 Gas?**
- **防止滥用**: 限制无限循环和恶意代码
- **资源分配**: 根据计算复杂度收费
- **激励矿工/验证者**: 通过 Gas 费用奖励区块生产者

### 4.2 Gas 相关概念

| 概念 | 说明 | 示例 |
|------|------|------|
| **Gas Limit** | 交易愿意消耗的最大 Gas 数量 | 21000 (简单转账) |
| **Gas Used** | 交易实际消耗的 Gas 数量 | 21000 |
| **Gas Price** | 每单位 Gas 的价格 (Gwei) | 20 Gwei |
| **Transaction Fee** | 交易费用 = Gas Used × Gas Price | 0.00042 ETH |

### 4.3 EIP-1559 费用机制

自 London 硬分叉后,以太坊采用 EIP-1559 费用机制:

```plaintext
Transaction Fee = (Base Fee + Priority Fee) × Gas Used

其中:
- Base Fee: 协议动态调整的基础费用 (会被销毁)
- Priority Fee: 用户支付给验证者的小费
- Max Fee Per Gas: 用户愿意支付的最大费用
- Max Priority Fee Per Gas: 用户愿意支付的最大小费
```

**实际支付费用**:

```plaintext
Actual Fee = min(Max Fee Per Gas, Base Fee + Max Priority Fee Per Gas) × Gas Used
```

**费用退款**:

```plaintext
Refund = (Max Fee Per Gas - Actual Fee Per Gas) × Gas Used
```

### 4.4 常见操作的 Gas 消耗

| 操作 | Gas 消耗 | 说明 |
|------|----------|------|
| 简单转账 (EOA → EOA) | 21,000 | 最基本的交易 |
| ERC-20 转账 | ~65,000 | 调用合约的 `transfer` 方法 |
| Uniswap 交易 | ~150,000 | 复杂的 DeFi 交互 |
| 部署简单合约 | ~200,000+ | 取决于合约大小 |
| NFT Mint | ~100,000+ | 取决于合约逻辑 |

---

## 5. PoS 共识机制

### 5.1 从 PoW 到 PoS

2022 年 9 月 15 日,以太坊完成了 **The Merge**,从工作量证明 (PoW) 转向权益证明 (PoS)。

| 特性 | PoW (Ethash) | PoS (Beacon Chain) |
|------|--------------|---------------------|
| **共识方式** | 算力竞争 | 质押 ETH |
| **能源消耗** | 高 (~100 TWh/年) | 低 (~0.01 TWh/年) |
| **区块时间** | ~13 秒 | ~12 秒 (固定) |
| **最终性** | 概率性 | 确定性 (2 个 epoch) |
| **验证者要求** | 专业矿机 | 32 ETH 质押 |

### 5.2 PoS 核心概念

**Validator (验证者)**:
- 质押 32 ETH 成为验证者
- 负责提议和证明区块
- 获得奖励或受到惩罚 (Slashing)

**Epoch 和 Slot**:
- **Slot**: 12 秒,一个区块生产时间
- **Epoch**: 32 个 Slot (6.4 分钟)

**Finality (最终性)**:
- 区块在 2 个 Epoch 后达到最终性
- 最终性区块不可逆转

### 5.3 执行层与共识层

The Merge 后,以太坊采用双层架构:

```plaintext
┌─────────────────────────────────────┐
│      共识层 (Consensus Layer)        │
│   - Beacon Chain                    │
│   - PoS 共识                        │
│   - 验证者管理                       │
│   - 最终性保证                       │
└─────────────────────────────────────┘
              ↕ Engine API
┌─────────────────────────────────────┐
│      执行层 (Execution Layer)        │
│   - Geth / 其他客户端                │
│   - EVM 执行                        │
│   - 交易处理                         │
│   - 状态管理                         │
└─────────────────────────────────────┘
```

**Geth 的角色**:
- Geth 是执行层客户端
- 负责执行交易和维护状态
- 通过 Engine API 与共识层客户端通信

---

## 6. 以太坊网络架构

### 6.1 网络类型

| 网络 | Chain ID | 用途 | 特性 |
|------|----------|------|------|
| **Mainnet** | 1 | 主网 | 真实价值,生产环境 |
| **Sepolia** | 11155111 | 测试网 | PoS 测试网,推荐使用 |
| **Holesky** | 17000 | 测试网 | 质押测试网 |
| **Goerli** | 5 | 测试网 | 即将弃用 |

### 6.2 节点类型

| 节点类型 | 同步模式 | 存储需求 | 用途 |
|----------|----------|----------|------|
| **Full Node** | Snap Sync | ~800 GB | 验证所有区块,保留最近状态 |
| **Archive Node** | Full Sync | ~12 TB | 保留所有历史状态 |
| **Light Node** | Light Sync | ~1 GB | 仅验证区块头 |

---

## 7. 实践练习

### 练习 1: 理解账户状态

1. 使用 Geth 控制台查询账户余额
2. 计算账户地址的 Keccak-256 哈希
3. 理解 nonce 的作用

### 练习 2: 发送交易

1. 在测试网 (Sepolia) 创建两个账户
2. 从水龙头获取测试 ETH
3. 发送一笔转账交易
4. 查询交易详情和收据

### 练习 3: Gas 费用计算

1. 查询当前的 Base Fee 和 Priority Fee
2. 计算一笔交易的实际费用
3. 理解 EIP-1559 的费用退款机制

---

## 8. 总结

本章介绍了以太坊的核心概念:

- ✅ **账户模型**: EOA 和合约账户的区别
- ✅ **交易机制**: 交易结构、类型和生命周期
- ✅ **Gas 机制**: Gas 的作用和 EIP-1559 费用市场
- ✅ **PoS 共识**: The Merge 和双层架构
- ✅ **网络架构**: 主网、测试网和节点类型

---

## 9. 延伸阅读

- [以太坊黄皮书](https://ethereum.github.io/yellowpaper/paper.pdf) - 以太坊技术规范
- [EIP-1559 详解](https://eips.ethereum.org/EIPS/eip-1559) - 费用市场改革
- [The Merge 文档](https://ethereum.org/en/roadmap/merge/) - PoS 转换
- [Geth 官方文档](https://geth.ethereum.org/docs) - Geth 客户端文档

---

**下一章**: [02 - Geth 架构](./02-Geth-Architecture.md)

