# 运营手册

**文档版本**: v1.0  
**创建时间**: 2025-10-18 14:52:14 CST  
**适用范围**: ABLE 代币 Uniswap V2 日常运营

---

## 1. 日常运营流程

### 1.1 每日检查清单

**上午 (10:00 AM)**:
- [ ] 检查池流动性 (TVL)
- [ ] 检查 ABLE 价格
- [ ] 检查 24h 交易量
- [ ] 检查异常交易
- [ ] 检查合约状态 (是否暂停)

**下午 (3:00 PM)**:
- [ ] 检查流动性挖矿奖励余额
- [ ] 检查质押数量变化
- [ ] 检查社区反馈
- [ ] 更新运营日志

**晚上 (9:00 PM)**:
- [ ] 生成日报
- [ ] 规划次日工作
- [ ] 检查预警系统

### 1.2 每周任务

**周一**:
- [ ] 生成上周数据报告
- [ ] 分析 TVL 变化趋势
- [ ] 分析交易量变化
- [ ] 评估奖励率是否需要调整

**周三**:
- [ ] 检查合约 Gas 使用情况
- [ ] 优化运营流程
- [ ] 社区 AMA 准备

**周五**:
- [ ] 收集用户反馈
- [ ] 准备周报
- [ ] 规划下周工作

### 1.3 每月任务

- [ ] 生成月度报告
- [ ] 评估流动性挖矿效果
- [ ] 调整奖励分配策略
- [ ] 合约安全审查
- [ ] 备份关键数据

---

## 2. 流动性管理

### 2.1 添加流动性

**场景**: TVL 下降,需要增加流动性

**步骤**:
```javascript
// 1. 准备资金
const ableAmount = ethers.parseEther("100000");  // 100K ABLE
const usdcAmount = ethers.parseUnits("10000", 6);  // 10K USDC

// 2. 调用 Manager 合约
const manager = await ethers.getContractAt("UniswapV2LiquidityManager", MANAGER_ADDRESS);

const tx = await manager.addLiquidity(
    POOL_ADDRESS,
    ableAmount,
    usdcAmount
);

await tx.wait();
console.log("Liquidity added!");
```

**注意事项**:
- 确保 Treasury 有足够余额
- 检查当前价格,避免大幅偏离
- 分批添加,避免一次性大额操作
- 记录操作日志

### 2.2 移除流动性

**场景**: 需要调整流动性或紧急提现

**步骤**:
```javascript
// 1. 查询当前 LP 余额
const lpBalance = await manager.getLPBalance(POOL_ADDRESS);
console.log("Current LP balance:", ethers.formatEther(lpBalance));

// 2. 移除部分流动性
const liquidityToRemove = lpBalance / 10n;  // 移除 10%

const tx = await manager.removeLiquidity(
    POOL_ADDRESS,
    liquidityToRemove
);

const receipt = await tx.wait();
console.log("Liquidity removed!");

// 3. 检查返还的代币数量
const event = receipt.logs.find(log => log.eventName === "LiquidityRemoved");
console.log("ABLE returned:", ethers.formatEther(event.args.ableAmount));
console.log("USDC returned:", ethers.formatUnits(event.args.pairedAmount, 6));
```

**注意事项**:
- 评估对价格的影响
- 避免在交易高峰期操作
- 提前通知社区 (如果是大额操作)
- 记录操作原因

### 2.3 流动性再平衡

**场景**: 价格偏离目标,需要调整流动性分布

**步骤**:
```javascript
// 1. 检查当前价格
const pool = await ethers.getContractAt("IUniswapV2Pair", POOL_ADDRESS);
const [reserve0, reserve1] = await pool.getReserves();
const currentPrice = Number(reserve1) / Number(reserve0);
const targetPrice = 0.10;  // $0.10

console.log("Current price:", currentPrice);
console.log("Target price:", targetPrice);
console.log("Deviation:", (currentPrice - targetPrice) / targetPrice * 100, "%");

// 2. 如果偏离超过 5%,考虑再平衡
if (Math.abs(currentPrice - targetPrice) / targetPrice > 0.05) {
    // 移除部分流动性
    await manager.removeLiquidity(POOL_ADDRESS, lpBalance / 2n);
    
    // 重新计算最优比例
    const newAbleAmount = calculateOptimalAble(targetPrice);
    const newUsdcAmount = calculateOptimalUsdc(targetPrice);
    
    // 重新添加流动性
    await manager.addLiquidity(POOL_ADDRESS, newAbleAmount, newUsdcAmount);
}
```

---

## 3. 流动性挖矿管理

### 3.1 调整奖励率

**场景**: 根据 TVL 和市场情况调整奖励

**步骤**:
```javascript
const mining = await ethers.getContractAt("AbleLiquidityMining", MINING_ADDRESS);

// 1. 查询当前奖励率
const currentRate = await mining.rewardRate();
console.log("Current reward rate:", ethers.formatEther(currentRate), "ABLE/second");

// 2. 计算新奖励率
// 目标 APR = 30%
// TVL = $5M
// ABLE 价格 = $0.10
const targetAPR = 0.30;
const tvl = 5000000;  // $5M
const ablePrice = 0.10;

const yearlyRewards = tvl * targetAPR / ablePrice;
const newRate = yearlyRewards / (365 * 24 * 60 * 60);  // 每秒奖励

console.log("New reward rate:", newRate, "ABLE/second");

// 3. 更新奖励率
const tx = await mining.setRewardRate(ethers.parseEther(newRate.toString()));
await tx.wait();
console.log("Reward rate updated!");
```

