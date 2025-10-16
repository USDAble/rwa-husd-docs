# 08-P2P-Network-and-Sync.md - P2P 网络与同步

**难度**: ⭐⭐⭐☆☆  
**预估学习时间**: 12-15 小时  
**前置知识**: 以太坊基础、Geth 节点管理、网络协议基础

---

## 学习目标

通过本文档的学习,你将能够:

1. 理解以太坊 P2P 网络架构和 devp2p 协议栈
2. 掌握节点发现机制 (Discovery v4/v5)
3. 理解 RLPx 加密传输协议
4. 掌握区块同步协议 (Snap Sync、Full Sync)
5. 学会使用 Geth 的对等节点管理工具

---

## 1. 以太坊 P2P 网络架构

### 1.1 网络层次结构

```
┌─────────────────────────────────────────┐
│      Application Layer (应用层)         │
│  ┌──────────┐  ┌──────────┐  ┌────────┐│
│  │   eth    │  │   snap   │  │  les   ││
│  │ (协议66+) │  │ (快照同步)│  │(轻客户端)││
│  └──────────┘  └──────────┘  └────────┘│
├─────────────────────────────────────────┤
│      Transport Layer (传输层)           │
│  ┌─────────────────────────────────┐   │
│  │  RLPx (加密传输协议)             │   │
│  │  - 握手 (ECIES 加密)             │   │
│  │  - 帧传输 (AES-256-CTR)          │   │
│  └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│      Discovery Layer (发现层)           │
│  ┌──────────────┐  ┌──────────────┐   │
│  │ Discovery v4 │  │ Discovery v5 │   │
│  │  (Kademlia)  │  │  (改进版)     │   │
│  └──────────────┘  └──────────────┘   │
├─────────────────────────────────────────┤
│      Network Layer (网络层)             │
│         UDP (发现) + TCP (传输)         │
└─────────────────────────────────────────┘
```

### 1.2 核心组件

- **devp2p**: 以太坊 P2P 协议栈
- **Discovery**: 节点发现协议 (基于 Kademlia DHT)
- **RLPx**: 加密传输协议
- **eth**: 以太坊主协议 (区块、交易传播)
- **snap**: 快照同步协议

---

## 2. 节点发现机制

### 2.1 Discovery v4 协议

**基于 Kademlia DHT** 的节点发现协议:

- **节点 ID**: 256-bit 公钥哈希
- **距离度量**: XOR 距离
- **K-桶**: 每个桶存储 16 个节点
- **UDP 端口**: 默认 30303

**核心消息类型**:

| 消息类型 | 功能 | 描述 |
|---------|------|------|
| **PING** | 探活 | 检查节点是否在线 |
| **PONG** | 响应 | 回应 PING 消息 |
| **FIND_NODE** | 查找 | 请求距离目标 ID 最近的节点 |
| **NEIGHBORS** | 响应 | 返回节点列表 |

**节点发现流程**:

```
┌──────────┐                    ┌──────────┐
│  Node A  │                    │  Node B  │
└─────┬────┘                    └─────┬────┘
      │                               │
      │  1. PING (检查 B 是否在线)     │
      │─────────────────────────────>│
      │                               │
      │  2. PONG (确认在线)            │
      │<─────────────────────────────│
      │                               │
      │  3. FIND_NODE (查找节点)       │
      │─────────────────────────────>│
      │                               │
      │  4. NEIGHBORS (返回节点列表)   │
      │<─────────────────────────────│
      │                               │
```

### 2.2 Discovery v5 协议

**改进特性**:

- **主题发现**: 支持按主题查找节点
- **更好的隐私**: 减少信息泄露
- **更高效**: 减少网络流量
- **ENR (Ethereum Node Record)**: 更丰富的节点信息

**启用 Discovery v5**:

```bash
geth --discovery.v5
```

### 2.3 Bootnodes (引导节点)

**作用**: 新节点启动时的初始连接点。

**配置自定义 Bootnodes**:

```bash
geth --bootnodes "enode://pubkey1@ip1:port1,enode://pubkey2@ip2:port2"
```

**主网默认 Bootnodes**:

```
enode://d860a01f9722d78051619d1e2351aba3f43f943f6f00718d1b9baa4101932a1f5011f16bb2b1bb35db20d6fe28fa0bf09636d26a87d31de9ec6203eeedb1f666@18.138.108.67:30303
enode://22a8232c3abc76a16ae9d6c3b164f98775fe226f0917b0ca871128a74a8e9630b458460865bab457221f1d448dd9791d24c4e5d88786180ac185df813a68d4de@3.209.45.79:30303
...
```

