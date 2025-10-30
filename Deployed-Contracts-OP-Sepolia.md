# RWA-HUSD OP Sepolia 测试网合约地址

## �� 网络信息

- **网络名称**: OP Sepolia Testnet
- **Chain ID**: 11155420
- **RPC URL**: https://sepolia.optimism.io
- **区块链浏览器**: https://sepolia-optimism.etherscan.io
- **环境状态**: 🧪 测试环境
- **分支**: op-sepolia
- **文档创建时间**: 2025-10-30 11:47:57 CST

---

## 📊 核心合约地址

### 1. SystemConfig (系统配置)

- **代理合约**: `0x25A6661277eE483Ad72e28782A2601E648C13A30`
- **实现合约**: `0x4b97b05797aAd140bA3cc44a9e009262c80D2035`
- **版本**: V9 UUPS 升级支持
- **部署时间**: 2025-10-30 08:30:15 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0x25A6661277eE483Ad72e28782A2601E648C13A30)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0x4b97b05797aAd140bA3cc44a9e009262c80D2035)
- **来源 repo**: rwa-husd-contracts

---

### 2. UserRegistry (用户注册)

- **代理合约**: `0x83A83DCDEE5E40bA5Cafb3409Df42F079F38a909`
- **实现合约**: `0x1EE6315C40A07086D7ea5DDA42Dfe3EAd954D192`
- **版本**: V9 UUPS 升级支持
- **部署时间**: 2025-10-30 08:30:15 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0x83A83DCDEE5E40bA5Cafb3409Df42F079F38a909)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0x1EE6315C40A07086D7ea5DDA42Dfe3EAd954D192)
- **来源 repo**: rwa-husd-contracts

---

### 3. MockUSDC (测试 USDC)

- **合约地址**: `0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925`
- **部署时间**: 2025-10-30 08:30:15 CST
- **验证状态**: ⏳ 待验证
- **区块链浏览器**: [合约地址](https://sepolia-optimism.etherscan.io/address/0x2FB6ABc41Fb257D17dFDa14463E635bb481e6925)
- **来源 repo**: rwa-husd-contracts
- **初始供应量**: 1,500,000 USDC (每个管理员 300,000)

---

### 4. RedemptionManager (赎回管理)

- **代理合约**: `0xf64A952e81bbBAa6C8c134e4ceb943dcB59F4772`
- **实现合约**: `0x010D113A7Ab7b71B3A2153e83aA203aebd9D5B9e`
- **版本**: V9 UUPS 升级支持
- **部署时间**: 2025-10-30 08:45:22 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0xf64A952e81bbBAa6C8c134e4ceb943dcB59F4772)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0x010D113A7Ab7b71B3A2153e83aA203aebd9D5B9e)
- **来源 repo**: redemption-contracts

---

## 📊 Treasury 合约地址

### 5. Treasury (国库)

- **代理合约**: `0xB9C3FC6c37Baf28190D7B01090371ee893071348`
- **实现合约**: `0x010D113A7Ab7b71B3A2153e83aA203aebd9D5B9e`
- **版本**: V9 UUPS 升级支持
- **部署时间**: 2025-10-30 11:12:45 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0xB9C3FC6c37Baf28190D7B01090371ee893071348)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0x010D113A7Ab7b71B3A2153e83aA203aebd9D5B9e)
- **来源 repo**: treasury-contracts
- **注意**: 重新部署以修复 SystemConfig 地址问题

---

### 6. RedemptionStrategy (赎回策略)

- **代理合约**: `0x6Ff4Bcc002a44669cc698Af611D39f82c8745D4C`
- **实现合约**: `0x4905d05bA60D9ea785e49257837A7DF251E908B7`
- **版本**: V9 UUPS 升级支持
- **部署时间**: 2025-10-30 11:12:45 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0x6Ff4Bcc002a44669cc698Af611D39f82c8745D4C)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0x4905d05bA60D9ea785e49257837A7DF251E908B7)
- **来源 repo**: treasury-contracts
- **注册状态**: ✅ 已注册到 Treasury
- **注册交易**: `0x86491b7b35e0c07bddaf28ac607dc4d8fcbbbbc7146aa61fed3e996821e1adbe`

---

## 📊 业务合约地址

### 7. TradeContract (交易合约)

- **代理合约**: `0x6D80d8FE131C6A9E017E92BD76f662Bd1559F176`
- **实现合约**: `0xd710821D9Edb6490EB6fF0dA5C572A64c176dCA8`
- **版本**: V9 UUPS 升级支持
- **部署时间**: 2025-10-30 11:25:33 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0x6D80d8FE131C6A9E017E92BD76f662Bd1559F176)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0xd710821D9Edb6490EB6fF0dA5C572A64c176dCA8)
- **来源 repo**: rwa-husd-trading

---

