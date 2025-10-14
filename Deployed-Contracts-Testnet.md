# RWA-HUSD 测试环境合约地址

## 📋 网络信息

-   **网络名称**: Optimism Mainnet (用作测试环境)
-   **Chain ID**: 10
-   **RPC URL**: https://mainnet.optimism.io
-   **区块链浏览器**: https://optimistic.etherscan.io
-   **环境状态**: 🧪 测试环境
-   **分支**: test-galahad
-   **文档创建时间**: 2025-10-12 09:46:00 CST

---

## 📊 核心合约地址（测试版）

### 1. SystemConfig (系统配置 - 测试版)

-   **代理合约**: `0x4b97b05797aAd140bA3cc44a9e009262c80D2035`
-   **实现合约**: `0x7D2b79f45F6c5B81a378037Cafa6fCB1d5E174c3`
-   **版本**: V9 UUPS 升级支持
-   **部署时间**: 2025-10-14 10:43 CST
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0x4b97b05797aad140ba3cc44a9e009262c80d2035)
    -   [实现合约](https://optimistic.etherscan.io/address/0x7d2b79f45f6c5b81a378037cafa6fcb1d5e174c3)
-   **来源 repo**: rwa-husd-contracts
-   **部署区块**: 142,405,725

---

### 2. UserRegistry (用户注册 - 测试版)

-   **代理合约**: `0xA4eB54d8160b43732bDCa1559Daf53640B960f49`
-   **实现合约**: `0xEA1007F10D1A38D2Cd55fCFE067224E4F4ca9367`
-   **版本**: V9 UUPS 升级支持
-   **部署时间**: 2025-10-14 10:43 CST
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0xa4eb54d8160b43732bdca1559daf53640b960f49)
    -   [实现合约](https://optimistic.etherscan.io/address/0xea1007f10d1a38d2cd55fcfe067224e4f4ca9367)
-   **来源 repo**: rwa-husd-contracts
-   **部署区块**: 142,405,725

---

### 3. RedemptionManager (赎回管理 - 测试版)

-   **代理合约**: `0x34d4285399AC5aCF90BEDA0a409257b7CeeCE14E`
-   **实现合约**: `0xd01b05a2297D0FF931E36a57ed8ad52A19441389`
-   **版本**: V9 UUPS 升级支持
-   **部署时间**: 2025-10-14 11:00 CST
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0x34d4285399ac5acf90beda0a409257b7ceece14e)
    -   [实现合约](https://optimistic.etherscan.io/address/0xd01b05a2297d0ff931e36a57ed8ad52a19441389)
-   **来源 repo**: redemption-contracts
-   **部署区块**: 142,406,208
-   **外部合约配置**:
    -   SystemConfig (V9): `0x4b97b05797aAd140bA3cc44a9e009262c80D2035`
    -   UserRegistry (V9): `0xA4eB54d8160b43732bDCa1559Daf53640B960f49`
    -   Payment Token: `0x6e6Ed6c3B17EbbBb2b1E330C4C2e0974c4782591`

---

### 4. Treasury (资金库 - 测试版)

-   **代理合约**: `0xC70d4aCE7c34068d4A4E86c321506CC52a6AF2eD`
-   **实现合约**: `0xd8c9F76fE4e09Ceb1036Aa874bda9180A7cf8a79`
-   **版本**: V9 UUPS 升级支持
-   **部署时间**: 2025-10-14 11:05 CST
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0xc70d4ace7c34068d4a4e86c321506cc52a6af2ed)
    -   [实现合约](https://optimistic.etherscan.io/address/0xd8c9f76fe4e09ceb1036aa874bda9180a7cf8a79)
-   **来源 repo**: treasury-contracts
-   **部署区块**: 142,406,273
-   **外部合约配置**:
    -   SystemConfig (V9): `0x4b97b05797aAd140bA3cc44a9e009262c80D2035`

---

### 5. RedemptionStrategy (赎回策略 - 测试版)

-   **代理合约**: `0x8D5743b821cD07F79Af1C5e543Fe62A80EDddC70`
-   **实现合约**: `0x178E844b5186E54Ff289D6E6FdAF0CE7c6Ff14db`
-   **版本**: V9 UUPS 升级支持
-   **部署时间**: 2025-10-14 11:05 CST
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0x8d5743b821cd07f79af1c5e543fe62a80edddc70)
    -   [实现合约](https://optimistic.etherscan.io/address/0x178e844b5186e54ff289d6e6fdaf0ce7c6ff14db)
-   **来源 repo**: treasury-contracts
-   **部署区块**: 142,406,273
-   **外部合约配置**:
    -   RedemptionManager (V9): `0x34d4285399AC5aCF90BEDA0a409257b7CeeCE14E`
    -   Treasury (V9): `0xC70d4aCE7c34068d4A4E86c321506CC52a6AF2eD`

---

### 6. TradeContract (交易合约 - 测试版)

-   **代理合约**: `0xCCb118EaF2dDc0651De036924eE644c805530164`
-   **实现合约**: `0x1dB75Ba341fF674F7982e7219cc5515683473508`
-   **版本**: V9 UUPS 升级支持
-   **部署时间**: 2025-10-14 11:10 CST
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0xccb118eaf2ddc0651de036924ee644c805530164)
    -   [实现合约](https://optimistic.etherscan.io/address/0x1db75ba341ff674f7982e7219cc5515683473508)
-   **来源 repo**: rwa-husd-trading
-   **部署区块**: 142,406,503
-   **外部合约配置**:
    -   SystemConfig (V9): `0x4b97b05797aAd140bA3cc44a9e009262c80D2035`
    -   UserRegistry (V9): `0xA4eB54d8160b43732bDCa1559Daf53640B960f49`
    -   Treasury (V9): `0xC70d4aCE7c34068d4A4E86c321506CC52a6AF2eD`

---

### 7. PropertyTokenFactory (房产代币工厂 - 测试版)

-   **代理合约**: `0x542ca6448b5a1207b6aA5BAE8D24483F8e414580`
-   **实现合约**: `0x3c50f98974a438E93C20DFda4eF22faBa921b696`
-   **版本**: V9 UUPS 升级支持
-   **部署时间**: 2025-10-14 11:20 CST
-   **架构**: UUPS 代理模式
-   **验证状态**: ⏳ 待验证
-   **来源 repo**: rwa-husd-contracts
-   **外部合约配置**:
    -   SystemConfig (V9): `0x4b97b05797aAd140bA3cc44a9e009262c80D2035`
    -   UserRegistry (V9): `0xA4eB54d8160b43732bDCa1559Daf53640B960f49`
    -   Treasury (V9): `0xC70d4aCE7c34068d4A4E86c321506CC52a6AF2eD`
    -   RedemptionManager (V9): `0x34d4285399AC5aCF90BEDA0a409257b7CeeCE14E`
    -   PropertyToken Implementation: `0x933270F0a369DBa05bB847f262e9A43f717D5d33`

---

### 8. RentCustodyContract (房租托管 - 测试版)

-   **代理合约**: `0xf64A952e81bbBAa6C8c134e4ceb943dcB59F4772`
-   **实现合约**: `0xc8916A1c6f7c29eaDCE1ff2631c7fA060438dF0f`
-   **版本**: v2.2.0
-   **部署时间**: 2025-08-28
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0xf64a952e81bbbaa6c8c134e4ceb943dcb59f4772)
    -   [实现合约](https://optimistic.etherscan.io/address/0xc8916A1c6f7c29eaDCE1ff2631c7fA060438dF0f)
-   **来源 repo**: batch-token-transfer
-   **默认支付代币**: `0x6e6Ed6c3B17EbbBb2b1E330C4C2e0974c4782591` (测试 USDC)

---

### 5. TradeContract (二级市场交易 - 测试版)

-   **代理合约**: `0xeb2A3CA953FFdde684187fA800552F9b5E2798C7`
-   **实现合约**: `0x330125bEEe51DE0945c7052A5a9604b78Ea50f3b`
-   **版本**: v1.1.0-security-enhanced
-   **架构**: UUPS 代理模式
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**:
    -   [代理合约](https://optimistic.etherscan.io/address/0xeb2a3ca953ffdde684187fa800552f9b5e2798c7)
    -   [实现合约](https://optimistic.etherscan.io/address/0x330125bEEe51DE0945c7052A5a9604b78Ea50f3b)
-   **来源 repo**: rwa-husd-trading
-   **用途**: 安全测试
-   **安全级别**: 🛡️ 增强

---

### 6. AbleVestingFactory (代币释放工厂 - 测试版)

-   **合约地址**: `0x7d902625397446FA4331a03Fc7275bAbf86637fc`
-   **部署时间**: 2025-09-23 08:18:35 (UTC+8)
-   **验证状态**: ✅ 已验证
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x7d902625397446fa4331a03fc7275babf86637fc)
-   **来源 repo**: rwa-real-estate-community-coin-contract

---

## 📊 共享生产环境合约

以下合约在测试环境中共享生产环境的合约地址：

### 1. PropertyTokenFactory (房产代币工厂)

-   **代理合约**: `0x23FB5125ec344b3611018222C7b1572E05b8D8e3`
-   **实现合约**: `0x8F29c72113Ab608953c44d5Fcd6740fc53962E71`
-   **说明**: 测试环境共享生产环境合约
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x23fb5125ec344b3611018222c7b1572e05b8d8e3)

---

### 2. PropertyToken (房产代币实现)

-   **实现合约**: `0xcbc23Bc33983ea0e5eD1e40D60993a71dCFC3790`
-   **说明**: 测试环境共享生产环境合约
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0xcbc23bc33983ea0e5ed1e40d60993a71dcfc3790)

