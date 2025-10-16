# 05-EVM-Deep-Dive.md - EVM 深入解析

**难度**: ⭐⭐⭐☆☆  
**预估学习时间**: 12-15 小时  
**前置知识**: 以太坊基础、Geth 架构、账户与交易机制

---

## 学习目标

通过本文档的学习,你将能够:

1. 深入理解 EVM (Ethereum Virtual Machine) 的架构和执行模型
2. 掌握 EVM 操作码 (Opcodes) 的分类和 Gas 计算机制
3. 理解 EVM 的存储模型 (Stack、Memory、Storage)
4. 学会使用 Geth 的 EVM 追踪工具进行调试和分析
5. 理解智能合约字节码的结构和执行流程

---

## 1. EVM 架构概述

### 1.1 什么是 EVM?

**EVM (Ethereum Virtual Machine)** 是以太坊的核心组件,是一个基于栈的虚拟机,负责执行智能合约字节码。

**核心特性**:
- **图灵完备**: 支持任意复杂的计算逻辑
- **确定性执行**: 相同输入保证相同输出
- **Gas 计量**: 每个操作都有明确的 Gas 成本
- **沙箱隔离**: 合约执行环境与外部隔离

### 1.2 EVM 执行模型

```
┌─────────────────────────────────────────┐
│          Transaction Input              │
│  (Bytecode + Calldata + Gas Limit)      │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│            EVM Execution                │
│  ┌─────────────────────────────────┐   │
│  │  Program Counter (PC)           │   │
│  │  Stack (256-bit words)          │   │
│  │  Memory (byte array)            │   │
│  │  Storage (persistent key-value) │   │
│  └─────────────────────────────────┘   │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│          Execution Result               │
│  (Return Data + Gas Used + State Δ)    │
└─────────────────────────────────────────┘
```

---

## 2. EVM 操作码 (Opcodes)

### 2.1 操作码分类

EVM 操作码按功能分为以下类别:

| 类别 | 操作码示例 | 功能描述 |
|------|-----------|---------|
| **算术运算** | `ADD`, `SUB`, `MUL`, `DIV`, `MOD` | 基本数学运算 |
| **比较运算** | `LT`, `GT`, `EQ`, `ISZERO` | 逻辑比较 |
| **位运算** | `AND`, `OR`, `XOR`, `NOT`, `BYTE` | 位操作 |
| **栈操作** | `PUSH1-32`, `POP`, `DUP1-16`, `SWAP1-16` | 栈管理 |
| **内存操作** | `MLOAD`, `MSTORE`, `MSTORE8` | 内存读写 |
| **存储操作** | `SLOAD`, `SSTORE` | 持久化存储 |
| **控制流** | `JUMP`, `JUMPI`, `JUMPDEST`, `STOP`, `RETURN` | 程序流程控制 |
| **环境信息** | `ADDRESS`, `CALLER`, `CALLVALUE`, `GASPRICE` | 获取交易/区块信息 |
| **区块信息** | `BLOCKHASH`, `COINBASE`, `TIMESTAMP`, `NUMBER` | 获取区块数据 |
| **合约调用** | `CALL`, `DELEGATECALL`, `STATICCALL`, `CREATE` | 合约交互 |
| **日志** | `LOG0-4` | 事件日志 |

### 2.2 常用操作码示例

#### 栈操作示例

```solidity
// Solidity 代码
uint256 a = 10;
uint256 b = 20;
uint256 c = a + b;

// 对应的 EVM 字节码 (简化)
PUSH1 0x0a    // 将 10 压入栈
PUSH1 0x14    // 将 20 压入栈
ADD           // 弹出两个值,相加后压入栈
PUSH1 0x00    // 存储位置
MSTORE        // 将结果存入内存
```

#### 存储操作示例

```solidity
// Solidity 代码
contract Storage {
    uint256 value;  // 存储槽 0
    
    function store(uint256 num) public {
        value = num;
    }
}

// 对应的 EVM 字节码 (简化)
PUSH1 0x00    // 存储槽 0
SLOAD         // 读取存储
PUSH1 0x00    // 存储槽 0
SSTORE        // 写入存储
```