### 8. RentCustodyContract (租金托管合约)

- **代理合约**: `0x4945136f18677487f94DF2eCe27D4f9C6999Ee4C`
- **实现合约**: `0xE6a03808a57CE5F105BF9165Bd24e1E465c8e824`
- **版本**: V2.2.0
- **部署时间**: 2025-10-30 11:35:18 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0x4945136f18677487f94DF2eCe27D4f9C6999Ee4C)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0xE6a03808a57CE5F105BF9165Bd24e1E465c8e824)
- **来源 repo**: batch-token-transfer/hardhat-contracts

---

### 9. PropertyTokenFactory (房产代币工厂)

- **代理合约**: `0xba148486550c757F0BD591ed0fbF0fe95F2BE251`
- **实现合约**: `0xc9b6b482607DD02c495271A86701a17ac3464851`
- **版本**: V9 UUPS 升级支持
- **部署时间**: 2025-10-30 11:45:25 CST
- **架构**: UUPS 代理模式
- **验证状态**: ⏳ 待验证
- **区块链浏览器**:
  - [代理合约](https://sepolia-optimism.etherscan.io/address/0xba148486550c757F0BD591ed0fbF0fe95F2BE251)
  - [实现合约](https://sepolia-optimism.etherscan.io/address/0xc9b6b482607DD02c495271A86701a17ac3464851)
- **来源 repo**: rwa-husd-contracts

---

### 10. PropertyToken Implementation (房产代币实现)

- **实现合约**: `0xe987E270d485B628386840cd81C41d2811213FFB`
- **部署时间**: 2025-10-30 11:45:25 CST
- **验证状态**: ⏳ 待验证
- **区块链浏览器**: [实现合约](https://sepolia-optimism.etherscan.io/address/0xe987E270d485B628386840cd81C41d2811213FFB)
- **来源 repo**: rwa-husd-contracts

---

### 11. PropertyToken 1 (RPT1)

- **代理合约**: `0x2Ee1b62a5C7F2388a93Ae6feA753a367f9EDFCec`
- **名称**: RWA Property Token 1
- **符号**: RPT1
- **总供应量**: 500,000
- **可售数量**: 500,000
- **部署时间**: 2025-10-30 11:45:25 CST
- **验证状态**: ⏳ 待验证
- **区块链浏览器**: [代理合约](https://sepolia-optimism.etherscan.io/address/0x2Ee1b62a5C7F2388a93Ae6feA753a367f9EDFCec)
- **来源 repo**: rwa-husd-contracts

---

### 12. PropertyToken 2 (RPT2)

- **代理合约**: `0xF878Dda6F71e3e5689232688D57Aaf3dB744029b`
- **名称**: RWA Property Token 2
- **符号**: RPT2
- **总供应量**: 500,000
- **可售数量**: 500,000
- **部署时间**: 2025-10-30 11:45:25 CST
- **验证状态**: ⏳ 待验证
- **区块链浏览器**: [代理合约](https://sepolia-optimism.etherscan.io/address/0xF878Dda6F71e3e5689232688D57Aaf3dB744029b)
- **来源 repo**: rwa-husd-contracts

---

## 🔑 管理员地址

- **ADMIN_1**: `0xefAEc4605722CA1B44280cCEaC669020F6856A07`
- **ADMIN_2**: `0xcc76a9BeEf0dF6F0ccD29515588c245F98744351`
- **ADMIN_3**: `0x30a12A0960B8F04f6fdd21493fe6708CE1e7C0C9`
- **ADMIN_4 (Deployer)**: `0xBdd427CFFA233858024B1D74220A9669918dC8a2`
- **ADMIN_5**: `0xe0d1A6498572c57a4f111E4cf8aD57430c92E23E`

每个管理员持有 300,000 MockUSDC。

---

## �� 重要说明

### Treasury 重新部署
- **原因**: 原部署使用了错误的 SystemConfig 地址 (`0x4b97...2035` 而不是 `0x25A6...3A30`)
- **影响**: 导致 RedemptionStrategy 注册失败
- **解决**: 重新部署 Treasury 和所有依赖合约 (TradeContract, RentCustodyContract, PropertyTokenFactory)

### 验证状态
- ✅ 所有合约初始化成功
- ✅ 所有权限配置完成
- ✅ RedemptionStrategy 成功注册到 Treasury
- ✅ 所有 PropertyToken 创建成功并激活
- ⏳ Etherscan 源码验证待完成

---

## 📚 相关文档

- [部署计划](/project_document/[003]OP-Sepolia部署计划.md)
- [RWA-HUSD 架构文档](/RWA-HUSD-Architecture.md)
- [OP Mainnet 测试环境](/Deployed-Contracts-Testnet.md)
- [生产环境合约](/Deployed-Contracts-Production.md)

---

**最后更新**: 2025-10-30 11:47:57 CST
