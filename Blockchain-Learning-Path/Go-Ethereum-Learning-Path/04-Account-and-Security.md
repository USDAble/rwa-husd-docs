# 04 - 账户与安全 (Account and Security)

**难度**: ⭐⭐☆☆☆ | **预估学习时间**: 8-10 小时

---

## 📋 本章概述

本章将介绍以太坊账户管理和安全最佳实践,包括 Clef 外部签名器的使用、密钥库管理和私钥保护策略。

### 学习目标

完成本章学习后,您将能够:

- 理解以太坊账户的密钥管理机制
- 使用 Clef 进行安全的账户管理
- 掌握密钥库的创建和备份
- 实施私钥保护最佳实践
- 理解交易签名流程

---

## 1. 以太坊账户基础

### 1.1 账户组成

以太坊账户由以下部分组成:

```plaintext
私钥 (Private Key)
  ↓ secp256k1 椭圆曲线
公钥 (Public Key)
  ↓ Keccak-256 哈希,取后 20 字节
地址 (Address)
```

**示例**:

```plaintext
私钥: 0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
公钥: 0x04a1b2c3d4e5f6...
地址: 0xca57f3b40b42fcce3c37b8d18adbca5260ca72ec
```

### 1.2 密钥安全原则

| 原则 | 说明 | 重要性 |
|------|------|--------|
| **私钥不可恢复** | 丢失私钥 = 永久丢失资产 | ⭐⭐⭐⭐⭐ |
| **私钥不可共享** | 任何人获得私钥 = 完全控制账户 | ⭐⭐⭐⭐⭐ |
| **加密存储** | 私钥必须加密存储 | ⭐⭐⭐⭐⭐ |
| **离线备份** | 多地备份,防止单点故障 | ⭐⭐⭐⭐☆ |
| **定期审计** | 检查账户安全性 | ⭐⭐⭐☆☆ |

---

## 2. Clef 外部签名器

### 2.1 什么是 Clef?

**Clef** 是 Geth 官方的外部签名器和账户管理工具,用于替代已弃用的 `personal` API。

**核心特性**:
- **外部签名**: 将私钥管理与 Geth 分离
- **规则引擎**: 自动化签名策略
- **安全存储**: 加密存储账户密码
- **审计日志**: 记录所有签名操作

### 2.2 Clef 架构

```plaintext
┌─────────────────────────────────────┐
│         应用程序 (DApp/Geth)         │
└────────────────┬────────────────────┘
                 │ JSON-RPC
┌────────────────▼────────────────────┐
│              Clef                   │
│  ┌──────────────────────────────┐  │
│  │      规则引擎 (Rules)         │  │
│  └──────────────────────────────┘  │
│  ┌──────────────────────────────┐  │
│  │      密钥库 (Keystore)        │  │
│  └──────────────────────────────┘  │
│  ┌──────────────────────────────┐  │
│  │   凭证存储 (Credentials)      │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
```

### 2.3 安装和初始化 Clef

**初始化 Clef**:

```bash
# 初始化 Clef (首次使用)
clef init

# 输出示例:
# WARNING!
# Clef is an account management tool. It may, like any software, contain bugs.
# Please take care to
# - backup your keystore files,
# - verify that the keystore(s) can be opened with your password.
# Clef is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY
# ...
# Enter 'ok' to proceed:
# > ok
# 
# The master seed of clef will be locked with a password.
# Please specify a password. Do not forget this password!
# Password:
# Repeat password:
# 
# A master seed has been generated into /home/user/.clef/masterseed.json
```

**指定配置目录**:

```bash
clef init --configdir /path/to/clef-config
```

### 2.4 创建账户

**使用 Clef 创建新账户**:

```bash
# 创建新账户
clef newaccount --keystore /path/to/keystore

# 输出示例:
# Please enter a password for the new account:
# Repeat password:
# 
# Generated account 0x168bc315a2ee09042d83d7c5811b533620531f67
```

**列出账户**:

```bash
# 列出所有账户
clef list-accounts --keystore /path/to/keystore

# 输出示例:
# 0x168bc315a2ee09042d83d7c5811b533620531f67
# 0x0b85e5a13e118466159b1e1b6a4234e5f9f784bb
```

### 2.5 启动 Clef

**基本启动**:

```bash
clef \
  --keystore /path/to/keystore \
  --configdir /path/to/clef-config \
  --chainid 11155111  # Sepolia 测试网
```

**启用 HTTP API**:

```bash
clef \
  --keystore /path/to/keystore \
  --configdir /path/to/clef-config \
  --chainid 11155111 \
  --http \
  --http.addr localhost \
  --http.port 8550
```

**Clef 启动日志**:

