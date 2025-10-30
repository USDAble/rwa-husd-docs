# 合约实现方案

**文档版本**: v1.0  
**创建时间**: 2025-10-18 14:52:14 CST  
**适用范围**: ABLE 代币 Uniswap V2 集成合约实现

---

## 1. UniswapV2LiquidityManager 实现

### 1.1 合约概述

**功能**: 管理 ABLE 代币在 Uniswap V2 上的流动性

**依赖**:
- OpenZeppelin Contracts v4.9.0+
- Uniswap V2 Core
- Uniswap V2 Periphery

### 1.2 核心实现 (伪代码)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

interface IUniswapV2Router02 {
    function addLiquidity(...) external returns (uint, uint, uint);
    function removeLiquidity(...) external returns (uint, uint);
}

interface IUniswapV2Factory {
    function createPair(address, address) external returns (address);
    function getPair(address, address) external view returns (address);
}

interface IUniswapV2Pair {
    function getReserves() external view returns (uint112, uint112, uint32);
    function token0() external view returns (address);
    function token1() external view returns (address);
    function totalSupply() external view returns (uint);
}

contract UniswapV2LiquidityManager is AccessControl, ReentrancyGuard, Pausable {
    using SafeERC20 for IERC20;
    
    // 角色定义
    bytes32 public constant OPERATOR_ROLE = keccak256("OPERATOR_ROLE");
    bytes32 public constant EMERGENCY_ROLE = keccak256("EMERGENCY_ROLE");
    
    // 核心合约
    IUniswapV2Router02 public immutable router;
    IUniswapV2Factory public immutable factory;
    IERC20 public immutable ableToken;
    address public immutable treasury;
    
    // 流动性头寸
    mapping(address => uint256) public lpBalances;  // pool => LP amount
    
    // 配置参数
    uint256 public constant MAX_SLIPPAGE = 500;  // 5% (basis points)
    uint256 public constant DEADLINE_DURATION = 300;  // 5 minutes
    
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
    
    // 构造函数
    constructor(
        address _router,
        address _factory,
        address _ableToken,
        address _treasury,
        address _admin
    ) {
        require(_router != address(0), "Invalid router");
        require(_factory != address(0), "Invalid factory");
        require(_ableToken != address(0), "Invalid ABLE token");
        require(_treasury != address(0), "Invalid treasury");
        
        router = IUniswapV2Router02(_router);
        factory = IUniswapV2Factory(_factory);
        ableToken = IERC20(_ableToken);
        treasury = _treasury;
        
        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
        _grantRole(OPERATOR_ROLE, _admin);
        _grantRole(EMERGENCY_ROLE, _admin);
    }
    
    // 创建池并添加初始流动性
    function createPoolAndAddLiquidity(
        address pairedToken,
        uint256 ableAmount,
        uint256 pairedAmount
    ) external onlyRole(OPERATOR_ROLE) nonReentrant whenNotPaused 
      returns (address pool, uint256 liquidity) {
        
        // 1. 检查池是否已存在
        pool = factory.getPair(address(ableToken), pairedToken);
        require(pool == address(0), "Pool already exists");
        
        // 2. 从 Treasury 获取资金
        _transferFromTreasury(address(ableToken), ableAmount);
        _transferFromTreasury(pairedToken, pairedAmount);
        
        // 3. 授权 Router
        ableToken.safeApprove(address(router), ableAmount);
        IERC20(pairedToken).safeApprove(address(router), pairedAmount);
        
        // 4. 添加流动性 (会自动创建池)
        (uint256 ableUsed, uint256 pairedUsed, uint256 liquidityMinted) = 
            router.addLiquidity(
                address(ableToken),
                pairedToken,
                ableAmount,
                pairedAmount,
                _calculateMinAmount(ableAmount),
                _calculateMinAmount(pairedAmount),
                address(this),
                _getDeadline()
            );
        
        // 5. 返还多余资金到 Treasury
        _returnExcessToTreasury(address(ableToken), ableAmount, ableUsed);
        _returnExcessToTreasury(pairedToken, pairedAmount, pairedUsed);
        
        // 6. 获取池地址并记录
        pool = factory.getPair(address(ableToken), pairedToken);
        lpBalances[pool] = liquidityMinted;
        
        emit LiquidityAdded(pool, ableUsed, pairedUsed, liquidityMinted);
        
        return (pool, liquidityMinted);
    }
    
    // 添加流动性到现有池
    function addLiquidity(
        address pool,
        uint256 ableAmount,
        uint256 pairedAmount
    ) external onlyRole(OPERATOR_ROLE) nonReentrant whenNotPaused 
      returns (uint256 liquidity) {
        
        // 1. 验证池存在
        require(pool != address(0), "Invalid pool");
        IUniswapV2Pair pair = IUniswapV2Pair(pool);
        
        address token0 = pair.token0();
        address token1 = pair.token1();
        address pairedToken = token0 == address(ableToken) ? token1 : token0;
        
        // 2. 从 Treasury 获取资金
        _transferFromTreasury(address(ableToken), ableAmount);
        _transferFromTreasury(pairedToken, pairedAmount);
        
        // 3. 授权 Router
        ableToken.safeApprove(address(router), ableAmount);
        IERC20(pairedToken).safeApprove(address(router), pairedAmount);
        
        // 4. 添加流动性
        (uint256 ableUsed, uint256 pairedUsed, uint256 liquidityMinted) = 
            router.addLiquidity(
                address(ableToken),
                pairedToken,
                ableAmount,
                pairedAmount,
                _calculateMinAmount(ableAmount),
                _calculateMinAmount(pairedAmount),
                address(this),
                _getDeadline()
            );
        
        // 5. 返还多余资金
        _returnExcessToTreasury(address(ableToken), ableAmount, ableUsed);
        _returnExcessToTreasury(pairedToken, pairedAmount, pairedUsed);
        
        // 6. 更新记录
        lpBalances[pool] += liquidityMinted;
        
        emit LiquidityAdded(pool, ableUsed, pairedUsed, liquidityMinted);
        
        return liquidityMinted;
    }
    
    // 移除流动性
    function removeLiquidity(
        address pool,
        uint256 liquidity
    ) external onlyRole(OPERATOR_ROLE) nonReentrant 
      returns (uint256 ableAmount, uint256 pairedAmount) {
        
        // 1. 验证
        require(liquidity > 0, "Invalid liquidity");
        require(lpBalances[pool] >= liquidity, "Insufficient LP balance");
        
        IUniswapV2Pair pair = IUniswapV2Pair(pool);
        address token0 = pair.token0();
        address token1 = pair.token1();
        
        // 2. 授权 Router
        IERC20(pool).safeApprove(address(router), liquidity);
        
        // 3. 移除流动性
        (uint256 amount0, uint256 amount1) = router.removeLiquidity(
            token0,
            token1,
            liquidity,
            0,  // 接受任何数量 (已在外部检查)
            0,
            treasury,  // 资金直接返还到 Treasury
            _getDeadline()
        );
        
        // 4. 更新记录
        lpBalances[pool] -= liquidity;
        
        // 5. 确定哪个是 ABLE
        (ableAmount, pairedAmount) = token0 == address(ableToken) 
            ? (amount0, amount1) 
            : (amount1, amount0);
        
        emit LiquidityRemoved(pool, liquidity, ableAmount, pairedAmount);
        
        return (ableAmount, pairedAmount);
    }
    
    // 查询 LP 价值
    function getLPValue(address pool) external view returns (
        uint256 ableAmount,
        uint256 pairedAmount
    ) {
        IUniswapV2Pair pair = IUniswapV2Pair(pool);
        
        uint256 lpBalance = lpBalances[pool];
        if (lpBalance == 0) return (0, 0);
        
        uint256 totalSupply = pair.totalSupply();
        (uint112 reserve0, uint112 reserve1,) = pair.getReserves();
        
        address token0 = pair.token0();
        
        uint256 amount0 = uint256(reserve0) * lpBalance / totalSupply;
        uint256 amount1 = uint256(reserve1) * lpBalance / totalSupply;
        
        (ableAmount, pairedAmount) = token0 == address(ableToken)
            ? (amount0, amount1)
            : (amount1, amount0);
    }
    
    // 内部函数: 从 Treasury 转账
    function _transferFromTreasury(address token, uint256 amount) internal {
        // 假设 Treasury 有 withdrawTo 函数
        // 实际实现需要根据 Treasury 合约接口调整
        IERC20(token).safeTransferFrom(treasury, address(this), amount);
    }
    
    // 内部函数: 返还多余资金到 Treasury
    function _returnExcessToTreasury(
        address token,
        uint256 desired,
        uint256 used
    ) internal {
        if (desired > used) {
            IERC20(token).safeTransfer(treasury, desired - used);
        }
    }
    
    // 内部函数: 计算最小数量 (考虑滑点)
    function _calculateMinAmount(uint256 amount) internal pure returns (uint256) {
        return amount * (10000 - MAX_SLIPPAGE) / 10000;
    }
    
    // 内部函数: 获取截止时间
    function _getDeadline() internal view returns (uint256) {
        return block.timestamp + DEADLINE_DURATION;
    }
    
    // 紧急暂停
    function pause() external onlyRole(EMERGENCY_ROLE) {
        _pause();
    }
    
    // 恢复
    function unpause() external onlyRole(EMERGENCY_ROLE) {
        _unpause();
    }
}
```

---

## 2. AbleLiquidityMining 实现

### 2.1 合约概述

**功能**: 管理 ABLE 代币流动性挖矿激励

**特性**:
- 单池或多池支持
- 动态奖励率
- 紧急提现

### 2.2 核心实现 (伪代码)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract AbleLiquidityMining is AccessControl, ReentrancyGuard, Pausable {
    using SafeERC20 for IERC20;
    
    bytes32 public constant OPERATOR_ROLE = keccak256("OPERATOR_ROLE");
    
    // 代币
    IERC20 public immutable ableToken;  // 奖励代币
    IERC20 public immutable lpToken;    // LP Token
    
    // 奖励参数
    uint256 public rewardRate;  // 每秒奖励数量
    uint256 public lastUpdateTime;
    uint256 public rewardPerTokenStored;
    
    // 质押信息
    uint256 public totalStaked;
    mapping(address => StakeInfo) public stakes;
    
    struct StakeInfo {
        uint256 amount;
        uint256 rewardPerTokenPaid;
        uint256 rewards;
    }
    
    // 事件
    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount);
    event RewardsClaimed(address indexed user, uint256 amount);
    event RewardRateUpdated(uint256 newRate);
    
    constructor(
        address _ableToken,
        address _lpToken,
        uint256 _initialRewardRate,
        address _admin
    ) {
        require(_ableToken != address(0), "Invalid ABLE token");
        require(_lpToken != address(0), "Invalid LP token");
        
        ableToken = IERC20(_ableToken);
        lpToken = IERC20(_lpToken);
        rewardRate = _initialRewardRate;
        lastUpdateTime = block.timestamp;
        
        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
        _grantRole(OPERATOR_ROLE, _admin);
    }
    
    // 质押 LP Token
    function stake(uint256 amount) external nonReentrant whenNotPaused updateReward(msg.sender) {
        require(amount > 0, "Cannot stake 0");
        
        totalStaked += amount;
        stakes[msg.sender].amount += amount;
        
        lpToken.safeTransferFrom(msg.sender, address(this), amount);
        
        emit Staked(msg.sender, amount);
    }
    
    // 解除质押
    function unstake(uint256 amount) external nonReentrant updateReward(msg.sender) {
        require(amount > 0, "Cannot unstake 0");
        require(stakes[msg.sender].amount >= amount, "Insufficient stake");
        
        totalStaked -= amount;
        stakes[msg.sender].amount -= amount;
        
        lpToken.safeTransfer(msg.sender, amount);
        
        emit Unstaked(msg.sender, amount);
    }
    
    // 领取奖励
    function claimRewards() external nonReentrant updateReward(msg.sender) {
        uint256 reward = stakes[msg.sender].rewards;
        if (reward > 0) {
            stakes[msg.sender].rewards = 0;
            ableToken.safeTransfer(msg.sender, reward);
            emit RewardsClaimed(msg.sender, reward);
        }
    }
    
    // 查询待领取奖励
    function pendingRewards(address user) external view returns (uint256) {
        uint256 rewardPerToken = _rewardPerToken();
        StakeInfo memory userStake = stakes[user];
        
        return userStake.amount * (rewardPerToken - userStake.rewardPerTokenPaid) / 1e18 
            + userStake.rewards;
    }
    
    // 更新奖励率
    function setRewardRate(uint256 newRate) external onlyRole(OPERATOR_ROLE) updateReward(address(0)) {
        rewardRate = newRate;
        emit RewardRateUpdated(newRate);
    }
    
    // 计算每个 Token 的奖励
    function _rewardPerToken() internal view returns (uint256) {
        if (totalStaked == 0) {
            return rewardPerTokenStored;
        }
        
        return rewardPerTokenStored + 
            (block.timestamp - lastUpdateTime) * rewardRate * 1e18 / totalStaked;
    }
    
    // 修饰器: 更新奖励
    modifier updateReward(address account) {
        rewardPerTokenStored = _rewardPerToken();
        lastUpdateTime = block.timestamp;
        
        if (account != address(0)) {
            stakes[account].rewards = _earned(account);
            stakes[account].rewardPerTokenPaid = rewardPerTokenStored;
        }
        _;
    }
    
    // 计算已赚取奖励
    function _earned(address account) internal view returns (uint256) {
        StakeInfo memory userStake = stakes[account];
        return userStake.amount * (_rewardPerToken() - userStake.rewardPerTokenPaid) / 1e18 
            + userStake.rewards;
    }
    
    // 紧急提现 (仅管理员)
    function emergencyWithdraw() external onlyRole(DEFAULT_ADMIN_ROLE) {
        uint256 amount = stakes[msg.sender].amount;
        require(amount > 0, "No stake");
        
        totalStaked -= amount;
        stakes[msg.sender].amount = 0;
        stakes[msg.sender].rewards = 0;
        
        lpToken.safeTransfer(msg.sender, amount);
    }
    
    // 暂停/恢复
    function pause() external onlyRole(OPERATOR_ROLE) {
        _pause();
    }
    
    function unpause() external onlyRole(OPERATOR_ROLE) {
        _unpause();
    }
}
```

---

**最后更新**: 2025-10-18 14:52:14 CST

**注意**: 以上代码为伪代码示例,实际部署前需要:
1. 完整的单元测试
2. 安全审计
3. Gas 优化
4. 与 Treasury 合约的实际接口对接