**调整策略**:
- TVL 增加 → 降低 APR (保持总奖励稳定)
- TVL 减少 → 提高 APR (吸引流动性)
- 市场低迷 → 提高 APR (激励参与)
- 市场火热 → 降低 APR (节省奖励)

### 3.2 补充奖励代币

**场景**: 奖励余额不足,需要补充

**步骤**:
```javascript
const ableToken = await ethers.getContractAt("IERC20", ABLE_ADDRESS);
const mining = await ethers.getContractAt("AbleLiquidityMining", MINING_ADDRESS);

// 1. 检查当前余额
const currentBalance = await ableToken.balanceOf(MINING_ADDRESS);
console.log("Current reward balance:", ethers.formatEther(currentBalance));

// 2. 计算需要补充的数量
const rewardRate = await mining.rewardRate();
const daysRemaining = Number(currentBalance) / (Number(rewardRate) * 86400);
console.log("Days remaining:", daysRemaining);

// 3. 如果少于 30 天,补充奖励
if (daysRemaining < 30) {
    const topUpAmount = rewardRate * 86400n * 90n;  // 补充 90 天
    await ableToken.transfer(MINING_ADDRESS, topUpAmount);
    console.log("Rewards topped up!");
}
```

---

## 4. 监控和预警

### 4.1 价格监控

```javascript
// monitoring/price-monitor.js
const { ethers } = require("ethers");

async function monitorPrice() {
    const provider = new ethers.JsonRpcProvider("https://mainnet.optimism.io");
    const pool = new ethers.Contract(POOL_ADDRESS, PAIR_ABI, provider);
    
    const [reserve0, reserve1] = await pool.getReserves();
    const price = Number(reserve1) / Number(reserve0);
    
    const targetPrice = 0.10;
    const deviation = Math.abs(price - targetPrice) / targetPrice;
    
    if (deviation > 0.05) {  // 偏离 5%
        sendAlert(`⚠️ Price deviation: ${(deviation * 100).toFixed(2)}%`);
    }
    
    if (deviation > 0.10) {  // 偏离 10%
        sendUrgentAlert(`🚨 URGENT: Price deviation: ${(deviation * 100).toFixed(2)}%`);
    }
}

setInterval(monitorPrice, 60000);  // 每分钟检查
```

### 4.2 流动性监控

```javascript
// monitoring/liquidity-monitor.js
async function monitorLiquidity() {
    const pool = new ethers.Contract(POOL_ADDRESS, PAIR_ABI, provider);
    const [reserve0, reserve1] = await pool.getReserves();
    
    const tvl = Number(reserve0) * ablePrice + Number(reserve1);
    
    if (tvl < MIN_TVL_THRESHOLD) {
        sendAlert(`⚠️ Low liquidity: $${tvl.toLocaleString()}`);
    }
    
    // 检查流动性变化
    const previousTVL = await getPreviousTVL();
    const change = (tvl - previousTVL) / previousTVL;
    
    if (Math.abs(change) > 0.2) {  // 变化超过 20%
        sendAlert(`⚠️ Large liquidity change: ${(change * 100).toFixed(2)}%`);
    }
}
```

### 4.3 异常交易监控

```javascript
// monitoring/transaction-monitor.js
async function monitorTransactions() {
    const pool = new ethers.Contract(POOL_ADDRESS, PAIR_ABI, provider);
    
    pool.on("Swap", async (sender, amount0In, amount1In, amount0Out, amount1Out, to) => {
        const swapValue = calculateSwapValue(amount0In, amount1In, amount0Out, amount1Out);
        
        if (swapValue > LARGE_SWAP_THRESHOLD) {
            sendAlert(`⚠️ Large swap: $${swapValue.toLocaleString()} by ${sender}`);
        }
    });
    
    pool.on("Burn", async (sender, amount0, amount1, to) => {
        const value = Number(amount0) * ablePrice + Number(amount1);
        
        if (value > LARGE_WITHDRAWAL_THRESHOLD) {
            sendAlert(`⚠️ Large liquidity removal: $${value.toLocaleString()} by ${sender}`);
        }
    });
}
```

---

## 5. 应急响应

### 5.1 价格异常

**场景**: ABLE 价格突然暴涨/暴跌

**响应步骤**:
1. **立即调查**:
   - 检查是否有大额交易
   - 检查是否有异常合约调用
   - 检查其他交易所价格

2. **评估影响**:
   - 计算价格偏离度
   - 评估是否需要干预
   - 评估对用户的影响

3. **采取行动**:
   - 如果是攻击: 立即暂停合约
   - 如果是市场波动: 考虑添加流动性稳定价格
   - 发布公告说明情况