---

### 3. Treasury (金库)

-   **代理合约**: `0x0793E2eCb0d4ccBCA8483890b29bcb1Af7dA7e2a`
-   **实现合约**: `0x6d75C828898dEAc596eBdc01A6F78f1d419ed5ad`
-   **说明**: 测试环境共享生产环境合约
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x0793e2ecb0d4ccbca8483890b29bcb1af7da7e2a)

---

### 4. RedemptionStrategy (赎回策略)

-   **代理合约**: `0x4D9EaFF6C3Dc50c493CA21b16383567D4B3e7FB8`
-   **实现合约**: `0x9688d01c0cfAFd913746645411217B9032a215EF`
-   **说明**: 测试环境共享生产环境合约
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x4d9eaff6c3dc50c493ca21b16383567d4b3e7fb8)

---

### 5. AbleToken (平台代币)

-   **合约地址**: `0xe69459d921C64B8dB54882100e626eed4773AaFF`
-   **说明**: 测试环境共享生产环境合约
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0xe69459d921c64b8db54882100e626eed4773aaff)

---

### 6. AbleVesting (代币释放)

-   **合约地址**: `0x3Cb1141f3E2833df813bCCd96ccd23b4239140e4`
-   **说明**: 测试环境共享生产环境合约
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x3cb1141f3e2833df813bccd96ccd23b4239140e4)

