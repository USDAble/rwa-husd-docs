# BSC Testnet 部署合约地址

**部署时间**: 2025-11-01  
**网络**: BSC Testnet (Chain ID: 97)  
**RPC URL**: https://data-seed-prebsc-1-s1.binance.org:8545/  
**浏览器**: https://testnet.bscscan.com  
**部署者**: 0xBdd427CFFA233858024B1D74220A9669918dC8a2

---

## 📋 合约地址汇总

### 核心合约 (Core Contracts)

| 合约名称     | 类型       | 代理地址 (Proxy)                             | 实现地址 (Implementation)                    | BscScan 链接                                                                           |
| ------------ | ---------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------- |
| SystemConfig | UUPS Proxy | `0x25A6661277eE483Ad72e28782A2601E648C13A30` | `0xb5C8Bb2D422C4909663261A2164263D1fCF9A0dd` | [查看](https://testnet.bscscan.com/address/0x25A6661277eE483Ad72e28782A2601E648C13A30) |
| UserRegistry | UUPS Proxy | `0x83A83DCDEE5E40bA5Cafb3409Df42F079F38a909` | `0xdd2AC83e35562D15479c8455aBAe887aaCAC018d` | [查看](https://testnet.bscscan.com/address/0x83A83DCDEE5E40bA5Cafb3409Df42F079F38a909) |
| MockUSDC     | ERC20      | `0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925` | -                                            | [查看](https://testnet.bscscan.com/address/0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925) |

### 赎回管理合约 (Redemption Contracts)

| 合约名称          | 类型       | 代理地址 (Proxy)                             | 实现地址 (Implementation)                    | BscScan 链接                                                                           |
| ----------------- | ---------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------- |
| RedemptionManager | UUPS Proxy | `0xf64A952e81bbBAa6C8c134e4ceb943dcB59F4772` | `0xA805cca025175E16fda96B92dCf567900EB11Dea` | [查看](https://testnet.bscscan.com/address/0xf64A952e81bbBAa6C8c134e4ceb943dcB59F4772) |

### 国库合约 (Treasury Contracts)

| 合约名称           | 类型       | 代理地址 (Proxy)                             | 实现地址 (Implementation)                    | BscScan 链接                                                                           |
| ------------------ | ---------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------- |
| Treasury           | UUPS Proxy | `0x956dc62ea643665B604F3a5Fb2A0DEF1f1f6A5b9` | `0x2169C2AfA9A94564AEb8bFAd683fA660D58F94b2` | [查看](https://testnet.bscscan.com/address/0x956dc62ea643665B604F3a5Fb2A0DEF1f1f6A5b9) |
| RedemptionStrategy | UUPS Proxy | `0x4941CCBAB7682dceF04135375FBFBFfd52EffFB4` | `0x3008bE2056f0135B4cf5248BB81AB6A5f7cF48Db` | [查看](https://testnet.bscscan.com/address/0x4941CCBAB7682dceF04135375FBFBFfd52EffFB4) |

### 交易合约 (Trading Contracts)

| 合约名称      | 类型       | 代理地址 (Proxy)                             | 实现地址 (Implementation)                    | BscScan 链接                                                                           |
| ------------- | ---------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------- |
| TradeContract | UUPS Proxy | `0x813Cb5D7ce1fFB83516cb3B97aB530D2eA6f67c1` | `0x2EEC668270F57F5F48Fc38CDC95f4d532E82a710` | [查看](https://testnet.bscscan.com/address/0x813Cb5D7ce1fFB83516cb3B97aB530D2eA6f67c1) |

### 批量转账合约 (Batch Transfer Contracts)

| 合约名称            | 类型       | 代理地址 (Proxy)                             | 实现地址 (Implementation)                    | BscScan 链接                                                                           |
| ------------------- | ---------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------- |
| RentCustodyContract | UUPS Proxy | `0x11c60DaFc9e99a12183c4b3c73f528a9BAAc7e00` | `0x851b4eE76FeD997a251e048977F042ac11dE54E5` | [查看](https://testnet.bscscan.com/address/0x11c60DaFc9e99a12183c4b3c73f528a9BAAc7e00) |

### 房产代币合约 (Property Token Contracts)

| 合约名称                       | 类型           | 代理地址 (Proxy)                             | 实现地址 (Implementation)                    | BscScan 链接                                                                           |
| ------------------------------ | -------------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------- |
| PropertyTokenFactory           | UUPS Proxy     | `0x22401F8Fd016ede8D5B778f74DdAea6A1BB70Ad7` | `0x1959b00722A6ebf6da90c4a0A7D04CF9049a2E97` | [查看](https://testnet.bscscan.com/address/0x22401F8Fd016ede8D5B778f74DdAea6A1BB70Ad7) |
| PropertyToken (Implementation) | Implementation | -                                            | `0x1Be5770A9471370Fd5fA8b8cB193c9DD69CC47a4` | [查看](https://testnet.bscscan.com/address/0x1Be5770A9471370Fd5fA8b8cB193c9DD69CC47a4) |
| PropertyToken RPT1             | ERC20          | `0x721a2320f19C336B05dca6C5fAaa972d5Ff88546` | -                                            | [查看](https://testnet.bscscan.com/address/0x721a2320f19C336B05dca6C5fAaa972d5Ff88546) |
| PropertyToken RPT2             | ERC20          | `0xE3Ee8b93Dc07913d652330598D84F046C32AE98B` | -                                            | [查看](https://testnet.bscscan.com/address/0xE3Ee8b93Dc07913d652330598D84F046C32AE98B) |

---

## 📊 部署统计

-   **总合约数**: 13 个
-   **累计 Gas 消耗**: 30,454,507 gas
-   **累计费用**: 0.0030454507 BNB (约 $1.83 USD)
-   **累计交易数**: 59 笔
-   **平均 Gas 价格**: 0.1 gwei
-   **部署时间**: 约 60 分钟

---

## 👥 管理员地址

所有管理员都已配置以下权限:

-   DEFAULT_ADMIN_ROLE
-   ADMIN_ROLE
-   OPERATOR_ROLE
-   KYC 批准状态

| 管理员             | 地址                                         | MockUSDC 余额 |
| ------------------ | -------------------------------------------- | ------------- |
| ADMIN_1            | `0xefAEc4605722CA1B44280cCEaC669020F6856A07` | 300,000 USDC  |
| ADMIN_2            | `0xcc76a9BeEf0dF6F0ccD29515588c245F98744351` | 300,000 USDC  |
| ADMIN_3            | `0x30a12A0960B8F04f6fdd21493fe6708CE1e7C0C9` | 300,000 USDC  |
| ADMIN_4 (Deployer) | `0xBdd427CFFA233858024B1D74220A9669918dC8a2` | 300,000 USDC  |
| ADMIN_5            | `0xe0d1A6498572c57a4f111E4cf8aD57430c92E23E` | 300,000 USDC  |

---

## 🔧 配置信息

### SystemConfig

-   **默认支付代币**: MockUSDC (`0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925`)
-   **自动配置**: 在 DeployCore.s.sol 中自动设置

### RedemptionManager

-   **费率**: 1000 (10%)
-   **SystemConfig**: `0x25A6661277eE483Ad72e28782A2601E648C13A30`
-   **UserRegistry**: `0x83A83DCDEE5E40bA5Cafb3409Df42F079F38a909`
-   **PaymentToken**: `0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925`

### RentCustodyContract

-   **版本**: 2.2.0
-   **最大批量大小**: 200
-   **SystemConfig**: `0x25A6661277eE483Ad72e28782A2601E648C13A30`
-   **默认支付代币**: `0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925`

---

## ⚠️ 重要发现

### Treasury 地址不一致

在 DeployBusiness.s.sol 日志中发现 Treasury 地址显示为 `0xB9C3FC6c37Baf28190D7B01090371ee893071348`,这与实际部署的 Treasury Proxy 地址 (`0x956dc62ea643665B604F3a5Fb2A0DEF1f1f6A5b9`) 不同。

**原因**: 从 `.env` 文件读取的旧地址  
**影响**: 不影响部署成功,但需要在后续验证时注意  
**建议**: 更新所有仓库的 `.env` 文件中的 TREASURY_ADDRESS

---

## 📝 部署顺序

1. **Batch 1** (2025-11-01 10:30): SystemConfig, UserRegistry, MockUSDC
2. **Batch 2** (2025-11-01 10:35): RedemptionManager
3. **Batch 3** (2025-11-01 10:40): Treasury, RedemptionStrategy
4. **Batch 4** (2025-11-01 10:45): TradeContract
5. **Batch 5** (2025-11-01 10:50): RentCustodyContract
6. **Batch 6** (2025-11-01 11:00): PropertyTokenFactory, PropertyToken Implementation, RPT1, RPT2

---

## 🔗 相关文档

-   [OP Sepolia 部署记录](./Deployed-Contracts-OP-Sepolia.md)
-   [BSC Testnet 部署计划](./project_document/[004]BSC-Testnet部署计划.md)
-   [OP Sepolia 部署计划](./project_document/[003]OP-Sepolia部署计划.md)

---

## 📌 快速复制配置

```bash
# BSC Testnet 配置
export BSC_TESTNET_RPC_URL="https://data-seed-prebsc-1-s1.binance.org:8545/"
export SYSTEM_CONFIG_ADDRESS="0x25A6661277eE483Ad72e28782A2601E648C13A30"
export USER_REGISTRY_ADDRESS="0x83A83DCDEE5E40bA5Cafb3409Df42F079F38a909"
export MOCK_USDC_ADDRESS="0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925"
export REDEMPTION_MANAGER_ADDRESS="0xf64A952e81bbBAa6C8c134e4ceb943dcB59F4772"
export TREASURY_ADDRESS="0x956dc62ea643665B604F3a5Fb2A0DEF1f1f6A5b9"
export REDEMPTION_STRATEGY_ADDRESS="0x4941CCBAB7682dceF04135375FBFBFfd52EffFB4"
export TRADE_CONTRACT_ADDRESS="0x813Cb5D7ce1fFB83516cb3B97aB530D2eA6f67c1"
export RENT_CUSTODY_ADDRESS="0x11c60DaFc9e99a12183c4b3c73f528a9BAAc7e00"
export PROPERTY_TOKEN_FACTORY_ADDRESS="0x22401F8Fd016ede8D5B778f74DdAea6A1BB70Ad7"
```

---

## ✅ 合约验证状态

**所有核心合约已在 BscScan 上成功验证!**

**已验证的合约** (10/13):

1. ✅ SystemConfig Implementation - [已验证](https://testnet.bscscan.com/address/0xb5C8Bb2D422C4909663261A2164263D1fCF9A0dd#code)
2. ✅ UserRegistry Implementation - [已验证](https://testnet.bscscan.com/address/0xdd2AC83e35562D15479c8455aBAe887aaCAC018d#code)
3. ✅ MockUSDC - [已验证](https://testnet.bscscan.com/address/0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925#code)
4. ✅ RedemptionManager Implementation - [已验证](https://testnet.bscscan.com/address/0xA805cca025175E16fda96B92dCf567900EB11Dea#code)
5. ✅ Treasury Implementation - [已验证](https://testnet.bscscan.com/address/0x2169C2AfA9A94564AEb8bFAd683fA660D58F94b2#code)
6. ✅ RedemptionStrategy Implementation - [已验证](https://testnet.bscscan.com/address/0x3008bE2056f0135B4cf5248BB81AB6A5f7cF48Db#code)
7. ✅ TradeContract Implementation - [已验证](https://testnet.bscscan.com/address/0x2EEC668270F57F5F48Fc38CDC95f4d532E82a710#code)
8. ✅ RentCustodyContract Implementation - [已验证](https://testnet.bscscan.com/address/0x851b4eE76FeD997a251e048977F042ac11dE54E5#code)
9. ✅ PropertyToken Implementation - [已验证](https://testnet.bscscan.com/address/0x1Be5770A9471370Fd5fA8b8cB193c9DD69CC47a4#code)
10. ✅ PropertyTokenFactory Implementation - [已验证](https://testnet.bscscan.com/address/0x1959b00722A6ebf6da90c4a0A7D04CF9049a2E97#code)

**未验证的合约** (3/13 - Proxy 合约,无需验证):

-   SystemConfig Proxy
-   UserRegistry Proxy
-   PropertyTokenFactory Proxy

**验证命令**:

```bash
# 使用 --chain bsc-testnet 参数即可自动验证
forge verify-contract <address> <contract_path>:<contract_name> --chain bsc-testnet --watch
```

---

**最后更新**: 2025-11-01 11:45:00 CST
