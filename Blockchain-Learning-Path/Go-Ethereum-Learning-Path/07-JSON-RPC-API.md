# 07-JSON-RPC-API.md - JSON-RPC API

**难度**: ⭐⭐⭐☆☆  
**预估学习时间**: 10-12 小时  
**前置知识**: 以太坊基础、Geth 节点管理、智能合约开发

---

## 学习目标

通过本文档的学习,你将能够:

1. 理解 JSON-RPC 协议和 Geth 的 RPC 架构
2. 掌握 Geth 的 HTTP、WebSocket 和 IPC 三种 RPC 接口
3. 熟练使用 `eth_*`、`net_*`、`web3_*` 等核心 API 命名空间
4. 学会使用订阅 (Subscriptions) 机制监听实时事件
5. 掌握使用 Web3.js/Ethers.js 库与 Geth 交互

---

## 1. JSON-RPC 协议概述

### 1.1 什么是 JSON-RPC?

**JSON-RPC** 是一种轻量级的远程过程调用 (RPC) 协议,使用 JSON 格式进行数据交换。

**请求格式**:

```json
{
  "jsonrpc": "2.0",
  "method": "eth_blockNumber",
  "params": [],
  "id": 1
}
```

**响应格式**:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x4b7"
}
```

### 1.2 Geth RPC 架构

```
┌─────────────────────────────────────────┐
│          Client Applications            │
│  (Web3.js, Ethers.js, curl, etc.)      │
└──────────────────┬──────────────────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
┌────────┐   ┌──────────┐   ┌──────────┐
│  HTTP  │   │WebSocket │   │   IPC    │
│  :8545 │   │  :8546   │   │ geth.ipc │
└────┬───┘   └─────┬────┘   └─────┬────┘
     │             │              │
     └─────────────┼──────────────┘
                   ▼
         ┌──────────────────┐
         │   RPC Server     │
         │  (API Modules)   │
         └────────┬─────────┘
                  │
    ┌─────────────┼─────────────┐
    ▼             ▼             ▼
┌────────┐   ┌────────┐   ┌────────┐
│  eth   │   │  net   │   │ admin  │
└────────┘   └────────┘   └────────┘
    ▼             ▼             ▼
┌────────┐   ┌────────┐   ┌────────┐
│ debug  │   │ txpool │   │ web3   │
└────────┘   └────────┘   └────────┘
```

---

## 2. RPC 接口配置

### 2.1 HTTP RPC 接口

**启用 HTTP RPC**:

```bash
# 基础启用
geth --http

# 自定义地址和端口
geth --http --http.addr 0.0.0.0 --http.port 8545

# 指定 API 命名空间
geth --http --http.api eth,net,web3

# 配置 CORS (跨域资源共享)
geth --http --http.corsdomain "https://remix.ethereum.org"

# 允许所有域 (不推荐用于生产环境)
geth --http --http.corsdomain '*'
```

**默认配置**:
- 地址: `127.0.0.1` (仅本地访问)
- 端口: `8545`
- API: `eth,net,web3`

### 2.2 WebSocket RPC 接口

**启用 WebSocket RPC**:

```bash
# 基础启用
geth --ws

# 自定义地址和端口
geth --ws --ws.addr 0.0.0.0 --ws.port 8546

# 指定 API 命名空间
geth --ws --ws.api eth,net,web3

# 配置允许的来源
geth --ws --ws.origins "http://myapp.example.com"

# 允许所有来源 (不推荐用于生产环境)
geth --ws --ws.origins '*'
```

**默认配置**:
- 地址: `127.0.0.1`
- 端口: `8546`
- API: `eth,net,web3`

### 2.3 IPC 接口

**IPC 默认路径**:
- **Linux/macOS**: `~/.ethereum/geth.ipc`
- **Windows**: `\\.\pipe\geth.ipc`

**自定义 IPC 路径**:

```bash
geth --ipcpath /path/to/custom.ipc
```

**禁用 IPC**:

```bash
geth --ipcdisable
```

**安全性**: IPC 是最安全的接口,默认启用所有 API 命名空间。

---

## 3. 核心 API 命名空间

### 3.1 eth 命名空间

#### 查询账户余额

```bash
# 使用 curl
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc": "2.0",
    "method": "eth_getBalance",
    "params": ["0xca57f3b40b42fcce3c37b8d18adbca5260ca72ec", "latest"],
    "id": 1
  }'

