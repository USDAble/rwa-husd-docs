# Uniswap V2 技术规格说明

**文档版本**: v1.0  
**创建时间**: 2025-10-18 14:52:14 CST  
**适用范围**: ABLE 代币 Uniswap V2 集成

---

## 1. Uniswap V2 协议概述

### 1.1 核心概念

**Uniswap V2** 是一个自动化做市商 (AMM) 协议,使用恒定乘积公式:

```
x * y = k
```

其中:
- `x` = Token0 储备量
- `y` = Token1 储备量
- `k` = 恒定值

### 1.2 核心合约

#### Factory 合约

**地址 (OP Mainnet)**: `0x0c3c1c532F1e39EdF36BE9Fe0bE1410313E074Bf`

**核心功能**:
```solidity
interface IUniswapV2Factory {
    // 创建交易对
    function createPair(address tokenA, address tokenB) external returns (address pair);
    
    // 获取交易对地址
    function getPair(address tokenA, address tokenB) external view returns (address pair);
    
    // 所有交易对数量
    function allPairsLength() external view returns (uint);
    
    // 手续费接收地址
    function feeTo() external view returns (address);
    
    // 手续费开关控制地址
    function feeToSetter() external view returns (address);
}
```

#### Router 合约

**地址 (OP Mainnet)**: `0x4A7b5Da61326A6379179b40d00F57E5bbDC962c2`

**核心功能**:
```solidity
interface IUniswapV2Router02 {
    // 添加流动性 (ERC20/ERC20)
    function addLiquidity(
        address tokenA,
        address tokenB,
        uint amountADesired,
        uint amountBDesired,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB, uint liquidity);
    
    // 添加流动性 (ETH/ERC20)
    function addLiquidityETH(
        address token,
        uint amountTokenDesired,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external payable returns (uint amountToken, uint amountETH, uint liquidity);
    
    // 移除流动性 (ERC20/ERC20)
    function removeLiquidity(
        address tokenA,
        address tokenB,
        uint liquidity,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB);
    
    // 移除流动性 (ETH/ERC20)
    function removeLiquidityETH(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external returns (uint amountToken, uint amountETH);
    
    // 交换 (精确输入)
    function swapExactTokensForTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
    
    // 交换 (精确输出)
    function swapTokensForExactTokens(
        uint amountOut,
        uint amountInMax,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
    
    // 获取输出数量
    function getAmountsOut(uint amountIn, address[] calldata path)
        external view returns (uint[] memory amounts);
    
    // 获取输入数量
    function getAmountsIn(uint amountOut, address[] calldata path)
        external view returns (uint[] memory amounts);
}
```

#### Pair 合约

**核心功能**:
```solidity
interface IUniswapV2Pair {
    // ERC20 标准函数
    function totalSupply() external view returns (uint);
    function balanceOf(address owner) external view returns (uint);
    function transfer(address to, uint value) external returns (bool);
    function approve(address spender, uint value) external returns (bool);
    function transferFrom(address from, address to, uint value) external returns (bool);
    
    // Pair 特有函数
    function token0() external view returns (address);
    function token1() external view returns (address);
    function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
    function price0CumulativeLast() external view returns (uint);
    function price1CumulativeLast() external view returns (uint);
    function kLast() external view returns (uint);
    
    // 流动性操作
    function mint(address to) external returns (uint liquidity);
    function burn(address to) external returns (uint amount0, uint amount1);
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;
    function skim(address to) external;
    function sync() external;
}
```

---

## 2. ABLE 代币流动性池设计

### 2.1 池配置

#### ABLE/USDC 主池

```javascript
{
    token0: "ABLE",
    token1: "USDC (0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85)",
    fee: "0.3%",  // Uniswap V2 固定费率
    initialLiquidity: {
        able: "2,500,000 ABLE",
        usdc: "250,000 USDC"
    },
    initialPrice: "$0.10 per ABLE",
    targetTVL: "$5,000,000"
}
```

#### ABLE/ETH 辅助池

```javascript
{
    token0: "ABLE",
    token1: "WETH (0x4200000000000000000000000000000000000006)",
    fee: "0.3%",
    initialLiquidity: {
        able: "2,000,000 ABLE",
        eth: "80 ETH"
    },
    initialPrice: "$0.10 per ABLE (ETH @ $2,500)",
    targetTVL: "$2,000,000"
}
```

### 2.2 价格计算

#### 公式

```
Price(ABLE) = Reserve(USDC) / Reserve(ABLE)
```

#### 示例

```javascript
// 初始状态
reserve_ABLE = 2,500,000
reserve_USDC = 250,000

price_ABLE = 250,000 / 2,500,000 = 0.10 USDC

// 买入 10,000 ABLE 后
// 使用恒定乘积公式: x * y = k
k = 2,500,000 * 250,000 = 625,000,000,000

// 新的储备量
new_reserve_ABLE = 2,490,000  // 减少 10,000
new_reserve_USDC = k / new_reserve_ABLE = 251,004.016  // 增加 1,004.016

// 新价格
new_price_ABLE = 251,004.016 / 2,490,000 = 0.1008 USDC

// 价格上涨 0.8%
```

### 2.3 滑点计算