4. **后续处理**:
   - 分析根本原因
   - 更新应急预案
   - 加强监控

### 5.2 流动性枯竭

**场景**: 大量 LP 撤出,流动性不足

**响应步骤**:
1. **紧急补充流动性**:
   ```javascript
   // 从 Treasury 紧急添加流动性
   await manager.addLiquidity(
       POOL_ADDRESS,
       ethers.parseEther("500000"),  // 500K ABLE
       ethers.parseUnits("50000", 6)  // 50K USDC
   );
   ```

2. **提高挖矿奖励**:
   ```javascript
   // 临时提高 APR 到 100%
   const newRate = calculateRewardRate(100);  // 100% APR
   await mining.setRewardRate(newRate);
   ```

3. **社区沟通**:
   - 发布公告说明情况
   - 承诺采取措施
   - 征求社区意见

### 5.3 智能合约漏洞

**场景**: 发现或怀疑合约存在漏洞

**响应步骤**:
1. **立即暂停**:
   ```javascript
   const manager = await ethers.getContractAt("UniswapV2LiquidityManager", MANAGER_ADDRESS);
   await manager.pause();
   
   const mining = await ethers.getContractAt("AbleLiquidityMining", MINING_ADDRESS);
   await mining.pause();
   ```

2. **移除流动性**:
   ```javascript
   // 移除所有流动性到安全地址
   const lpBalance = await manager.getLPBalance(POOL_ADDRESS);
   await manager.removeLiquidity(POOL_ADDRESS, lpBalance);
   ```

3. **通知用户**:
   - 发布紧急公告
   - 说明情况和应对措施
   - 提供用户操作指南

4. **修复和恢复**:
   - 修复漏洞
   - 重新审计
   - 部署新合约
   - 迁移流动性

---

## 6. 数据报告

### 6.1 日报模板

```markdown
# ABLE Uniswap 日报 - YYYY-MM-DD

## 关键指标
- TVL: $X,XXX,XXX (+X.X%)
- ABLE 价格: $0.XX (+X.X%)
- 24h 交易量: $XXX,XXX
- LP 数量: XXX 个
- 质押 LP: XX%

## 重要事件
- [时间] 事件描述

## 异常情况
- 无 / 详细描述

## 明日计划
- 任务 1
- 任务 2
```

### 6.2 周报模板

```markdown
# ABLE Uniswap 周报 - Week XX, YYYY

## 本周概况
- TVL 变化: $X,XXX,XXX → $X,XXX,XXX (+X.X%)
- 平均日交易量: $XXX,XXX
- 新增 LP: XX 个
- ABLE 价格: $0.XX → $0.XX (+X.X%)

## 流动性挖矿
- 总质押: $X,XXX,XXX
- 本周奖励: XXX,XXX ABLE
- 平均 APR: XX%

## 运营亮点
- 亮点 1
- 亮点 2

## 问题和改进
- 问题 1 → 解决方案
- 问题 2 → 改进计划

## 下周计划
- 计划 1
- 计划 2
```

---

## 7. 工具和脚本

### 7.1 快速查询脚本

```javascript
// scripts/quick-stats.js
async function getQuickStats() {
    const pool = await ethers.getContractAt("IUniswapV2Pair", POOL_ADDRESS);
    const manager = await ethers.getContractAt("UniswapV2LiquidityManager", MANAGER_ADDRESS);
    const mining = await ethers.getContractAt("AbleLiquidityMining", MINING_ADDRESS);
    
    // 池信息
    const [reserve0, reserve1] = await pool.getReserves();
    const totalSupply = await pool.totalSupply();
    
    // Manager 信息
    const lpBalance = await manager.getLPBalance(POOL_ADDRESS);
    const [ableValue, usdcValue] = await manager.getLPValue(POOL_ADDRESS);
    
    // Mining 信息
    const rewardRate = await mining.rewardRate();
    const totalStaked = await mining.totalStaked();
    
    console.log("=== Quick Stats ===");
    console.log("Pool Reserves:", ethers.formatEther(reserve0), "ABLE,", ethers.formatUnits(reserve1, 6), "USDC");
    console.log("Manager LP:", ethers.formatEther(lpBalance));
    console.log("Manager Value:", ethers.formatEther(ableValue), "ABLE,", ethers.formatUnits(usdcValue, 6), "USDC");
    console.log("Reward Rate:", ethers.formatEther(rewardRate), "ABLE/second");
    console.log("Total Staked:", ethers.formatEther(totalStaked), "LP");
}

getQuickStats();
```

### 7.2 批量操作脚本

```javascript
// scripts/batch-operations.js
async function batchAddLiquidity(pools, amounts) {
    for (let i = 0; i < pools.length; i++) {
        console.log(`Adding liquidity to pool ${i + 1}/${pools.length}...`);
        await manager.addLiquidity(pools[i], amounts[i].able, amounts[i].paired);
        console.log("✓ Done");
    }
}
```

---

**最后更新**: 2025-10-18 14:52:14 CST

**重要提示**:
1. 所有操作都要记录日志
2. 重大操作前要备份数据
3. 保持与社区的透明沟通
4. 定期审查和更新运营流程

