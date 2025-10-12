# RWA-HUSD 测试环境合约地址

## 📋 网络信息

- **网络名称**: Optimism Mainnet (用作测试环境)
- **Chain ID**: 10
- **RPC URL**: https://mainnet.optimism.io
- **区块链浏览器**: https://optimistic.etherscan.io
- **环境状态**: 🧪 测试环境
- **分支**: test-galahad
- **文档创建时间**: 2025-10-12 09:46:00 CST

---

## 📊 核心合约地址（测试版）

### 1. SystemConfig (系统配置 - 测试版)

- **合约地址**: `0xadA073d6C7E0F7d4Dc518edbfe01DBD4f65ea91E`
- **版本**: V7 UUPS升级支持
- **部署时间**: 2025-08-20
- **架构**: UUPS代理模式
- **验证状态**: ✅ 已验证
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0xada073d6c7e0f7d4dc518edbfe01dbd4f65ea91e)
- **来源repo**: rwa-husd-contracts

---

### 2. UserRegistry (用户注册 - 测试版)

- **合约地址**: `0x35957503d5aDc45bd255897925042831dEa05615`
- **版本**: V7 UUPS升级支持
- **部署时间**: 2025-08-20
- **架构**: UUPS代理模式
- **验证状态**: ✅ 已验证
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x35957503d5adc45bd255897925042831dea05615)
- **来源repo**: rwa-husd-contracts

---

### 3. RedemptionManager (赎回管理 - 测试版)

- **代理合约**: `0x0b73cC161afA61d1aBD2c6e0fB1668Ea3e69dBA0`
- **实现合约**: `0x20aD9DdC6b3Ca02B75C0F50542B8A413897B0302`
- **版本**: V6.2 (固定总额快照方案)
- **部署时间**: 2025-08-20
- **架构**: UUPS代理模式
- **验证状态**: ✅ 已验证
- **区块链浏览器**: 
  - [代理合约](https://optimistic.etherscan.io/address/0x0b73cC161afA61d1aBD2c6e0fB1668Ea3e69dBA0)
  - [实现合约](https://optimistic.etherscan.io/address/0x20aD9DdC6b3Ca02B75C0F50542B8A413897B0302)
- **来源repo**: redemption-contracts
- **外部合约配置**:
  - SystemConfig (TEST): `0xadA073d6C7E0F7d4Dc518edbfe01DBD4f65ea91E`
  - UserRegistry (TEST): `0x35957503d5aDc45bd255897925042831dEa05615`
  - Payment Token: `0x6e6Ed6c3B17EbbBb2b1E330C4C2e0974c4782591`

---

### 4. RentCustodyContract (房租托管 - 测试版)

- **代理合约**: `0xf64A952e81bbBAa6C8c134e4ceb943dcB59F4772`
- **实现合约**: `0xc8916A1c6f7c29eaDCE1ff2631c7fA060438dF0f`
- **版本**: v2.2.0
- **部署时间**: 2025-08-28
- **架构**: UUPS代理模式
- **验证状态**: ✅ 已验证
- **区块链浏览器**: 
  - [代理合约](https://optimistic.etherscan.io/address/0xf64a952e81bbbaa6c8c134e4ceb943dcb59f4772)
  - [实现合约](https://optimistic.etherscan.io/address/0xc8916A1c6f7c29eaDCE1ff2631c7fA060438dF0f)
- **来源repo**: batch-token-transfer
- **默认支付代币**: `0x6e6Ed6c3B17EbbBb2b1E330C4C2e0974c4782591` (测试USDC)

---

### 5. TradeContract (二级市场交易 - 测试版)

- **代理合约**: `0xeb2A3CA953FFdde684187fA800552F9b5E2798C7`
- **实现合约**: `0x330125bEEe51DE0945c7052A5a9604b78Ea50f3b`
- **版本**: v1.1.0-security-enhanced
- **架构**: UUPS代理模式
- **验证状态**: ✅ 已验证
- **区块链浏览器**: 
  - [代理合约](https://optimistic.etherscan.io/address/0xeb2a3ca953ffdde684187fa800552f9b5e2798c7)
  - [实现合约](https://optimistic.etherscan.io/address/0x330125bEEe51DE0945c7052A5a9604b78Ea50f3b)
- **来源repo**: rwa-husd-trading
- **用途**: 安全测试
- **安全级别**: 🛡️ 增强

---

### 6. AbleVestingFactory (代币释放工厂 - 测试版)

- **合约地址**: `0x7d902625397446FA4331a03Fc7275bAbf86637fc`
- **部署时间**: 2025-09-23 08:18:35 (UTC+8)
- **验证状态**: ✅ 已验证
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x7d902625397446fa4331a03fc7275babf86637fc)
- **来源repo**: rwa-real-estate-community-coin-contract

---

## 📊 共享生产环境合约

以下合约在测试环境中共享生产环境的合约地址：

### 1. PropertyTokenFactory (房产代币工厂)

- **代理合约**: `0x23FB5125ec344b3611018222C7b1572E05b8D8e3`
- **实现合约**: `0x8F29c72113Ab608953c44d5Fcd6740fc53962E71`
- **说明**: 测试环境共享生产环境合约
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x23fb5125ec344b3611018222c7b1572e05b8d8e3)

---

### 2. PropertyToken (房产代币实现)

- **实现合约**: `0xcbc23Bc33983ea0e5eD1e40D60993a71dCFC3790`
- **说明**: 测试环境共享生产环境合约
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0xcbc23bc33983ea0e5ed1e40d60993a71dcfc3790)

---

### 3. Treasury (金库)

- **代理合约**: `0x0793E2eCb0d4ccBCA8483890b29bcb1Af7dA7e2a`
- **实现合约**: `0x6d75C828898dEAc596eBdc01A6F78f1d419ed5ad`
- **说明**: 测试环境共享生产环境合约
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x0793e2ecb0d4ccbca8483890b29bcb1af7da7e2a)

---

### 4. RedemptionStrategy (赎回策略)

- **代理合约**: `0x4D9EaFF6C3Dc50c493CA21b16383567D4B3e7FB8`
- **实现合约**: `0x9688d01c0cfAFd913746645411217B9032a215EF`
- **说明**: 测试环境共享生产环境合约
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x4d9eaff6c3dc50c493ca21b16383567d4b3e7fb8)

---

### 5. AbleToken (平台代币)

- **合约地址**: `0xe69459d921C64B8dB54882100e626eed4773AaFF`
- **说明**: 测试环境共享生产环境合约
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0xe69459d921c64b8db54882100e626eed4773aaff)