```log
INFO [10-16|15:30:00.000] Starting Clef
INFO [10-16|15:30:00.001] IPC endpoint opened url=/home/user/.clef/clef.ipc
INFO [10-16|15:30:00.002] HTTP endpoint opened url=http://localhost:8550
```

### 2.6 Geth 与 Clef 集成

**启动 Geth 并连接 Clef**:

```bash
geth \
  --sepolia \
  --datadir /path/to/datadir \
  --signer /path/to/clef/clef.ipc \
  --http \
  --http.api eth,net,web3
```

**或使用 HTTP 连接**:

```bash
geth \
  --sepolia \
  --datadir /path/to/datadir \
  --signer http://localhost:8550 \
  --http \
  --http.api eth,net,web3
```

---

## 3. 账户操作

### 3.1 查询账户列表

**使用 Geth 控制台**:

```javascript
// 连接到 Geth
geth attach /path/to/datadir/geth.ipc

// 查询账户列表 (需要 Clef 批准)
eth.accounts

// Clef 会提示:
// Request context:
//   NA - ipc - NA
// 
// List available accounts?
// Approve? [y/N]:
```

**使用 JSON-RPC**:

```bash
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc":"2.0",
    "method":"eth_accounts",
    "params":[],
    "id":1
  }'
```

### 3.2 查询账户余额

```javascript
// 查询余额
eth.getBalance("0x168bc315a2ee09042d83d7c5811b533620531f67")

// 返回 Wei 单位
// 1000000000000000000

// 转换为 Ether
web3.fromWei(eth.getBalance("0x168bc315a2ee09042d83d7c5811b533620531f67"), "ether")
// "1"
```

### 3.3 发送交易

**使用 Geth 控制台**:

```javascript
// 构造交易
var tx = {
  from: "0x168bc315a2ee09042d83d7c5811b533620531f67",
  to: "0x0b85e5a13e118466159b1e1b6a4234e5f9f784bb",
  value: web3.toWei(0.1, "ether")
};

// 发送交易 (需要 Clef 批准)
eth.sendTransaction(tx);
```

**Clef 批准提示**:

```plaintext
--------- Transaction request-------------
to: 0x0b85e5a13e118466159b1e1b6a4234e5f9f784bb
from: 0x168bc315a2ee09042d83d7c5811b533620531f67 [chksum ok]
value: 100000000000000000 wei
gas: 0x5208 (21000)
maxFeePerGas: 2425000057 wei
maxPriorityFeePerGas: 2424999967 wei
nonce: 0x0 (0)
chainid: 0xaa36a7
-------------------------------------------
Approve? [y/N]:
> y
Please enter the password for account 0x168bc315a2ee09042d83d7c5811b533620531f67
Password:
```

---

## 4. 密钥库管理

### 4.1 密钥库文件格式

**密钥库文件** (`UTC--<timestamp>--<address>`):

```json
{
  "address": "168bc315a2ee09042d83d7c5811b533620531f67",
  "crypto": {
    "cipher": "aes-128-ctr",
    "ciphertext": "a1b2c3d4e5f6...",
    "cipherparams": {
      "iv": "1234567890abcdef"
    },
    "kdf": "scrypt",
    "kdfparams": {
      "dklen": 32,
      "n": 262144,
      "p": 1,
      "r": 8,
      "salt": "abcdef1234567890..."
    },
    "mac": "fedcba0987654321..."
  },
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "version": 3
}
```

**加密参数**:
- **cipher**: AES-128-CTR 加密算法
- **kdf**: Scrypt 密钥派生函数
- **n**: Scrypt 成本参数 (262144 = 标准强度)
- **mac**: 消息认证码,验证密码正确性

### 4.2 导入和导出账户

**导入私钥**:

```bash
# 使用 Clef 导入私钥
clef importraw <hexkey>

# 或使用 Geth (已弃用)
geth account import --datadir /path/to/datadir ./keyfile
```

**导出账户** (不推荐):

Geth 和 Clef 不提供直接导出私钥的功能,以防止意外泄露。如需导出,可以:
1. 手动解密密钥库文件 (需要密码)
2. 使用第三方工具 (风险高)

### 4.3 备份密钥库

**备份策略**:

```bash
# 1. 备份整个 keystore 目录
tar -czf keystore-backup-$(date +%Y%m%d).tar.gz /path/to/keystore

# 2. 加密备份文件
gpg -c keystore-backup-20231016.tar.gz

# 3. 存储到多个位置
# - 本地加密硬盘
# - 离线 USB 驱动器
# - 云存储 (加密后)
```

**恢复密钥库**:

```bash
# 1. 解密备份文件
gpg -d keystore-backup-20231016.tar.gz.gpg > keystore-backup-20231016.tar.gz

# 2. 解压到 keystore 目录
tar -xzf keystore-backup-20231016.tar.gz -C /path/to/keystore
```