---

## 📊 代币合约地址

### 1. MockUSDC (测试代币)

-   **合约地址**: `0x6e6Ed6c3B17EbbBb2b1E330C4C2e0974c4782591`
-   **代币类型**: ERC20 (Mock)
-   **代币名称**: USDC
-   **代币精度**: 6 位小数
-   **说明**: 与生产环境共享相同的 MockUSDC 合约
-   **用途**: 测试代币，避免真实资金风险
-   **验证状态**: ✅ 已部署
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x6e6ed6c3b17ebbbb2b1e330c4c2e0974c4782591)

---

### 2. MockUSDT (测试代币)

-   **合约地址**: `0x8470c4aC059E21cd409701338560BA993D27CFE7`
-   **代币类型**: ERC20 (Mock)
-   **代币名称**: USDT
-   **代币精度**: 6 位小数
-   **状态**: ✅ 已添加到白名单，用于测试
-   **用途**: 测试代币，避免真实资金风险
-   **验证状态**: ✅ 已部署
-   **区块链浏览器**: [查看合约](https://optimistic.etherscan.io/address/0x8470c4ac059e21cd409701338560ba993d27cfe7)
-   **来源 repo**: shop

---

## ⏳ 待部署合约

### 1. PaymentContract (支付合约 - 测试版)

-   **状态**: ⏳ 待部署
-   **说明**: 测试环境尚未部署 PaymentContract 的测试版本

---

### 2. StakingAuthorization (质押授权)

-   **状态**: ⏳ 待部署
-   **说明**: 在 RWA-HUSD-Architecture.md 中有提及，但尚未找到已部署的合约地址

---

## ⚠️ 测试环境注意事项

### 数据隔离

-   🔄 **环境隔离**: 测试环境与生产环境数据完全隔离
-   🔑 **独立配置**: 使用独立的 SystemConfig 和 UserRegistry
-   📊 **独立数据**: 测试数据不会影响生产环境

### 测试代币

-   💧 **无实际价值**: 使用的是测试代币（MockUSDC, MockUSDT），无实际价值
-   🎁 **免费获取**: 可以通过测试合约免费获取测试代币
-   ⚠️ **仅供测试**: 测试代币仅用于功能测试和开发验证

### 快速迭代

-   🚀 **快速部署**: 可以快速部署和测试新功能
-   🔧 **灵活配置**: 可以灵活调整合约参数
-   📊 **性能测试**: 可以进行压力测试和性能评估

### 共享合约

-   🔗 **部分共享**: PropertyTokenFactory, Treasury 等合约共享生产环境
-   ⚠️ **谨慎操作**: 操作共享合约时需要特别小心，避免影响生产环境
-   📝 **建议**: 优先使用独立的测试版合约进行测试

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

-   **2025-10-12**: 创建测试环境合约地址文档
-   **2025-09-23**: AbleVestingFactory 部署
-   **2025-08-28**: RentCustodyContract v2.2.0 部署
-   **2025-08-20**: SystemConfig 和 UserRegistry V7 部署，RedemptionManager V6.2 部署