```javascript
function calculateSlippage(amountIn, reserveIn, reserveOut) {
    // Uniswap V2 公式 (扣除 0.3% 手续费)
    const amountInWithFee = amountIn * 997;
    const numerator = amountInWithFee * reserveOut;
    const denominator = (reserveIn * 1000) + amountInWithFee;
    const amountOut = numerator / denominator;
    
    // 理想输出 (无滑点)
    const idealAmountOut = (amountIn * reserveOut) / reserveIn;
    
    // 滑点
    const slippage = (idealAmountOut - amountOut) / idealAmountOut * 100;
    
    return slippage;
}

// 示例: 买入 10,000 ABLE
const slippage = calculateSlippage(
    1000,        // 1000 USDC
    250000,      // USDC 储备
    2500000      // ABLE 储备
);
// slippage ≈ 0.4%
```

---

## 3. 合约接口定义

### 3.1 UniswapV2LiquidityManager

```solidity
interface IUniswapV2LiquidityManager {
    // 事件
    event LiquidityAdded(
        address indexed pool,
        uint256 ableAmount,
        uint256 otherAmount,
        uint256 liquidity
    );
    
    event LiquidityRemoved(
        address indexed pool,
        uint256 liquidity,
        uint256 ableAmount,
        uint256 otherAmount
    );
    
    event FeesCollected(
        address indexed pool,
        uint256 ableAmount,
        uint256 otherAmount
    );
    
    // 创建池
    function createPool(
        address tokenA,
        address tokenB,
        uint256 amountA,
        uint256 amountB
    ) external returns (address pool, uint256 liquidity);
    
    // 添加流动性
    function addLiquidity(
        address tokenA,
        address tokenB,
        uint256 amountA,
        uint256 amountB
    ) external returns (uint256 liquidity);
    
    // 移除流动性
    function removeLiquidity(
        address pool,
        uint256 liquidity
    ) external returns (uint256 amountA, uint256 amountB);
    
    // 查询 LP 价值
    function getLPValue(address pool) external view returns (
        uint256 ableAmount,
        uint256 otherAmount
    );
    
    // 查询 LP 余额
    function getLPBalance(address pool) external view returns (uint256);
}
```

### 3.2 AbleLiquidityMining

```solidity
interface IAbleLiquidityMining {
    // 事件
    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount);
    event RewardsClaimed(address indexed user, uint256 amount);
    event RewardRateUpdated(uint256 newRate);
    
    // 质押
    function stake(uint256 amount) external;
    
    // 解除质押
    function unstake(uint256 amount) external;
    
    // 领取奖励
    function claimRewards() external;
    
    // 查询待领取奖励
    function pendingRewards(address user) external view returns (uint256);
    
    // 查询质押信息
    function getStakeInfo(address user) external view returns (
        uint256 amount,
        uint256 rewardDebt,
        uint256 lastUpdateTime
    );
    
    // 管理员函数
    function setRewardRate(uint256 newRate) external;
    function pause() external;
    function unpause() external;
}
```

---

## 4. 数据结构设计

### 4.1 StakeInfo

```solidity
struct StakeInfo {
    uint256 amount;          // 质押的 LP Token 数量
    uint256 rewardDebt;      // 已计算但未领取的奖励
    uint256 lastUpdateTime;  // 最后更新时间
}
```

### 4.2 PoolInfo

```solidity
struct PoolInfo {
    address lpToken;         // LP Token 地址
    uint256 allocPoint;      // 分配权重 (用于多池)
    uint256 lastRewardTime;  // 最后奖励时间
    uint256 accRewardPerShare;  // 累计每股奖励
    uint256 totalStaked;     // 总质押量
}
```

### 4.3 LiquidityPosition

```solidity
struct LiquidityPosition {
    address pool;            // 池地址
    uint256 liquidity;       // LP Token 数量
    uint256 ableAmount;      // ABLE 数量
    uint256 otherAmount;     // 配对代币数量
    uint256 addedAt;         // 添加时间
}
```

---

## 5. 安全参数

### 5.1 滑点保护

```solidity
// 最大滑点 (basis points)
uint256 public constant MAX_SLIPPAGE = 500;  // 5%

// 计算最小输出
function calculateMinOutput(
    uint256 amountIn,
    uint256 reserveIn,
    uint256 reserveOut
) public pure returns (uint256) {
    uint256 amountOut = getAmountOut(amountIn, reserveIn, reserveOut);
    uint256 minOutput = amountOut * (10000 - MAX_SLIPPAGE) / 10000;
    return minOutput;
}
```

### 5.2 时间锁

```solidity
// 操作截止时间 (5 分钟)
uint256 public constant DEADLINE_DURATION = 300;

function getDeadline() public view returns (uint256) {
    return block.timestamp + DEADLINE_DURATION;
}
```

### 5.3 重入保护

```solidity
// 使用 OpenZeppelin ReentrancyGuard
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract UniswapV2LiquidityManager is ReentrancyGuard {
    function addLiquidity(...) external nonReentrant {
        // ...
    }
}
```

---

## 6. Gas 优化

### 6.1 批量操作

```solidity
// 批量添加流动性
function batchAddLiquidity(
    address[] calldata pools,
    uint256[] calldata amounts
) external {
    require(pools.length == amounts.length, "Length mismatch");
    
    for (uint256 i = 0; i < pools.length; i++) {
        _addLiquidity(pools[i], amounts[i]);
    }
}
```

### 6.2 存储优化

```solidity
// 使用 uint128 代替 uint256 (如果数值范围允许)
struct OptimizedStakeInfo {
    uint128 amount;
    uint128 rewardDebt;
    uint64 lastUpdateTime;
}
```

---

**最后更新**: 2025-10-18 14:52:14 CST