---

## 3. RLPx 加密传输协议

### 3.1 RLPx 握手流程

**ECIES (Elliptic Curve Integrated Encryption Scheme)** 加密握手:

```
┌──────────┐                    ┌──────────┐
│  Node A  │                    │  Node B  │
└─────┬────┘                    └─────┬────┘
      │                               │
      │  1. auth (A 的公钥 + 签名)     │
      │─────────────────────────────>│
      │                               │
      │  2. ack (B 的公钥 + 签名)      │
      │<─────────────────────────────│
      │                               │
      │  3. 协商会话密钥                │
      │  (ECDH 密钥交换)                │
      │                               │
      │  4. 加密通信 (AES-256-CTR)     │
      │<────────────────────────────>│
      │                               │
```

### 3.2 帧传输格式

**加密帧结构**:

```
┌────────────────────────────────────┐
│  Header (16 bytes, 加密)            │
│  - frame-size (3 bytes)            │
│  - header-data (13 bytes)          │
├────────────────────────────────────┤
│  Frame Data (加密)                  │
│  - protocol-id (1 byte)            │
│  - message-id (variable)           │
│  - payload (variable)              │
├────────────────────────────────────┤
│  Padding (对齐到 16 字节)           │
└────────────────────────────────────┘
```

---

## 4. 区块同步协议

### 4.1 同步模式对比

| 同步模式 | 速度 | 磁盘空间 | 验证程度 | 适用场景 |
|---------|------|---------|---------|---------|
| **Snap Sync** | 最快 | 中等 | 状态快照 + 区块头 | 默认推荐 |
| **Full Sync** | 慢 | 大 | 完整验证所有区块 | 归档节点 |
| **Light Sync** | 快 | 最小 | 仅区块头 | 轻客户端 |

### 4.2 Snap Sync (快照同步)

**工作原理**:

1. **下载区块头**: 从创世区块到最新区块
2. **下载状态快照**: 最新区块的完整状态
3. **验证快照**: 通过 Merkle 证明验证
4. **下载最近区块**: 快照点之后的完整区块

**优势**:
- 同步速度快 (数小时内完成)
- 磁盘空间适中
- 可以立即开始验证新区块

**启用 Snap Sync**:

```bash
geth --syncmode snap
```

### 4.3 Full Sync (完整同步)

**工作原理**:

1. **下载所有区块**: 从创世区块开始
2. **执行所有交易**: 重新计算每个区块的状态
3. **完整验证**: 验证所有区块和状态转换

**优势**:
- 最高安全性
- 完整历史数据

**劣势**:
- 同步时间长 (数周)
- 磁盘空间大 (>1TB)

**启用 Full Sync**:

```bash
geth --syncmode full
```

### 4.4 同步状态监控

```javascript
// 在 Geth 控制台
eth.syncing

// 返回示例:
{
  currentBlock: 15000000,
  highestBlock: 18000000,
  knownStates: 500000000,
  pulledStates: 450000000,
  startingBlock: 0
}

// 如果已同步完成,返回 false
```

---

## 5. 对等节点管理

### 5.1 查看连接的对等节点

```javascript
// 在 Geth 控制台
admin.peers

// 返回示例:
[{
  caps: ["eth/68", "snap/1"],
  enode: "enode://6a4576fb...@157.90.35.166:30303",
  id: "6b36f791352f15eb3ec4f67787074ab8ad9d487e...",
  name: "Geth/v1.13.14-stable/linux-amd64/go1.21.7",
  network: {
    inbound: false,
    localAddress: "172.17.0.2:33666",
    remoteAddress: "157.90.35.166:30303",
    static: false,
    trusted: false
  },
  protocols: {
    eth: { version: 68 },
    snap: { version: 1 }
  }
}]
```

### 5.2 添加静态对等节点

**方法 1: 命令行配置**

创建 `static-nodes.json`:

```json
[
  "enode://f4642fa65af50cfdea8fa7414a5def7bb7991478b768e296f5e4a54e8b995de102e0ceae2e826f293c481b5325f89be6d207b003382e18a8ecba66fbaf6416c0@33.4.2.1:30303",
  "enode://..."
]
```