---

## 3. EVM 存储模型

### 3.1 三种存储区域

#### Stack (栈)

- **容量**: 最大 1024 个 256-bit 字
- **访问**: LIFO (后进先出)
- **Gas 成本**: 最低 (3 Gas)
- **用途**: 临时计算

#### Memory (内存)

- **容量**: 动态扩展 (按需分配)
- **访问**: 字节数组,随机访问
- **Gas 成本**: 中等 (3 Gas + 扩展成本)
- **生命周期**: 交易执行期间
- **用途**: 函数参数、返回值、临时数据

#### Storage (存储)

- **容量**: 2^256 个 256-bit 槽位
- **访问**: 键值对 (key-value)
- **Gas 成本**: 最高 (20,000 Gas 写入)
- **生命周期**: 永久持久化
- **用途**: 合约状态变量

### 3.2 Gas 成本对比

```javascript
// 在 Geth 控制台测试 Gas 成本
var contract = eth.contract(abi).at(address);

// Stack 操作 (最便宜)
// ADD: 3 Gas

// Memory 操作 (中等)
// MSTORE: 3 Gas + memory_expansion_cost

// Storage 操作 (最贵)
// SSTORE (从 0 到非 0): 20,000 Gas
// SSTORE (从非 0 到非 0): 5,000 Gas
// SLOAD: 2,100 Gas (冷访问), 100 Gas (热访问)
```

---

## 4. EVM 追踪与调试

### 4.1 使用 debug_traceTransaction

Geth 提供了强大的 EVM 追踪工具,可以查看每个操作码的执行细节。

#### 基础追踪

```javascript
// 在 Geth 控制台执行
debug.traceTransaction('0xfc9359e49278b7ba99f59edac0e3de49956e46e530a53c15aa71226b7aa92c6f');
```

**返回结果结构**:

```json
{
  "gas": 25523,
  "failed": false,
  "returnValue": "",
  "structLogs": [
    {
      "pc": 0,
      "op": "PUSH1",
      "gas": 64580,
      "gasCost": 3,
      "depth": 1,
      "stack": [],
      "memory": null,
      "storage": {}
    },
    {
      "pc": 2,
      "op": "PUSH1",
      "gas": 64577,
      "gasCost": 3,
      "depth": 1,
      "stack": ["0x60"],
      "memory": null,
      "storage": {}
    }
  ]
}
```

#### 高级追踪选项

```javascript
// 启用内存和返回数据追踪
debug.traceTransaction('0x...', {
  enableMemory: true,
  disableStack: false,
  disableStorage: false,
  enableReturnData: true
});
```

### 4.2 内置追踪器 (Built-in Tracers)

Geth 提供了多种内置追踪器:

#### opcountTracer - 统计操作码数量

```javascript
debug.traceCall({
  from: '0x35a9f94af726f07b5162df7e828cc9dc8439e7d0',
  to: '0xc8ba32cab1757528daf49033e3673fae77dcf05d',
  data: '0xd1a2eab2...'
}, 'latest', {tracer: 'opcountTracer'});

// 返回: 总操作码执行次数
```

#### unigramTracer - 操作码频率统计

```javascript
debug.traceCall({...}, 'latest', {tracer: 'unigramTracer'});

// 返回示例:
{
  "ADD": 36,
  "AND": 23,
  "CALLDATALOAD": 6,
  "DUP1": 29,
  "PUSH1": 45,
  "SLOAD": 12,
  "SSTORE": 5
}
```

#### bigramTracer - 操作码序列分析

```javascript
debug.traceCall({...}, 'latest', {tracer: 'bigramTracer'});

// 返回示例:
{
  "ADD-ADD": 5,
  "PUSH1-DUP1": 12,
  "SLOAD-ISZERO": 3
}
```

---

## 5. 智能合约字节码分析

