# 03 - 节点设置与管理 (Node Setup and Management)

**难度**: ⭐⭐☆☆☆ | **预估学习时间**: 10-12 小时

---

## 📋 本章概述

本章将详细介绍如何安装、配置和管理 Geth 节点,包括不同的同步模式、网络配置和节点运维最佳实践。

### 学习目标

完成本章学习后,您将能够:

- 安装和配置 Geth 客户端
- 理解不同的同步模式及其适用场景
- 配置主网和测试网节点
- 掌握节点启动参数和配置文件
- 进行基本的节点运维和故障排查

---

## 1. Geth 安装

### 1.1 系统要求

**最低硬件要求**:

| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| **CPU** | 2 核 | 4 核以上 |
| **内存** | 8 GB | 16 GB 以上 |
| **存储** | 1 TB SSD | 2 TB NVMe SSD |
| **网络** | 10 Mbps | 25 Mbps 以上 |

**存储需求** (主网):

| 节点类型 | 存储需求 | 说明 |
|----------|----------|------|
| **Full Node (Snap Sync)** | ~800 GB | 保留最近 128 个区块状态 |
| **Archive Node** | ~12 TB | 保留所有历史状态 |
| **Light Node** | ~1 GB | 仅验证区块头 |

### 1.2 安装方法

#### 方法 1: 使用预编译二进制文件

**Linux/macOS**:

```bash
# 下载最新版本
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.13.0-b2baa3e6.tar.gz

# 解压
tar -xvf geth-linux-amd64-1.13.0-b2baa3e6.tar.gz

# 移动到系统路径
sudo mv geth-linux-amd64-1.13.0-b2baa3e6/geth /usr/local/bin/

# 验证安装
geth version
```

#### 方法 2: 使用包管理器

**Ubuntu/Debian**:

```bash
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

**macOS (Homebrew)**:

```bash
brew tap ethereum/ethereum
brew install ethereum
```

#### 方法 3: 从源码编译

```bash
# 安装 Go (需要 Go 1.21+)
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

# 克隆 Geth 仓库
git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum

# 编译
make geth

# 验证
build/bin/geth version
```

---

## 2. 节点同步模式

### 2.1 同步模式对比

| 同步模式 | 同步时间 | 存储需求 | 验证程度 | 适用场景 |
|----------|----------|----------|----------|----------|
| **Snap Sync** | ~6-12 小时 | ~800 GB | 中等 | 默认推荐,快速同步 |
| **Full Sync** | ~1-2 周 | ~800 GB | 完全 | 完整验证所有区块 |
| **Light Sync** | ~分钟级 | ~1 GB | 最低 | 轻客户端,仅验证区块头 |

### 2.2 Snap Sync (快照同步)

**特性**:
- **默认模式**: Geth 1.10+ 的默认同步方式
- **快速同步**: 从最近的区块开始同步
- **状态快照**: 并行下载状态快照
- **Healing 阶段**: 修复缺失的状态数据

**启动命令**:

```bash
# Snap Sync 是默认模式
geth --syncmode snap

# 或者简化为
geth
```

**同步过程**:

```plaintext
1. 下载区块头 (Headers)
2. 下载区块体 (Bodies)
3. 下载收据 (Receipts)
4. 并行下载状态快照 (State Snapshot)
5. Healing 阶段 (修复缺失数据)
```

**监控同步进度**:

```javascript
// 在 Geth 控制台中
eth.syncing

// 返回示例:
{
  currentBlock: 18500000,
  highestBlock: 18600000,
  knownStates: 500000000,
  pulledStates: 450000000,
  startingBlock: 18400000
}

// 同步完成后返回 false
```

### 2.3 Full Sync (完全同步)

**特性**:
- **完整验证**: 执行每个区块的所有交易
- **独立验证**: 不依赖其他节点的状态
- **慢速同步**: 需要 1-2 周时间

**启动命令**:

```bash
geth --syncmode full
```

### 2.4 Archive Node (归档节点)

**特性**:
- **保留所有历史状态**: 从创世区块到最新区块
- **巨大存储需求**: ~12 TB
- **快速历史查询**: 适合区块浏览器、分析工具

**启动命令**:

```bash
# Full Archive Node (完整归档)
geth --syncmode full --gcmode archive

# Partial Archive Node (部分归档,从同步点开始)
geth --syncmode snap --gcmode archive
```

**Path-based Archive Node** (推荐,v1.13+):

```bash
geth \
  --syncmode full \
  --gcmode archive \
  --history.state=0 \
  --state.scheme=path
```

---

## 3. 网络配置

### 3.1 主网 (Mainnet)

**Chain ID**: 1

**启动主网节点**:

```bash
geth --mainnet
# 或者简化为
geth
```

**完整配置示例**:

```bash
geth \
  --mainnet \
  --datadir /mnt/geth-mainnet \
  --syncmode snap \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8545 \
  --http.api eth,net,web3 \
  --ws \
  --ws.addr 0.0.0.0 \
  --ws.port 8546 \
  --ws.api eth,net,web3 \
  --authrpc.addr localhost \
  --authrpc.port 8551 \
  --authrpc.jwtsecret /path/to/jwtsecret \
  --maxpeers 50