放置在数据目录: `~/.ethereum/geth/static-nodes.json`

**方法 2: 运行时添加**

```javascript
// 在 Geth 控制台
admin.addPeer("enode://f4642fa65af50cfdea8fa7414a5def7bb7991478b768e296f5e4a54e8b995de102e0ceae2e826f293c481b5325f89be6d207b003382e18a8ecba66fbaf6416c0@33.4.2.1:30303");
```

### 5.3 添加受信任节点

```javascript
// 在 Geth 控制台
admin.addTrustedPeer("enode://...");

// 移除受信任节点
admin.removeTrustedPeer("enode://...");
```

**受信任节点特性**:
- 即使对等节点槽位已满,仍可连接
- 优先保持连接

### 5.4 网络配置选项

```bash
# 设置最大对等节点数
geth --maxpeers 50

# 设置最大待处理连接数
geth --maxpendpeers 10

# 禁用节点发现 (仅连接静态节点)
geth --nodiscover

# 设置监听端口
geth --port 30303

# 设置发现端口
geth --discovery.port 30303

# 限制网络通信到特定 IP 范围
geth --netrestrict "192.168.1.0/24"
```

---

## 6. devp2p 工具

### 6.1 安装 devp2p 工具

```bash
go install github.com/ethereum/go-ethereum/cmd/devp2p@latest
```

### 6.2 节点发现测试

```bash
# Ping 节点
devp2p discv4 ping <enode-url>

# 解析节点记录
devp2p discv4 resolve <enode-url>

# 爬取网络节点
devp2p discv4 crawl -timeout 30m all-nodes.json
```

### 6.3 ENR 工具

```bash
# 解码 ENR (Ethereum Node Record)
devp2p enrdump <base64-encoded-enr>

# 示例输出:
# Node ID: a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7
# URLv4: enode://a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7@3.209.45.79:30303
# Record Sequence: 1159
# eth: [eth/66 eth/67 eth/68]
# ip: 3.209.45.79
# secp256k1: 0x03ca634cae0d49acb401d8a4c6b6fe8c55b70d115bf400769cc1400f3258cd3138
# snap: [snap/1]
# tcp: 30303
# udp: 30303
```

---

## 7. 实践练习

### 练习 1: 监控网络连接

```javascript
// 在 Geth 控制台
setInterval(() => {
  console.log('Peers:', net.peerCount);
  console.log('Listening:', net.listening);
  console.log('Syncing:', eth.syncing);
}, 5000);
```

### 练习 2: 分析对等节点

```javascript
// 统计对等节点的协议版本
var peers = admin.peers;
var ethVersions = {};

peers.forEach(peer => {
  var version = peer.protocols.eth.version;
  ethVersions[version] = (ethVersions[version] || 0) + 1;
});

console.log('ETH Protocol Versions:', ethVersions);
```

### 练习 3: 搭建私有网络

```bash
# 节点 1
geth --datadir node1 --networkid 12345 --port 30303 --nodiscover console

# 获取节点 1 的 enode
> admin.nodeInfo.enode

# 节点 2
geth --datadir node2 --networkid 12345 --port 30304 --bootnodes "<node1-enode>" console

# 验证连接
> net.peerCount
```

---

## 8. 总结

本文档介绍了以太坊 P2P 网络的核心机制:

- **devp2p 协议栈**: Discovery + RLPx + eth/snap
- **节点发现**: Discovery v4 (Kademlia DHT) 和 v5
- **加密传输**: RLPx (ECIES 握手 + AES-256-CTR)
- **同步协议**: Snap Sync (推荐) vs Full Sync
- **对等节点管理**: 静态节点、受信任节点、网络配置

---

## 9. 参考资料

- [Geth P2P Documentation](https://geth.ethereum.org/docs/fundamentals/peer-to-peer)
- [devp2p Specifications](https://github.com/ethereum/devp2p)
- [Ethereum Wire Protocol](https://github.com/ethereum/devp2p/blob/master/caps/eth.md)
- [Discovery v4 Specification](https://github.com/ethereum/devp2p/blob/master/discv4.md)
- [RLPx Transport Protocol](https://github.com/ethereum/devp2p/blob/master/rlpx.md)

---

**下一步**: 学习 [09-EVM-Tracing-and-Debugging.md](./09-EVM-Tracing-and-Debugging.md) - EVM 追踪与调试

