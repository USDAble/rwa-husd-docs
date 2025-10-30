# 部署指南

**文档版本**: v1.0  
**创建时间**: 2025-10-18 14:52:14 CST  
**适用范围**: ABLE 代币 Uniswap V2 集成部署流程

---

## 1. 部署前准备

### 1.1 环境要求

**开发工具**:
- Node.js >= 18.0.0
- Hardhat >= 2.19.0
- Ethers.js >= 6.0.0

**网络配置**:
```javascript
// hardhat.config.js
module.exports = {
    networks: {
        opSepolia: {
            url: "https://sepolia.optimism.io",
            accounts: [PRIVATE_KEY],
            chainId: 11155420
        },
        opMainnet: {
            url: "https://mainnet.optimism.io",
            accounts: [PRIVATE_KEY],
            chainId: 10
        }
    },
    etherscan: {
        apiKey: {
            opSepolia: OP_ETHERSCAN_API_KEY,
            opMainnet: OP_ETHERSCAN_API_KEY
        }
    }
};
```

### 1.2 资金准备

**测试网 (OP Sepolia)**:
- ETH (用于 Gas): 0.1 ETH (从水龙头获取)
- 测试 ABLE: 10,000,000 ABLE
- 测试 USDC: 1,000,000 USDC

**主网 (OP Mainnet)**:
- ETH (用于 Gas): 0.5 ETH
- ABLE: 5,000,000 ABLE (初始流动性 + 储备)
- USDC: 500,000 USDC
- ABLE 奖励: 100,000,000 ABLE (流动性挖矿)

### 1.3 合约地址确认

**OP Mainnet**:
```javascript
const ADDRESSES = {
    UNISWAP_V2_FACTORY: "0x0c3c1c532F1e39EdF36BE9Fe0bE1410313E074Bf",
    UNISWAP_V2_ROUTER: "0x4A7b5Da61326A6379179b40d00F57E5bbDC962c2",
    USDC: "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85",
    WETH: "0x4200000000000000000000000000000000000006",
    ABLE: "0x...",  // 待部署
    TREASURY: "0x..."  // 现有合约
};
```

---

## 2. 测试网部署流程

### 2.1 部署 ABLE 代币 (如果未部署)

```javascript
// scripts/01-deploy-able-token.js
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();
    console.log("Deploying ABLE token with:", deployer.address);
    
    const AbleToken = await ethers.getContractFactory("AbleToken");
    const ableToken = await AbleToken.deploy(
        "ABLE Token",
        "ABLE",
        ethers.parseEther("1000000000")  // 1B supply
    );
    
    await ableToken.waitForDeployment();
    const address = await ableToken.getAddress();
    
    console.log("ABLE Token deployed to:", address);
    
    // 验证合约
    await run("verify:verify", {
        address: address,
        constructorArguments: [
            "ABLE Token",
            "ABLE",
            ethers.parseEther("1000000000")
        ]
    });
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

运行:
```bash
npx hardhat run scripts/01-deploy-able-token.js --network opSepolia
```

### 2.2 部署 UniswapV2LiquidityManager

```javascript
// scripts/02-deploy-liquidity-manager.js
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();
    
    const ROUTER = "0x4A7b5Da61326A6379179b40d00F57E5bbDC962c2";
    const FACTORY = "0x0c3c1c532F1e39EdF36BE9Fe0bE1410313E074Bf";
    const ABLE = "0x...";  // 从步骤 2.1 获取
    const TREASURY = "0x...";  // 现有 Treasury 地址
    
    const LiquidityManager = await ethers.getContractFactory("UniswapV2LiquidityManager");
    const manager = await LiquidityManager.deploy(
        ROUTER,
        FACTORY,
        ABLE,
        TREASURY,
        deployer.address  // admin
    );
    
    await manager.waitForDeployment();
    const address = await manager.getAddress();
    
    console.log("LiquidityManager deployed to:", address);
    
    // 验证合约
    await run("verify:verify", {
        address: address,
        constructorArguments: [ROUTER, FACTORY, ABLE, TREASURY, deployer.address]
    });
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