```

### 3.2 测试网 (Testnet)

#### Sepolia (推荐)

**Chain ID**: 11155111  
**特性**: PoS 测试网,推荐用于开发测试

```bash
geth --sepolia --datadir /mnt/geth-sepolia
```

#### Holesky

**Chain ID**: 17000  
**特性**: 质押测试网

```bash
geth --holesky --datadir /mnt/geth-holesky
```

#### Goerli (即将弃用)

**Chain ID**: 5  
**特性**: 传统测试网,即将停止支持

```bash
geth --goerli --datadir /mnt/geth-goerli
```

### 3.3 开发者模式 (Developer Mode)

**特性**:
- **即时挖矿**: 有交易时立即出块
- **预充值账户**: 自动创建并充值账户
- **临时区块链**: 重启后数据丢失

**启动开发者模式**:

```bash
# 基本开发者模式
geth --dev

# 持久化数据
geth --dev --datadir dev-chain

# 集成 Remix IDE
geth --dev \
  --http \
  --http.api eth,web3,net \
  --http.corsdomain "https://remix.ethereum.org"
```

---

## 4. 节点配置

### 4.1 命令行参数

**常用参数**:

```bash
geth \
  --datadir /path/to/datadir \        # 数据目录
  --syncmode snap \                   # 同步模式
  --networkid 1 \                     # 网络 ID
  --port 30303 \                      # P2P 监听端口
  --maxpeers 50 \                     # 最大对等节点数
  --cache 4096 \                      # 内存缓存 (MB)
  --http \                            # 启用 HTTP-RPC
  --http.addr 0.0.0.0 \              # HTTP-RPC 监听地址
  --http.port 8545 \                  # HTTP-RPC 端口
  --http.api eth,net,web3 \          # HTTP-RPC API
  --ws \                              # 启用 WebSocket-RPC
  --ws.addr 0.0.0.0 \                # WebSocket-RPC 监听地址
  --ws.port 8546 \                    # WebSocket-RPC 端口
  --ws.api eth,net,web3 \            # WebSocket-RPC API
  --authrpc.addr localhost \          # Engine API 地址
  --authrpc.port 8551 \               # Engine API 端口
  --authrpc.jwtsecret /path/jwt \    # JWT Secret 文件
  --verbosity 3                       # 日志级别 (0-5)
```

### 4.2 配置文件

**生成配置文件**:

```bash
# 生成 Sepolia 测试网配置
geth --sepolia --syncmode snap dumpconfig > sepolia.toml

# 使用配置文件启动
geth --sepolia --config sepolia.toml
```

**配置文件示例** (`sepolia.toml`):

```toml
[Eth]
NetworkId = 11155111
SyncMode = "snap"
DatabaseCache = 512
TrieCleanCache = 154
TrieDirtyCache = 256
SnapshotCache = 102

[Eth.TxPool]
Locals = []
PriceLimit = 1
PriceBump = 10
AccountSlots = 16
GlobalSlots = 5120

[Node]
DataDir = "sepoliadata"
HTTPHost = ""
HTTPPort = 8545
HTTPVirtualHosts = ["localhost"]
HTTPModules = ["net", "web3", "eth"]
WSHost = ""
WSPort = 8546
WSModules = ["net", "web3", "eth"]
AuthAddr = "localhost"
AuthPort = 8551

[Node.P2P]
MaxPeers = 50
NoDiscovery = false
BootstrapNodes = [
  "enode://4e5e92199ee224a01932a377160aa432f31d0b351f84ab413a8e0a42f4f36476f8fb1cbe914af0d9aef0d51665c214cf653c651c4bbd9d5550a934f241f1682b@138.197.51.181:30303",
  "enode://143e11fb766781d22d92a2e33f8f104cddae4411a122295ed1fdb6638de96a6ce65f5b7c964ba3763bba27961738fef7d3ecc739268f3e5e771fb4c87b6234ba@146.190.1.103:30303"
]
```

### 4.3 数据目录管理

**数据目录结构**:

```plaintext
<datadir>/
├── geth/
│   ├── chaindata/          # 区块链数据
│   ├── lightchaindata/     # 轻客户端数据
│   ├── nodes/              # 节点发现数据
│   └── LOCK                # 实例锁
├── keystore/               # 账户密钥
└── geth.ipc                # IPC 文件
```

**指定数据目录**:

```bash
geth --datadir /mnt/ssd/geth-mainnet
```

**清理数据库** (谨慎操作):

```bash
# 停止 Geth
sudo systemctl stop geth

# 删除数据库 (保留 keystore)
geth removedb --datadir /path/to/datadir