# 返回 (Wei 单位,十六进制)
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0xc7d54951f87f7c0"
}
```

#### 获取区块号

```bash
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc": "2.0",
    "method": "eth_blockNumber",
    "params": [],
    "id": 1
  }'

# 返回
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x4b7"  // 十六进制区块号
}
```

#### 发送交易

```bash
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc": "2.0",
    "method": "eth_sendTransaction",
    "params": [{
      "from": "0xca57f3b40b42fcce3c37b8d18adbca5260ca72ec",
      "to": "0xce8dba5e4157c2b284d8853afeeea259344c1653",
      "value": "0x16345785d8a0000"
    }],
    "id": 1
  }'

# 返回交易哈希
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0xac8b347d70a82805edb85fc136fc2c4e77d31677c2f9e4e7950e0342f0dc7e7c"
}
```

#### 调用智能合约 (eth_call)

```bash
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [{
      "to": "0xebe8efa441b9302a0d7eaecc277c09d20d684540",
      "data": "0x0be5b6ba"
    }, "latest"],
    "id": 1
  }'
```

### 3.2 net 命名空间

```javascript
// 在 Geth 控制台
net.listening  // 是否监听网络连接
net.peerCount  // 连接的对等节点数量
net.version    // 网络 ID (1=主网, 11155111=Sepolia)
```

**通过 RPC 调用**:

```bash
# 检查网络监听状态
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"net_listening","params":[],"id":1}'

# 获取对等节点数量
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}'
```

### 3.3 web3 命名空间

```javascript
// 在 Geth 控制台
web3.clientVersion  // Geth 客户端版本
web3.sha3("0x68656c6c6f20776f726c64")  // Keccak-256 哈希
```

### 3.4 admin 命名空间

**注意**: `admin` API 仅通过 IPC 或受信任的 HTTP/WS 连接暴露。

```javascript
// 在 Geth 控制台
admin.nodeInfo  // 节点信息
admin.peers     // 连接的对等节点详情
admin.addPeer("enode://...")  // 添加静态对等节点
admin.startHTTP("127.0.0.1", 8545)  // 动态启动 HTTP RPC
admin.stopHTTP()  // 停止 HTTP RPC
```

---

## 4. 订阅机制 (Subscriptions)

### 4.1 订阅新区块头

**WebSocket 连接**:

```javascript
// 使用 Web3.js
const Web3 = require('web3');
const web3 = new Web3('ws://localhost:8546');

// 订阅新区块头
const subscription = web3.eth.subscribe('newBlockHeaders', (error, blockHeader) => {
  if (error) {
    console.error(error);
    return;
  }
  console.log('New block:', blockHeader.number);
});

// 取消订阅
subscription.unsubscribe((error, success) => {
  if (success) {
    console.log('Successfully unsubscribed');
  }
});
```

**原始 JSON-RPC**:

```json
// 订阅请求
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "eth_subscribe",
  "params": ["newHeads"]
}

// 订阅响应
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0x9cef478923ff08bf67fde6c64013158d"
}

// 事件通知
{
  "jsonrpc": "2.0",
  "method": "eth_subscription",
  "params": {
    "subscription": "0x9cef478923ff08bf67fde6c64013158d",
    "result": {
      "difficulty": "0xd9263f42a87",
      "number": "0x65a8",
      "hash": "0x...",
      "timestamp": "0x5a2f3b01"
    }
  }
}
```

### 4.2 订阅日志 (Logs)

```javascript
// 订阅特定合约的事件
const subscription = web3.eth.subscribe('logs', {
  address: '0x123...',
  topics: ['0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef']
}, (error, log) => {
  if (!error) {
    console.log('Event:', log);
  }
});
```

### 4.3 订阅待处理交易

```javascript
const subscription = web3.eth.subscribe('pendingTransactions', (error, txHash) => {
  if (!error) {
    console.log('Pending tx:', txHash);
  }
});
```

---

## 5. 使用 Web3.js 库

### 5.1 安装和连接

```bash
npm install web3
```

```javascript
const Web3 = require('web3');

// HTTP 连接
const web3 = new Web3('http://localhost:8545');

// WebSocket 连接
const web3 = new Web3('ws://localhost:8546');