### 5.1 字节码结构

智能合约字节码通常包含以下部分:

```
┌────────────────────────────────────┐
│  Contract Creation Code            │
│  (部署时执行,返回 Runtime Code)     │
├────────────────────────────────────┤
│  Runtime Code                      │
│  ├─ Function Dispatcher (函数选择器)│
│  ├─ Function 1 Code                │
│  ├─ Function 2 Code                │
│  └─ ...                            │
├────────────────────────────────────┤
│  Metadata (可选)                    │
│  (Solidity 编译器元数据)            │
└────────────────────────────────────┘
```

### 5.2 函数选择器 (Function Selector)

```solidity
// Solidity 合约
contract Example {
    function store(uint256 num) public { ... }
    function retrieve() public view returns (uint256) { ... }
}

// 函数签名哈希 (前 4 字节)
// store(uint256) -> 0x6057361d
// retrieve() -> 0x2e64cec1
```

**字节码中的函数分发逻辑**:

```
PUSH1 0x00
CALLDATALOAD      // 加载 calldata 前 32 字节
PUSH1 0xE0
SHR               // 右移 224 位,提取前 4 字节 (函数选择器)
DUP1
PUSH4 0x6057361d  // store(uint256) 的选择器
EQ
PUSH2 0x0050
JUMPI             // 如果匹配,跳转到 store 函数代码
DUP1
PUSH4 0x2e64cec1  // retrieve() 的选择器
EQ
PUSH2 0x0070
JUMPI             // 如果匹配,跳转到 retrieve 函数代码
```

---

## 6. 实践练习

### 练习 1: 追踪简单交易

```bash
# 1. 启动 Geth 开发模式
geth --dev --http --http.api eth,web3,net,debug

# 2. 部署简单合约并调用
# 3. 使用 debug.traceTransaction 分析执行过程
```

### 练习 2: 分析 Gas 消耗

```javascript
// 在 Geth 控制台
var tx = eth.getTransaction('0x...');
var receipt = eth.getTransactionReceipt('0x...');
var trace = debug.traceTransaction('0x...');

console.log('Gas Limit:', tx.gas);
console.log('Gas Used:', receipt.gasUsed);
console.log('Total Opcodes:', trace.structLogs.length);
```

### 练习 3: 自定义追踪器

```javascript
// 统计 SLOAD/SSTORE 操作
var storageTracer = {
  retVal: [],
  step: function(log, db) {
    if(log.op.toNumber() == 0x54) {  // SLOAD
      this.retVal.push(log.getPC() + ": SLOAD " + log.stack.peek(0).toString(16));
    }
    if(log.op.toNumber() == 0x55) {  // SSTORE
      this.retVal.push(log.getPC() + ": SSTORE " + 
        log.stack.peek(0).toString(16) + " <- " + 
        log.stack.peek(1).toString(16));
    }
  },
  fault: function(log, db) {
    this.retVal.push("FAULT: " + JSON.stringify(log));
  },
  result: function(ctx, db) {
    return this.retVal;
  }
};

debug.traceTransaction('0x...', {tracer: storageTracer});
```

---

## 7. 总结

本文档深入介绍了 EVM 的核心概念:

- **EVM 架构**: 基于栈的虚拟机,确定性执行
- **操作码**: 11 大类操作码,涵盖算术、存储、控制流等
- **存储模型**: Stack (最快) < Memory (中等) < Storage (最慢/最贵)
- **追踪工具**: `debug_traceTransaction` 和内置追踪器
- **字节码结构**: 创建代码 + 运行时代码 + 函数分发器

---

## 8. 参考资料

- [Geth EVM Tracing Documentation](https://geth.ethereum.org/docs/developers/evm-tracing)
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [EVM Opcodes Reference](https://www.evm.codes/)
- [Solidity Documentation](https://docs.soliditylang.org/)

---

**下一步**: 学习 [06-Smart-Contract-Development.md](./06-Smart-Contract-Development.md) - 智能合约开发

