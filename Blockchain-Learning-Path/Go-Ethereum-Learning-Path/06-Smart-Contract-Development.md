# 06-Smart-Contract-Development.md - 智能合约开发

**难度**: ⭐⭐⭐☆☆  
**预估学习时间**: 12-15 小时  
**前置知识**: 以太坊基础、EVM 深入解析、Solidity 基础

---

## 学习目标

通过本文档的学习,你将能够:

1. 掌握使用 Geth 开发模式进行本地智能合约开发
2. 学会使用 Remix IDE 与 Geth 集成开发合约
3. 理解智能合约的编译、部署和交互流程
4. 掌握使用 Go 语言与智能合约交互 (abigen 工具)
5. 学会使用 Hardhat/Foundry 等现代开发框架

---

## 1. Geth 开发模式 (Developer Mode)

### 1.1 什么是开发模式?

Geth 的 **Developer Mode** 提供了一个轻量级的本地测试环境:

- **即时出块**: 有交易时立即生成区块
- **预分配账户**: 自动创建并预充值的测试账户
- **零配置**: 无需创世区块配置
- **快速重置**: 重启即清空数据

### 1.2 启动开发模式

```bash
# 基础启动
geth --dev

# 启用 HTTP RPC (用于 Remix 连接)
geth --dev --http --http.api eth,web3,net --http.corsdomain "https://remix.ethereum.org"

# 自定义数据目录
geth --dev --datadir ./dev-chain --http --http.api eth,web3,net,debug
```

**启动后的日志**:

```
INFO [05-09|12:27:09.680] Setting new local account address=0x7Aa16266Ba3d309e3cb278B452b1A6307E52Fb62
INFO [05-09|12:27:09.680] Starting Geth in developer mode...
```

### 1.3 连接到开发节点

```bash
# 使用 IPC 连接
geth attach ipc:/path/to/geth.ipc

# 使用 HTTP 连接
geth attach http://localhost:8545
```

---

## 2. 使用 Remix IDE 开发合约

### 2.1 Remix 与 Geth 集成

**步骤**:

1. **启动 Geth 开发模式** (启用 CORS):

```bash
geth --dev --http --http.api eth,web3,net --http.corsdomain "https://remix.ethereum.org"
```

2. **打开 Remix IDE**: https://remix.ethereum.org

3. **配置环境**:
   - 在 Remix 中选择 "Deploy & Run Transactions"
   - Environment 选择 "External HTTP Provider"
   - 输入 Geth RPC 地址: `http://localhost:8545`

### 2.2 编写简单合约

在 Remix 中创建 `Storage.sol`:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.8.2 <0.9.0;

/**
 * @title Storage
 * @dev Store & retrieve value in a variable
 */
contract Storage {
    uint256 number;

    /**
     * @dev Store value in variable
     * @param num value to store
     */
    function store(uint256 num) public {
        number = num;
    }

    /**
     * @dev Return value 
     * @return value of 'number'
     */
    function retrieve() public view returns (uint256) {
        return number;
    }
}
```

### 2.3 部署合约

1. **编译合约**: 点击 "Solidity Compiler" -> "Compile Storage.sol"
2. **部署合约**: 
   - 选择 "Deploy & Run Transactions"
   - 点击 "Deploy"
   - 在 Geth 控制台查看部署日志:

```
INFO [05-09|12:27:09.680] Submitted contract creation hash=0xbf2d2d1c... from=0x7Aa16266... nonce=1 contract=0x4aA11DdfD817... value=0
INFO [05-09|12:27:09.681] 🔨 mined potential block number=2 hash=e927bc..f2c8ed
```

3. **交互测试**:
   - 调用 `store(42)`
   - 调用 `retrieve()` 验证结果

---

## 3. 使用 Solidity 编译器 (solc)

### 3.1 安装 solc

```bash
# macOS
brew install solidity

# Ubuntu
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc

# 验证安装
solc --version
```

### 3.2 编译合约

```bash
# 编译生成 ABI
solc --abi Storage.sol -o build

# 编译生成字节码
solc --bin Storage.sol -o build

# 同时生成 ABI 和字节码
solc --abi --bin Storage.sol -o build
```

**生成的文件**:
- `build/Storage.abi`: 合约 ABI (Application Binary Interface)
- `build/Storage.bin`: 合约字节码

---

## 4. 使用 Go 语言与合约交互

### 4.1 安装 abigen 工具

```bash
# 安装 abigen (Go 绑定生成器)
go install github.com/ethereum/go-ethereum/cmd/abigen@latest

# 验证安装
abigen --version
```

### 4.2 生成 Go 绑定

```bash
# 从 ABI 和字节码生成 Go 绑定
abigen --abi build/Storage.abi \
       --bin build/Storage.bin \
       --pkg main \
       --type Storage \
       --out Storage.go
