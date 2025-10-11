# RWA-HUSD SaaS平台技术栈详解

**文档版本**: v1.0  
**创建时间**: 2025-10-11 09:10:00 CST  
**文档类型**: 技术栈详解  

---

## 📑 目录

1. [前端技术栈](#1-前端技术栈)
2. [后端技术栈](#2-后端技术栈)
3. [区块链集成](#3-区块链集成)
4. [安全方案](#4-安全方案)
5. [部署方案](#5-部署方案)

---

## 1. 前端技术栈

### 1.1 核心框架

#### React 18.x
**选型理由**:
- 生态系统成熟，社区活跃
- 组件化开发，代码复用性高
- Virtual DOM，性能优秀
- Hooks API，状态管理简洁

**替代方案**:
- Vue 3: 更简单易学，但企业级生态不如React
- Angular: 功能全面，但学习曲线陡峭

#### Next.js 14.x
**选型理由**:
- 服务端渲染（SSR），SEO友好
- 静态站点生成（SSG），性能优秀
- API路由，前后端一体化
- 自动代码分割，加载速度快

**替代方案**:
- Nuxt.js: Vue生态的SSR框架
- Remix: 新兴框架，但生态不够成熟

#### TypeScript 5.x
**选型理由**:
- 类型安全，减少运行时错误
- 代码提示，开发效率高
- 重构友好，维护成本低
- 与React生态完美集成

**替代方案**:
- JavaScript: 灵活性高，但缺少类型检查

### 1.2 UI框架

#### TailwindCSS 3.x
**选型理由**:
- 原子化CSS，快速开发
- 响应式设计，移动端友好
- 可定制性强，符合设计系统
- 生产环境体积小（PurgeCSS）

**替代方案**:
- Material-UI: 组件丰富，但定制性差
- Ant Design: 企业级UI，但体积较大

#### shadcn/ui
**选型理由**:
- 基于Radix UI，可访问性好
- 现代设计风格，符合Web3审美
- 可定制性强，可直接修改源码
- 与TailwindCSS完美集成

**替代方案**:
- Chakra UI: 组件丰富，但性能不如shadcn/ui
- Headless UI: 无样式组件，需要自己写CSS

### 1.3 Web3集成

#### Wagmi 2.x
**选型理由**:
- React Hooks API，易于使用
- TypeScript支持，类型安全
- 自动处理钱包连接、签名、交易
- 与Viem完美集成

**替代方案**:
- Web3-React: 功能强大，但API复杂
- useDApp: 简单易用，但功能有限

#### Viem 2.x
**选型理由**:
- 轻量级（体积小于Ethers.js）
- TypeScript原生支持
- 性能优秀（比Ethers.js快2-3倍）
- 模块化设计，按需引入

**替代方案**:
- Ethers.js: 更成熟，但体积较大
- Web3.js: 功能全面，但API设计老旧

### 1.4 状态管理

#### Zustand 4.x
**选型理由**:
- 轻量级（体积小于Redux）
- API简单，学习曲线平缓
- 性能优秀，无不必要的重渲染
- TypeScript支持良好

**替代方案**:
- Redux Toolkit: 功能强大，但复杂度高
- Jotai: 原子化状态，但生态不够成熟

#### React Query 5.x
**选型理由**:
- 自动缓存管理
- 自动重试和错误处理
- 乐观更新，用户体验好
- 与Wagmi完美集成

**替代方案**:
- SWR: 简单易用，但功能不如React Query
- Apollo Client: GraphQL专用，不适合REST API

---

## 2. 后端技术栈

### 2.1 核心框架

#### NestJS 10.x
**选型理由**:
- 企业级框架，架构清晰
- 模块化设计，代码组织良好
- 依赖注入，测试友好
- TypeScript原生支持

**替代方案**:
- Express.js: 简单灵活，但缺少企业级特性
- Fastify: 性能优秀，但生态不如NestJS

### 2.2 数据库

#### PostgreSQL 15.x
**选型理由**:
- ACID保证，数据一致性强
- JSON/JSONB支持，灵活性高
- 性能优秀，支持复杂查询
- 开源免费，社区活跃

**替代方案**:
- MySQL: 更流行，但JSON支持不如PostgreSQL
- MongoDB: 灵活性高，但缺少ACID保证

#### Prisma 5.x
**选型理由**:
- TypeScript原生支持，类型安全
- 自动生成类型定义
- 迁移管理友好
- 查询性能优秀

**替代方案**:
- TypeORM: 功能更强，但性能不如Prisma
- Sequelize: 成熟稳定，但TypeScript支持不佳

### 2.3 缓存和队列

#### Redis 7.x
**选型理由**:
- 高性能，支持多种数据结构
- 持久化支持，数据不丢失
- 分布式锁，并发控制
- 发布订阅，消息队列

**替代方案**:
- Memcached: 更简单，但功能有限
- RabbitMQ: 消息队列专用，但复杂度高

#### Bull 4.x
**选型理由**:
- 基于Redis，性能优秀
- 支持延迟任务和定时任务
- 监控友好，可视化界面
- 可靠性高，支持重试

**替代方案**:
- BullMQ: Bull的升级版，但生态不够成熟
- Agenda: 基于MongoDB，但性能不如Bull

---

## 3. 区块链集成

### 3.1 合约交互

#### Ethers.js 6.x
**选型理由**:
- 成熟稳定，文档完善
- 社区活跃，问题解决快
- 功能全面，支持所有EVM链
- TypeScript支持良好

**替代方案**:
- Viem: 更轻量，但后端使用Ethers.js更成熟
- Web3.js: 功能全面，但API设计老旧

### 3.2 事件索引

#### The Graph
**选型理由**:
- 高性能，支持GraphQL查询
- 实时更新，延迟低
- 去中心化，可靠性高
- 支持多链

**替代方案**:
- 自建索引服务: 灵活性高，但开发成本高
- Moralis: 功能全面，但中心化

### 3.3 节点服务

#### Alchemy
**选型理由**:
- 可靠性高，99.9% SLA
- API丰富，功能全面
- 监控友好，实时告警
- 免费额度充足

**替代方案**:
- Infura: 更成熟，但免费额度较少
- QuickNode: 性能优秀，但价格较高

---

## 4. 安全方案

### 4.1 认证和授权

#### JWT + Refresh Token
**实现方案**:
```typescript
// Access Token: 15分钟过期
// Refresh Token: 7天过期
// 存储: Access Token存localStorage, Refresh Token存HttpOnly Cookie

interface JWTPayload {
  userId: string;
  role: 'admin' | 'institution' | 'investor';
  permissions: string[];
}

// 认证流程
1. 用户登录 → 返回Access Token + Refresh Token
2. 请求API → 携带Access Token
3. Access Token过期 → 使用Refresh Token刷新
4. Refresh Token过期 → 重新登录
```

#### RBAC（基于角色的访问控制）
**角色定义**:
- **Super Admin**: 平台超级管理员
- **Institution Admin**: 机构管理员
- **Institution Operator**: 机构操作员
- **Investor**: 投资者
- **Property Owner**: 房产所有者

**权限矩阵**:
| 功能 | Super Admin | Institution Admin | Institution Operator | Investor | Property Owner |
|------|-------------|-------------------|---------------------|----------|----------------|
| 创建机构 | ✅ | ❌ | ❌ | ❌ | ❌ |
| 质押ABLE | ❌ | ✅ | ❌ | ❌ | ❌ |
| 创建资产 | ❌ | ✅ | ✅ | ❌ | ❌ |
| 购买代币 | ❌ | ❌ | ❌ | ✅ | ❌ |
| 充值租金 | ❌ | ❌ | ❌ | ❌ | ✅ |

### 4.2 数据加密

#### AES-256加密
**加密范围**:
- 用户敏感信息（身份证号、护照号）
- KYC文档
- 私钥（如果托管）

**加密方案**:
```typescript
// 使用crypto-js库
import CryptoJS from 'crypto-js';

// 加密
const encrypted = CryptoJS.AES.encrypt(
  plaintext,
  process.env.ENCRYPTION_KEY
).toString();

// 解密
const decrypted = CryptoJS.AES.decrypt(
  encrypted,
  process.env.ENCRYPTION_KEY
).toString(CryptoJS.enc.Utf8);
```

### 4.3 API安全

#### Rate Limiting
**限流策略**:
- 登录接口: 5次/分钟
- 注册接口: 3次/小时
- 查询接口: 100次/分钟
- 交易接口: 10次/分钟

**实现方案**:
```typescript
// 使用@nestjs/throttler
import { ThrottlerModule } from '@nestjs/throttler';

ThrottlerModule.forRoot({
  ttl: 60,
  limit: 100,
});
```

#### CORS + CSP
**CORS配置**:
```typescript
app.enableCors({
  origin: ['https://app.rwa-husd.com'],
  credentials: true,
});
```

**CSP配置**:
```typescript
helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https:'],
  },
});
```

---

## 5. 部署方案

### 5.1 容器化

#### Docker
**Dockerfile示例**:
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "run", "start:prod"]
```

#### Docker Compose
**docker-compose.yml示例**:
```yaml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
  backend:
    build: ./backend
    ports:
      - "4000:4000"
    depends_on:
      - postgres
      - redis
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: rwa_husd
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
  redis:
    image: redis:7-alpine
```

### 5.2 Kubernetes

#### Deployment配置
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: rwa-husd/backend:latest
        ports:
        - containerPort: 4000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
```

### 5.3 CI/CD

#### GitHub Actions
```yaml
name: Deploy to Production
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker Image
        run: docker build -t rwa-husd/backend:${{ github.sha }} .
      - name: Push to Registry
        run: docker push rwa-husd/backend:${{ github.sha }}
      - name: Deploy to K8s
        run: kubectl set image deployment/backend backend=rwa-husd/backend:${{ github.sha }}
```

---

**文档维护**: RWA-HUSD 技术团队  
**联系方式**: tech@rwa-husd.com  
**最后更新**: 2025-10-11 09:10:00 CST