# 重新初始化
geth --datadir /path/to/datadir
```

---

## 5. 节点运维

### 5.1 使用 Systemd 管理节点

**创建 Systemd 服务文件** (`/etc/systemd/system/geth.service`):

```ini
[Unit]
Description=Geth Ethereum Client
After=network.target

[Service]
Type=simple
User=geth
ExecStart=/usr/local/bin/geth \
  --mainnet \
  --datadir /mnt/geth-mainnet \
  --syncmode snap \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8545 \
  --http.api eth,net,web3 \
  --authrpc.addr localhost \
  --authrpc.port 8551 \
  --authrpc.jwtsecret /etc/geth/jwtsecret \
  --maxpeers 50
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**管理服务**:

```bash
# 重新加载 Systemd 配置
sudo systemctl daemon-reload

# 启动 Geth
sudo systemctl start geth

# 停止 Geth
sudo systemctl stop geth

# 重启 Geth
sudo systemctl restart geth

# 开机自启
sudo systemctl enable geth

# 查看状态
sudo systemctl status geth

# 查看日志
sudo journalctl -u geth -f
```

### 5.2 数据库维护

#### 数据库检查

```bash
# 检查数据库存储使用情况
geth db inspect --datadir /path/to/datadir

# 查看数据库统计信息
geth db stats --datadir /path/to/datadir
```

#### 状态裁剪 (Offline Pruning)

```bash
# 停止 Geth
sudo systemctl stop geth

# 执行裁剪 (需要数小时)
geth snapshot prune-state --datadir /path/to/datadir

# 重启 Geth
sudo systemctl start geth
```

#### 历史数据裁剪 (History Pruning)

```bash
# 停止 Geth
sudo systemctl stop geth

# 裁剪历史数据 (删除 Merge 前的区块体和收据)
geth prune-history --datadir /path/to/datadir

# 重启 Geth
sudo systemctl start geth
```

### 5.3 监控节点状态

**使用 Geth 控制台**:

```bash
# 连接到 IPC
geth attach /path/to/datadir/geth.ipc

# 或连接到 HTTP-RPC
geth attach http://localhost:8545
```

**常用监控命令**:

```javascript
// 查看同步状态
eth.syncing

// 查看当前区块高度
eth.blockNumber

// 查看对等节点数量
net.peerCount

// 查看对等节点详情
admin.peers

// 查看节点信息
admin.nodeInfo

// 查看交易池状态
txpool.status
```

---

## 6. 实践练习

### 练习 1: 搭建 Sepolia 测试网节点

1. 安装 Geth
2. 生成 JWT Secret
3. 启动 Sepolia 节点
4. 监控同步进度

```bash
# 生成 JWT Secret
openssl rand -hex 32 > jwtsecret

# 启动节点
geth --sepolia \
  --datadir /mnt/geth-sepolia \
  --http \
  --http.api eth,net,web3 \
  --authrpc.jwtsecret ./jwtsecret

# 监控同步
geth attach /mnt/geth-sepolia/geth.ipc --exec eth.syncing
```

### 练习 2: 配置 Systemd 服务

1. 创建 Systemd 服务文件
2. 启动并启用服务
3. 查看日志和状态

### 练习 3: 数据库维护

1. 检查数据库大小
2. 执行状态裁剪
3. 验证裁剪效果

---

## 7. 故障排查

### 常见问题

**问题 1: 同步卡住**

```bash
# 检查对等节点
geth attach --exec admin.peers

# 手动添加对等节点
admin.addPeer("enode://...")

# 重启节点
sudo systemctl restart geth
```

**问题 2: 磁盘空间不足**

```bash
# 检查磁盘使用
df -h

# 执行状态裁剪
geth snapshot prune-state --datadir /path/to/datadir
```

**问题 3: 内存不足**

```bash
# 减少缓存大小
geth --cache 2048  # 默认 4096 MB
```

---

## 8. 总结

本章介绍了 Geth 节点的设置与管理:

- ✅ **安装方法**: 预编译二进制、包管理器、源码编译
- ✅ **同步模式**: Snap Sync、Full Sync、Archive Node
- ✅ **网络配置**: 主网、测试网、开发者模式
- ✅ **节点配置**: 命令行参数、配置文件
- ✅ **节点运维**: Systemd 管理、数据库维护、监控

---

## 9. 延伸阅读

- [Geth 安装文档](https://geth.ethereum.org/docs/getting-started/installing-geth)
- [同步模式详解](https://geth.ethereum.org/docs/fundamentals/sync-modes)
- [配置文件参考](https://geth.ethereum.org/docs/fundamentals/config-files)
- [命令行选项](https://geth.ethereum.org/docs/fundamentals/command-line-options)

---

**上一章**: [02 - Geth 架构](./02-Geth-Architecture.md)  
**下一章**: [04 - 账户与安全](./04-Account-and-Security.md)