```

**生成的 `Storage.go` 包含**:
- `DeployStorage()`: 部署合约函数
- `Storage` 结构体: 合约实例
- `Store()`, `Retrieve()`: 合约方法绑定

### 4.3 部署合约 (Go 代码)

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
)

func main() {
	// 连接到 Geth 节点
	conn, err := ethclient.Dial("http://localhost:8545")
	if err != nil {
		log.Fatalf("Failed to connect to Geth: %v", err)
	}

	// 加载私钥
	privateKey, err := crypto.HexToECDSA("your_private_key_hex")
	if err != nil {
		log.Fatalf("Failed to load private key: %v", err)
	}

	// 获取链 ID
	chainID, err := conn.ChainID(context.Background())
	if err != nil {
		log.Fatalf("Failed to get chain ID: %v", err)
	}

	// 创建交易发送者
	auth, err := bind.NewKeyedTransactorWithChainID(privateKey, chainID)
	if err != nil {
		log.Fatalf("Failed to create transactor: %v", err)
	}

	// 部署合约
	address, tx, instance, err := DeployStorage(auth, conn)
	if err != nil {
		log.Fatalf("Failed to deploy contract: %v", err)
	}

	fmt.Printf("Contract deployed at: 0x%x\n", address)
	fmt.Printf("Transaction hash: 0x%x\n", tx.Hash())

	// 等待交易确认
	receipt, err := bind.WaitMined(context.Background(), conn, tx)
	if err != nil {
		log.Fatalf("Failed to wait for deployment: %v", err)
	}
	fmt.Printf("Contract deployed in block: %d\n", receipt.BlockNumber)

	// 调用 store 方法
	tx, err = instance.Store(auth, big.NewInt(42))
	if err != nil {
		log.Fatalf("Failed to call store: %v", err)
	}
	fmt.Printf("Store transaction: 0x%x\n", tx.Hash())

	// 等待交易确认
	bind.WaitMined(context.Background(), conn, tx)

	// 调用 retrieve 方法
	value, err := instance.Retrieve(&bind.CallOpts{})
	if err != nil {
		log.Fatalf("Failed to call retrieve: %v", err)
	}
	fmt.Printf("Retrieved value: %d\n", value)
}
```

### 4.4 使用模拟后端测试

```go
package main

import (
	"context"
	"fmt"
	"math/big"

	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient/simulated"
	"github.com/ethereum/go-ethereum/params"
)

func main() {
	// 生成测试密钥
	key, _ := crypto.GenerateKey()
	chainID := params.AllDevChainProtocolChanges.ChainID
	auth, _ := bind.NewKeyedTransactorWithChainID(key, chainID)

	// 创建模拟后端
	sim := simulated.NewBackend(map[common.Address]types.Account{
		auth.From: {Balance: big.NewInt(9e18)},
	})

	// 部署合约
	_, tx, store, err := DeployStorage(auth, sim.Client())
	if err != nil {
		panic(err)
	}
	fmt.Printf("Deploy pending: 0x%x\n", tx.Hash())

	// 挖矿确认
	sim.Commit()

	// 调用 store
	tx, _ = store.Store(auth, big.NewInt(420))
	fmt.Printf("State update pending: 0x%x\n", tx.Hash())
	sim.Commit()

	// 调用 retrieve
	val, _ := store.Retrieve(nil)
	fmt.Printf("Value: %v\n", val)
}
```

---

## 5. 使用 Hardhat 开发框架

### 5.1 安装 Hardhat

```bash
# 创建项目目录
mkdir my-hardhat-project
cd my-hardhat-project

# 初始化 npm 项目
npm init -y

# 安装 Hardhat
npm install --save-dev hardhat

# 初始化 Hardhat 项目
npx hardhat init
```

### 5.2 配置 Hardhat 连接 Geth

编辑 `hardhat.config.js`:

```javascript
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.19",
  networks: {
    gethdev: {
      url: "http://127.0.0.1:8545",
      accounts: ["0x...your_private_key..."]
    }
  }
};
```

### 5.3 编写和部署合约

```javascript
// scripts/deploy.js
const hre = require("hardhat");

async function main() {
  const Storage = await hre.ethers.getContractFactory("Storage");
  const storage = await Storage.deploy();

  await storage.waitForDeployment();

  console.log("Storage deployed to:", await storage.getAddress());

  // 调用合约方法
  const tx = await storage.store(42);
  await tx.wait();

  const value = await storage.retrieve();
  console.log("Retrieved value:", value.toString());
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

**部署命令**:

```bash
npx hardhat run scripts/deploy.js --network gethdev
```

---

## 6. 实践练习

### 练习 1: 部署 ERC20 代币

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("MyToken", "MTK") {
        _mint(msg.sender, initialSupply);
    }
}
```

### 练习 2: 使用 Geth 控制台部署合约

```javascript
// 在 Geth 控制台
var abi = [...];  // 合约 ABI
var bytecode = "0x...";  // 合约字节码

var MyContract = eth.contract(abi);
var myContract = MyContract.new({
  from: eth.accounts[0],
  data: bytecode,
  gas: 3000000
}, function(err, contract) {
  if (!err) {
    if (contract.address) {
      console.log("Contract deployed at:", contract.address);
    }
  }
});
```

### 练习 3: 监听合约事件

```javascript
// Hardhat 脚本
const storage = await ethers.getContractAt("Storage", contractAddress);

storage.on("ValueChanged", (oldValue, newValue, event) => {
  console.log(`Value changed from ${oldValue} to ${newValue}`);
});
```

---

## 7. 总结

本文档介绍了智能合约开发的完整流程:

- **Geth 开发模式**: 快速本地测试环境
- **Remix IDE**: 可视化开发和调试
- **Solidity 编译器**: 生成 ABI 和字节码
- **Go 语言集成**: 使用 abigen 生成绑定
- **Hardhat 框架**: 现代化开发工具链

---

## 8. 参考资料

- [Geth Developer Mode](https://geth.ethereum.org/docs/developers/dapp-developer/dev-mode)
- [Remix IDE](https://remix.ethereum.org/)
- [Solidity Documentation](https://docs.soliditylang.org/)
- [Hardhat Documentation](https://hardhat.org/docs)
- [Go-Ethereum Native Bindings](https://geth.ethereum.org/docs/developers/dapp-developer/native-bindings)

---

**下一步**: 学习 [07-JSON-RPC-API.md](./07-JSON-RPC-API.md) - JSON-RPC API