// IPC 连接
const web3 = new Web3(new Web3.providers.IpcProvider('/path/to/geth.ipc'));
```

### 5.2 常用操作

```javascript
// 获取账户列表
const accounts = await web3.eth.getAccounts();

// 获取余额
const balance = await web3.eth.getBalance(accounts[0]);
console.log(web3.utils.fromWei(balance, 'ether'), 'ETH');

// 发送交易
const tx = await web3.eth.sendTransaction({
  from: accounts[0],
  to: '0x...',
  value: web3.utils.toWei('0.1', 'ether')
});

// 获取区块
const block = await web3.eth.getBlock('latest');

// 获取交易收据
const receipt = await web3.eth.getTransactionReceipt(tx.transactionHash);
```

### 5.3 与智能合约交互

```javascript
const abi = [...];  // 合约 ABI
const address = '0x...';  // 合约地址

const contract = new web3.eth.Contract(abi, address);

// 调用只读方法
const value = await contract.methods.retrieve().call();

// 发送交易
const tx = await contract.methods.store(42).send({
  from: accounts[0],
  gas: 100000
});
```

---

## 6. 使用 Ethers.js 库

### 6.1 安装和连接

```bash
npm install ethers
```

```javascript
const { ethers } = require('ethers');

// HTTP 连接
const provider = new ethers.JsonRpcProvider('http://localhost:8545');

// WebSocket 连接
const provider = new ethers.WebSocketProvider('ws://localhost:8546');
```

### 6.2 常用操作

```javascript
// 获取区块号
const blockNumber = await provider.getBlockNumber();

// 获取余额
const balance = await provider.getBalance('0x...');
console.log(ethers.formatEther(balance), 'ETH');

// 发送交易
const wallet = new ethers.Wallet('0x...privateKey...', provider);
const tx = await wallet.sendTransaction({
  to: '0x...',
  value: ethers.parseEther('0.1')
});
await tx.wait();
```

### 6.3 与智能合约交互

```javascript
const abi = [...];
const address = '0x...';

const contract = new ethers.Contract(address, abi, provider);

// 调用只读方法
const value = await contract.retrieve();

// 发送交易
const contractWithSigner = contract.connect(wallet);
const tx = await contractWithSigner.store(42);
await tx.wait();
```

---

## 7. 实践练习

### 练习 1: 批量查询区块

```javascript
// 使用 JSON-RPC 批量请求
const fetch = require('node-fetch');

const requests = [];
for (let i = 100; i < 110; i++) {
  requests.push({
    method: 'eth_getBlockByNumber',
    params: [`0x${i.toString(16)}`, false],
    id: i - 100,
    jsonrpc: '2.0'
  });
}

const response = await fetch('http://127.0.0.1:8545', {
  method: 'POST',
  body: JSON.stringify(requests),
  headers: { 'Content-Type': 'application/json' }
});

const data = await response.json();
console.log(data);
```

### 练习 2: 监听新区块并分析

```javascript
web3.eth.subscribe('newBlockHeaders', async (error, blockHeader) => {
  if (!error) {
    const block = await web3.eth.getBlock(blockHeader.number, true);
    console.log(`Block ${block.number}: ${block.transactions.length} txs`);
  }
});
```

### 练习 3: 过滤合约事件

```javascript
const contract = new web3.eth.Contract(abi, address);

contract.events.Transfer({
  filter: { from: '0x...' },
  fromBlock: 'latest'
}, (error, event) => {
  console.log('Transfer event:', event.returnValues);
});
```

---

## 8. 总结

本文档介绍了 Geth JSON-RPC API 的核心内容:

- **RPC 接口**: HTTP (8545)、WebSocket (8546)、IPC (最安全)
- **API 命名空间**: `eth`, `net`, `web3`, `admin`, `debug`
- **订阅机制**: 实时监听区块、日志、待处理交易
- **客户端库**: Web3.js、Ethers.js

---

## 9. 参考资料

- [Geth JSON-RPC Documentation](https://geth.ethereum.org/docs/interacting-with-geth/rpc)
- [Ethereum JSON-RPC Specification](https://ethereum.github.io/execution-apis/api-documentation/)
- [Web3.js Documentation](https://web3js.readthedocs.io/)
- [Ethers.js Documentation](https://docs.ethers.org/)

---

**下一步**: 学习 [08-P2P-Network-and-Sync.md](./08-P2P-Network-and-Sync.md) - P2P 网络与同步