运行:
```bash
npx hardhat run scripts/02-deploy-liquidity-manager.js --network opSepolia
```

### 2.3 部署 AbleLiquidityMining

```javascript
// scripts/03-deploy-liquidity-mining.js
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();
    
    const ABLE = "0x...";
    const LP_TOKEN = "0x...";  // 创建池后获取
    const INITIAL_REWARD_RATE = ethers.parseEther("100");  // 100 ABLE/秒
    
    const LiquidityMining = await ethers.getContractFactory("AbleLiquidityMining");
    const mining = await LiquidityMining.deploy(
        ABLE,
        LP_TOKEN,
        INITIAL_REWARD_RATE,
        deployer.address
    );
    
    await mining.waitForDeployment();
    const address = await mining.getAddress();
    
    console.log("LiquidityMining deployed to:", address);
    
    // 验证合约
    await run("verify:verify", {
        address: address,
        constructorArguments: [ABLE, LP_TOKEN, INITIAL_REWARD_RATE, deployer.address]
    });
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

### 2.4 创建流动性池

```javascript
// scripts/04-create-pool.js
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();
    
    const MANAGER_ADDRESS = "0x...";
    const ABLE_ADDRESS = "0x...";
    const USDC_ADDRESS = "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85";
    
    const manager = await ethers.getContractAt("UniswapV2LiquidityManager", MANAGER_ADDRESS);
    const ableToken = await ethers.getContractAt("IERC20", ABLE_ADDRESS);
    const usdcToken = await ethers.getContractAt("IERC20", USDC_ADDRESS);
    
    // 1. 准备资金
    const ableAmount = ethers.parseEther("10000");  // 10K ABLE (测试)
    const usdcAmount = ethers.parseUnits("1000", 6);  // 1K USDC (测试)
    
    // 2. 授权 Manager
    await ableToken.approve(MANAGER_ADDRESS, ableAmount);
    await usdcToken.approve(MANAGER_ADDRESS, usdcAmount);
    
    // 3. 创建池
    console.log("Creating pool...");
    const tx = await manager.createPoolAndAddLiquidity(
        USDC_ADDRESS,
        ableAmount,
        usdcAmount
    );
    
    const receipt = await tx.wait();
    console.log("Pool created! Tx:", receipt.hash);
    
    // 4. 获取池地址
    const factory = await ethers.getContractAt(
        "IUniswapV2Factory",
        "0x0c3c1c532F1e39EdF36BE9Fe0bE1410313E074Bf"
    );
    const poolAddress = await factory.getPair(ABLE_ADDRESS, USDC_ADDRESS);
    console.log("Pool address:", poolAddress);
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

### 2.5 测试验证