---

## 5. Clef 规则引擎

### 5.1 什么是规则引擎?

Clef 的规则引擎允许自动化签名策略,无需每次手动批准。

**规则文件** (`rules.js`):

```javascript
// 自动批准小额转账
function ApproveTx(req) {
  var limit = new BigNumber("100000000000000000"); // 0.1 ETH
  var value = new BigNumber(req.transaction.value);
  
  if (value.lt(limit)) {
    return "Approve";
  }
  return "Reject";
}

// 自动批准账户列表请求 (仅 IPC)
function ApproveListing(req) {
  if (req.metadata.scheme == "ipc") {
    return "Approve";
  }
}
```

### 5.2 启用规则引擎

```bash
# 证明规则文件 (首次使用)
clef attest --configdir /path/to/clef-config rules.js

# 启动 Clef 并加载规则
clef \
  --keystore /path/to/keystore \
  --configdir /path/to/clef-config \
  --chainid 11155111 \
  --rules rules.js
```

### 5.3 存储账户密码

Clef 可以安全存储账户密码,实现自动解锁:

```bash
# 存储账户密码
clef setpw 0x168bc315a2ee09042d83d7c5811b533620531f67

# 输出:
# Please enter a password to store for this address:
# Password:
# Repeat password:
# 
# Please provide the master password to unlock the credential store
# Password:
# 
# Credential stored successfully
```

**凭证存储文件** (`credentials.json`):

```json
{
  "0x168bc315a2ee09042d83d7c5811b533620531f67": {
    "iv": "6SC062CfaUW8uSqH",
    "c": "C+S5kaJyrarrxrAESs4EmPjL5zmg5tRh0Q=="
  }
}
```

---

## 6. 安全最佳实践

### 6.1 私钥保护

| 实践 | 说明 | 优先级 |
|------|------|--------|
| **永不明文存储** | 私钥必须加密存储 | ⭐⭐⭐⭐⭐ |
| **使用硬件钱包** | Ledger、Trezor 等 | ⭐⭐⭐⭐⭐ |
| **离线签名** | 冷钱包签名,热钱包广播 | ⭐⭐⭐⭐☆ |
| **多重签名** | 需要多个签名才能执行 | ⭐⭐⭐⭐☆ |
| **定期轮换** | 定期更换账户 | ⭐⭐⭐☆☆ |

### 6.2 密码管理

**强密码要求**:
- 至少 16 个字符
- 包含大小写字母、数字、特殊字符
- 不使用字典单词
- 不重复使用密码

**密码存储**:
- 使用密码管理器 (1Password、Bitwarden)
- 离线备份密码
- 不在代码或配置文件中硬编码密码

### 6.3 网络安全

**防火墙配置**:

```bash
# 仅允许本地访问 HTTP-RPC
--http.addr 127.0.0.1

# 仅允许特定 IP 访问
--http.addr 192.168.1.100

# 限制 API 暴露
--http.api eth,net,web3  # 不暴露 admin, debug, personal
```

**使用 HTTPS 和 WSS**:

```bash
# 使用反向代理 (Nginx) 提供 HTTPS
# 不直接暴露 Geth HTTP-RPC
```

---

## 7. 实践练习

### 练习 1: 使用 Clef 创建和管理账户

1. 初始化 Clef
2. 创建两个测试账户
3. 查询账户列表和余额
4. 发送测试交易

### 练习 2: 配置 Clef 规则引擎

1. 编写规则文件
2. 证明规则文件
3. 启动 Clef 并加载规则
4. 测试自动批准功能

### 练习 3: 备份和恢复密钥库

1. 备份 keystore 目录
2. 加密备份文件
3. 删除原 keystore
4. 恢复备份并验证

---

## 8. 总结

本章介绍了以太坊账户管理和安全:

- ✅ **账户基础**: 私钥、公钥、地址的关系
- ✅ **Clef 签名器**: 外部签名器的安装和使用
- ✅ **密钥库管理**: 创建、备份、恢复账户
- ✅ **规则引擎**: 自动化签名策略
- ✅ **安全最佳实践**: 私钥保护、密码管理、网络安全

---

## 9. 延伸阅读

- [Clef 官方文档](https://geth.ethereum.org/docs/tools/clef/introduction)
- [账户管理指南](https://geth.ethereum.org/docs/fundamentals/account-management)
- [Clef 规则引擎](https://geth.ethereum.org/docs/tools/clef/rules)
- [安全最佳实践](https://ethereum.org/en/developers/docs/security/)

---

**上一章**: [03 - 节点设置与管理](./03-Node-Setup-and-Management.md)  
**下一章**: [05 - EVM 深入解析](./05-EVM-Deep-Dive.md)