---

### 6. AbleVesting (代币释放)

- **合约地址**: `0x3Cb1141f3E2833df813bCCd96ccd23b4239140e4`
- **说明**: 测试环境共享生产环境合约
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x3cb1141f3e2833df813bccd96ccd23b4239140e4)

---

## 📊 代币合约地址

### 1. MockUSDC (测试代币)

- **合约地址**: `0x6e6Ed6c3B17EbbBb2b1E330C4C2e0974c4782591`
- **代币类型**: ERC20 (Mock)
- **代币名称**: USDC
- **代币精度**: 6位小数
- **说明**: 与生产环境共享相同的MockUSDC合约
- **用途**: 测试代币，避免真实资金风险
- **验证状态**: ✅ 已部署
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x6e6ed6c3b17ebbbb2b1e330c4c2e0974c4782591)

---

### 2. MockUSDT (测试代币)

- **合约地址**: `0x8470c4aC059E21cd409701338560BA993D27CFE7`
- **代币类型**: ERC20 (Mock)
- **代币名称**: USDT
- **代币精度**: 6位小数
- **状态**: ✅ 已添加到白名单，用于测试
- **用途**: 测试代币，避免真实资金风险
- **验证状态**: ✅ 已部署
- **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x8470c4ac059e21cd409701338560ba993d27cfe7)
- **来源repo**: shop

---

## ⏳ 待部署合约

### 1. PaymentContract (支付合约 - 测试版)

- **状态**: ⏳ 待部署
- **说明**: 测试环境尚未部署PaymentContract的测试版本

---

### 2. StakingAuthorization (质押授权)

- **状态**: ⏳ 待部署
- **说明**: 在RWA-HUSD-Architecture.md中有提及，但尚未找到已部署的合约地址

---

## ⚠️ 测试环境注意事项

### 数据隔离
- 🔄 **环境隔离**: 测试环境与生产环境数据完全隔离
- 🔑 **独立配置**: 使用独立的SystemConfig和UserRegistry
- 📊 **独立数据**: 测试数据不会影响生产环境

### 测试代币
- 💧 **无实际价值**: 使用的是测试代币（MockUSDC, MockUSDT），无实际价值
- 🎁 **免费获取**: 可以通过测试合约免费获取测试代币
- ⚠️ **仅供测试**: 测试代币仅用于功能测试和开发验证

### 快速迭代
- 🚀 **快速部署**: 可以快速部署和测试新功能
- 🔧 **灵活配置**: 可以灵活调整合约参数
- 📊 **性能测试**: 可以进行压力测试和性能评估

### 共享合约
- 🔗 **部分共享**: PropertyTokenFactory, Treasury等合约共享生产环境
- ⚠️ **谨慎操作**: 操作共享合约时需要特别小心，避免影响生产环境
- 📝 **建议**: 优先使用独立的测试版合约进行测试

---

## 📖 使用说明

### 连接到测试合约

```javascript
// 导入ethers.js
const { ethers } = require("ethers");

// 连接到Optimism Mainnet (测试环境)
const provider = new ethers.providers.JsonRpcProvider("https://mainnet.optimism.io");

// 连接到TradeContract (测试版)
const tradeAddress = "0xeb2A3CA953FFdde684187fA800552F9b5E2798C7";
const tradeABI = [...]; // 从Contract-ABIs.md获取
const trade = new ethers.Contract(tradeAddress, tradeABI, provider);

// 查询合约信息
const systemConfig = await trade.systemConfig();
console.log("SystemConfig地址:", systemConfig);
```

---

### 查询合约信息

```javascript
// 查询RedemptionManager (测试版) 信息
const redemptionAddress = "0x0b73cC161afA61d1aBD2c6e0fB1668Ea3e69dBA0";
const redemptionABI = [...]; // 从Contract-ABIs.md获取
const redemption = new ethers.Contract(redemptionAddress, redemptionABI, provider);

// 获取快照信息
const propertyTokenAddress = "0x987FB05ac801103DBB05ba74f9C9443967322Fbb";
const totalRedeemable = await redemption.getTotalRedeemableAmount(propertyTokenAddress);
const isSet = await redemption.isRedeemableAmountSet(propertyTokenAddress);

console.log("总可赎回金额:", totalRedeemable.toString());
console.log("快照是否已设置:", isSet);
```

---

## 📝 更新日志

- **2025-10-12**: 创建测试环境合约地址文档
- **2025-09-23**: AbleVestingFactory部署
- **2025-08-28**: RentCustodyContract v2.2.0部署
- **2025-08-20**: SystemConfig和UserRegistry V7部署，RedemptionManager V6.2部署