```javascript
// scripts/05-test-pool.js
const { ethers } = require("hardhat");

async function main() {
    const POOL_ADDRESS = "0x...";
    const ABLE_ADDRESS = "0x...";
    const USDC_ADDRESS = "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85";
    
    const pool = await ethers.getContractAt("IUniswapV2Pair", POOL_ADDRESS);
    const ableToken = await ethers.getContractAt("IERC20", ABLE_ADDRESS);
    const usdcToken = await ethers.getContractAt("IERC20", USDC_ADDRESS);
    
    // 1. 检查储备量
    const [reserve0, reserve1] = await pool.getReserves();
    console.log("Reserve0:", ethers.formatEther(reserve0));
    console.log("Reserve1:", ethers.formatUnits(reserve1, 6));
    
    // 2. 检查价格
    const token0 = await pool.token0();
    const price = token0 === ABLE_ADDRESS
        ? Number(reserve1) / Number(reserve0)
        : Number(reserve0) / Number(reserve1);
    console.log("ABLE Price:", price, "USDC");
    
    // 3. 测试交易
    const router = await ethers.getContractAt(
        "IUniswapV2Router02",
        "0x4A7b5Da61326A6379179b40d00F57E5bbDC962c2"
    );
    
    const amountIn = ethers.parseUnits("100", 6);  // 100 USDC
    await usdcToken.approve(router.address, amountIn);
    
    const path = [USDC_ADDRESS, ABLE_ADDRESS];
    const amounts = await router.getAmountsOut(amountIn, path);
    console.log("Expected ABLE output:", ethers.formatEther(amounts[1]));
    
    const tx = await router.swapExactTokensForTokens(
        amountIn,
        0,
        path,
        deployer.address,
        Math.floor(Date.now() / 1000) + 300
    );
    
    await tx.wait();
    console.log("Swap successful!");
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

---

## 3. 主网部署流程

### 3.1 部署前最终检查

- [ ] 所有测试网测试通过
- [ ] 代码审计完成
- [ ] 所有审计问题已修复
- [ ] 部署脚本已验证
- [ ] 多签钱包已配置 (如果使用)
- [ ] 应急预案已准备
- [ ] 团队成员已就位

### 3.2 主网部署步骤

**步骤 1: 部署合约** (与测试网相同,但使用 `--network opMainnet`)

```bash
# 1. 部署 ABLE Token (如果未部署)
npx hardhat run scripts/01-deploy-able-token.js --network opMainnet

# 2. 部署 LiquidityManager
npx hardhat run scripts/02-deploy-liquidity-manager.js --network opMainnet

# 3. 创建池
npx hardhat run scripts/04-create-pool.js --network opMainnet

# 4. 部署 LiquidityMining
npx hardhat run scripts/03-deploy-liquidity-mining.js --network opMainnet
```

**步骤 2: 配置权限**

```javascript
// scripts/06-configure-permissions.js
const { ethers } = require("hardhat");

async function main() {
    const MANAGER_ADDRESS = "0x...";
    const MINING_ADDRESS = "0x...";
    const OPERATOR_ADDRESS = "0x...";  // 运营地址
    const EMERGENCY_ADDRESS = "0x...";  // 紧急响应地址
    
    const manager = await ethers.getContractAt("UniswapV2LiquidityManager", MANAGER_ADDRESS);
    
    // 授予 OPERATOR 角色
    const OPERATOR_ROLE = ethers.keccak256(ethers.toUtf8Bytes("OPERATOR_ROLE"));
    await manager.grantRole(OPERATOR_ROLE, OPERATOR_ADDRESS);
    
    // 授予 EMERGENCY 角色
    const EMERGENCY_ROLE = ethers.keccak256(ethers.toUtf8Bytes("EMERGENCY_ROLE"));
    await manager.grantRole(EMERGENCY_ROLE, EMERGENCY_ADDRESS);
    
    console.log("Permissions configured!");
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

**步骤 3: 添加初始流动性**

```javascript
// scripts/07-add-initial-liquidity.js
const { ethers } = require("hardhat");

async function main() {
    const MANAGER_ADDRESS = "0x...";
    const ABLE_ADDRESS = "0x...";
    const USDC_ADDRESS = "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85";
    
    const manager = await ethers.getContractAt("UniswapV2LiquidityManager", MANAGER_ADDRESS);
    
    // 主池: ABLE/USDC
    const ableAmount = ethers.parseEther("2500000");  // 2.5M ABLE
    const usdcAmount = ethers.parseUnits("250000", 6);  // 250K USDC
    
    console.log("Adding liquidity to ABLE/USDC pool...");
    const tx = await manager.createPoolAndAddLiquidity(
        USDC_ADDRESS,
        ableAmount,
        usdcAmount
    );
    
    const receipt = await tx.wait();
    console.log("Liquidity added! Tx:", receipt.hash);
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

**步骤 4: 启动流动性挖矿**

```javascript
// scripts/08-start-mining.js
const { ethers } = require("hardhat");

async function main() {
    const MINING_ADDRESS = "0x...";
    const ABLE_ADDRESS = "0x...";
    
    const mining = await ethers.getContractAt("AbleLiquidityMining", MINING_ADDRESS);
    const ableToken = await ethers.getContractAt("IERC20", ABLE_ADDRESS);
    
    // 转入奖励代币 (100M ABLE)
    const rewardAmount = ethers.parseEther("100000000");
    await ableToken.transfer(MINING_ADDRESS, rewardAmount);
    
    console.log("Mining rewards transferred!");
    console.log("Liquidity mining is now active!");
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

### 3.3 部署后验证

```javascript
// scripts/09-verify-deployment.js
const { ethers } = require("hardhat");

async function main() {
    const POOL_ADDRESS = "0x...";
    const MANAGER_ADDRESS = "0x...";
    const MINING_ADDRESS = "0x...";
    
    // 1. 验证池
    const pool = await ethers.getContractAt("IUniswapV2Pair", POOL_ADDRESS);
    const [reserve0, reserve1] = await pool.getReserves();
    console.log("✓ Pool reserves:", ethers.formatEther(reserve0), ethers.formatUnits(reserve1, 6));
    
    // 2. 验证 Manager
    const manager = await ethers.getContractAt("UniswapV2LiquidityManager", MANAGER_ADDRESS);
    const lpBalance = await manager.getLPBalance(POOL_ADDRESS);
    console.log("✓ Manager LP balance:", ethers.formatEther(lpBalance));
    
    // 3. 验证 Mining
    const mining = await ethers.getContractAt("AbleLiquidityMining", MINING_ADDRESS);
    const rewardRate = await mining.rewardRate();
    console.log("✓ Mining reward rate:", ethers.formatEther(rewardRate), "ABLE/second");
    
    // 4. 验证权限
    const OPERATOR_ROLE = ethers.keccak256(ethers.toUtf8Bytes("OPERATOR_ROLE"));
    const hasRole = await manager.hasRole(OPERATOR_ROLE, OPERATOR_ADDRESS);
    console.log("✓ Operator role granted:", hasRole);
    
    console.log("\n✅ All verifications passed!");
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

---

## 4. 部署后配置

### 4.1 Etherscan 验证

```bash
# 验证所有合约
npx hardhat verify --network opMainnet ABLE_ADDRESS "ABLE Token" "ABLE" "1000000000000000000000000000"
npx hardhat verify --network opMainnet MANAGER_ADDRESS ROUTER FACTORY ABLE TREASURY ADMIN
npx hardhat verify --network opMainnet MINING_ADDRESS ABLE LP_TOKEN REWARD_RATE ADMIN
```

### 4.2 前端集成

```javascript
// frontend/src/config/contracts.js
export const CONTRACTS = {
    ABLE: "0x...",
    USDC: "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85",
    POOL: "0x...",
    LIQUIDITY_MANAGER: "0x...",
    LIQUIDITY_MINING: "0x...",
    UNISWAP_ROUTER: "0x4A7b5Da61326A6379179b40d00F57E5bbDC962c2"
};
```

### 4.3 监控配置

```javascript
// monitoring/config.js
module.exports = {
    rpcUrl: "https://mainnet.optimism.io",
    contracts: {
        pool: "0x...",
        manager: "0x...",
        mining: "0x..."
    },
    alerts: {
        lowLiquidity: ethers.parseEther("100000"),
        priceDeviation: 0.1,  // 10%
        largeSwap: ethers.parseEther("10000")
    },
    webhooks: {
        telegram: "https://...",
        discord: "https://..."
    }
};
```

---

**最后更新**: 2025-10-18 14:52:14 CST

**重要提示**:
1. 主网部署前务必在测试网完整测试
2. 使用多签钱包管理关键权限
3. 分阶段添加流动性,先小额测试
4. 部署后立即配置监控系统

