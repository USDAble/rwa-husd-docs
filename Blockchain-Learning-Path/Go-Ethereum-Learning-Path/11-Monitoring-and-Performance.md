# 11-Monitoring-and-Performance.md - 监控与性能优化

**难度**: ⭐⭐⭐⭐☆  
**预估学习时间**: 15-18 小时  
**前置知识**: Geth 节点管理、Linux 系统管理、监控工具基础

---

## 学习目标

通过本文档的学习,你将能够:

1. 理解 Geth 监控架构和指标导出机制
2. 掌握 Prometheus 集成配置和查询语法
3. 学会搭建 InfluxDB + Grafana 监控系统
4. 理解关键性能指标的含义和分析方法
5. 掌握 Geth 性能优化的最佳实践

---

## 1. Geth 监控架构

### 1.1 指标导出机制

Geth 内置了完整的指标收集系统,支持多种导出方式:

```
┌─────────────────────────────────────────┐
│          Geth 监控架构                  │
├─────────────────────────────────────────┤
│  Geth 内部指标收集                      │
│  ┌──────────┐  ┌──────────┐  ┌────────┐│
│  │ CPU/内存  │  │ 区块处理  │  │ P2P网络││
│  └──────────┘  └──────────┘  └────────┘│
├─────────────────────────────────────────┤
│  指标导出接口                           │
│  ┌──────────────┐  ┌──────────────┐   │
│  │ Prometheus   │  │ InfluxDB     │   │
│  │ (Pull 模式)  │  │ (Push 模式)  │   │
│  └──────────────┘  └──────────────┘   │
├─────────────────────────────────────────┤
│  可视化层                               │
│  ┌──────────────┐  ┌──────────────┐   │
│  │ Grafana      │  │ Prometheus UI│   │
│  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────┘
```

### 1.2 Prometheus vs InfluxDB

| 特性 | Prometheus | InfluxDB |
|------|-----------|----------|
| **数据模型** | 时间序列 (Pull) | 时间序列 (Push) |
| **查询语言** | PromQL | InfluxQL |
| **存储** | 本地时序数据库 | 专用时序数据库 |
| **集成难度** | 简单 (HTTP 端点) | 中等 (需配置推送) |
| **适用场景** | 轻量级监控 | 长期数据存储 |

**推荐方案**:
- **开发/测试**: Prometheus (快速搭建)
- **生产环境**: InfluxDB + Grafana (更强大的数据分析)

---

## 2. Prometheus 集成

### 2.1 配置 Geth 指标导出

**启动 Geth 并启用 Prometheus 端点**:

```bash
geth \
  --metrics \
  --metrics.addr 127.0.0.1 \
  --metrics.port 6060 \
  --http \
  --http.api eth,web3,net
```

**验证指标端点**:

```bash
curl http://127.0.0.1:6060/debug/metrics/prometheus
```

**输出示例**:

```
# HELP system_cpu_sysload System CPU load
# TYPE system_cpu_sysload gauge
system_cpu_sysload 0.45

# HELP chain_head_block Current head block number
# TYPE chain_head_block gauge
chain_head_block 18500000

# HELP txpool_pending Pending transactions in pool
# TYPE txpool_pending gauge
txpool_pending 125
```

### 2.2 Prometheus 配置

**创建配置目录**:

```bash
mkdir -p ~/prometheus
cd ~/prometheus
```

**创建 `prometheus.yml`**:

```yaml
global:
  scrape_interval: 15s       # 每 15 秒抓取一次指标
  evaluation_interval: 15s   # 每 15 秒评估一次规则

# 加载规则文件
rule_files:
  - 'record.geth.rules.yml'

# 抓取配置
scrape_configs:
  - job_name: 'go-ethereum'
    scrape_interval: 10s
    metrics_path: /debug/metrics/prometheus
    static_configs:
      - targets:
          - '127.0.0.1:6060'
        labels:
          chain: ethereum
          instance: mainnet-node-1
```

**创建记录规则 `record.geth.rules.yml`**:

```yaml
groups:
  - name: geth_recording_rules
    interval: 10s
    rules:
      # 计算每秒处理的区块数
      - record: geth:blocks_per_second
        expr: rate(chain_head_block[1m])
      
      # 计算平均 Gas 使用率
      - record: geth:avg_gas_usage
        expr: avg_over_time(txpool_pending[5m])
```

### 2.3 启动 Prometheus

**使用 Docker 启动**:

```bash
docker run \
  -p 9090:9090 \
  -v ~/prometheus:/etc/prometheus \
  prom/prometheus:latest
```

**访问 Prometheus UI**: http://localhost:9090

**常用 PromQL 查询**:

```promql
# 当前区块高度
chain_head_block

# CPU 使用率
system_cpu_sysload

# 每秒处理交易数
rate(txpool_pending[1m])

# 内存使用量 (MB)
system_memory_used / 1024 / 1024
```

---

## 3. InfluxDB + Grafana 方案

### 3.1 InfluxDB 安装

**Debian/Ubuntu 安装**:

```bash
# 添加 InfluxData 仓库
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | \
  sudo tee /etc/apt/sources.list.d/influxdb.list

# 安装 InfluxDB
sudo apt update
sudo apt install influxdb influxdb-client -y

# 启动服务
sudo systemctl enable influxdb
sudo systemctl start influxdb
```

### 3.2 配置 InfluxDB

**创建数据库和用户**:

```bash
# 进入 InfluxDB Shell
influx

# 创建数据库
CREATE DATABASE geth

# 创建用户
CREATE USER geth WITH PASSWORD 'your_secure_password'

# 验证
SHOW DATABASES
SHOW USERS

# 退出
exit
```

### 3.3 配置 Geth 推送数据

**启动 Geth 并启用 InfluxDB 推送**:

```bash
geth \
  --metrics \
  --metrics.influxdb \
  --metrics.influxdb.endpoint "http://127.0.0.1:8086" \
  --metrics.influxdb.database "geth" \
  --metrics.influxdb.username "geth" \
  --metrics.influxdb.password "your_secure_password" \
  --metrics.influxdb.tags "host=mainnet-node-1"
```

**验证数据写入**:

```bash
influx -username 'geth' -password 'your_secure_password'

# 切换到 geth 数据库
USE geth

# 查看所有测量值 (measurements)
SHOW MEASUREMENTS

# 查询最新数据
SELECT * FROM "geth.chain/head/block.gauge" ORDER BY time DESC LIMIT 10
```

### 3.4 Grafana 配置

**安装 Grafana**:

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana -y

# 启动服务
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

**访问 Grafana**: http://localhost:3000 (默认账号: admin/admin)

**添加 InfluxDB 数据源**:

1. 进入 Configuration → Data Sources
2. 点击 "Add data source"
3. 选择 "InfluxDB"
4. 配置参数:

```
Name: InfluxDB-Geth
Query Language: InfluxQL
HTTP:
  URL: http://localhost:8086
  Access: Server (default)
InfluxDB Details:
  Database: geth
  User: geth
  Password: your_secure_password
  HTTP Method: GET
```

5. 点击 "Save & Test"

---

## 4. 关键性能指标详解

### 4.1 系统资源指标

**CPU 指标**:

```
system/cpu/sysload.gauge      # 系统 CPU 负载
system/cpu/syswait.gauge      # I/O 等待时间
system/cpu/procload.gauge     # Geth 进程 CPU 使用率
```

**Grafana 面板配置**:

```json
{
  "targets": [
    {
      "measurement": "geth.system/cpu/sysload.gauge",
      "alias": "System Load"
    },
    {
      "measurement": "geth.system/cpu/procload.gauge",
      "alias": "Geth Process"
    }
  ]
}
```

**内存指标**:

```
system/memory/used.gauge      # 已使用内存
system/memory/available.gauge # 可用内存
system/memory/held.gauge      # Geth 持有内存
```

**磁盘 I/O 指标**:

```
system/disk/readbytes.meter   # 磁盘读取速率
system/disk/writebytes.meter  # 磁盘写入速率
system/disk/readcount.meter   # 读取操作次数
system/disk/writecount.meter  # 写入操作次数
```

### 4.2 区块处理性能

**区块同步指标**:

```
chain/head/block.gauge        # 当前区块高度
chain/head/header.gauge       # 当前区块头高度
eth/downloader/blocks.meter   # 下载区块速率
```

**区块处理时间** (单位: 微秒):

```
chain/execution.timer         # 交易执行时间
chain/validation.timer        # 区块验证时间
chain/commit.timer            # 状态提交时间
chain/account/reads.timer     # 账户读取时间
chain/account/updates.timer   # 账户更新时间
chain/storage/reads.timer     # 存储读取时间
chain/storage/updates.timer   # 存储更新时间
```

**Grafana 查询示例**:

```influxql
SELECT mean("value") 
FROM "geth.chain/execution.timer" 
WHERE time > now() - 1h 
GROUP BY time(1m)
```

### 4.3 P2P 网络指标

**对等节点指标**:

```
p2p/peers.gauge               # 当前连接的对等节点数
p2p/ingress.meter             # 入站流量速率
p2p/egress.meter              # 出站流量速率
```

**交易传播指标**:

```
txpool/pending.gauge          # 待处理交易数
txpool/queued.gauge           # 队列中的交易数
eth/fetcher/announces.meter   # 交易公告速率
eth/fetcher/broadcasts.meter  # 交易广播速率
```

### 4.4 数据库性能

**LevelDB 指标**:

```
chaindata/db/comptime.meter   # 压缩时间
chaindata/db/compread.meter   # 压缩读取量
chaindata/db/compwrite.meter  # 压缩写入量
chaindata/db/diskread.meter   # 磁盘读取速率
chaindata/db/diskwrite.meter  # 磁盘写入速率
```

**Freezer (古老数据) 指标**:

```
ancient/read.meter            # 古老数据读取速率
ancient/write.meter           # 古老数据写入速率
ancient/size.gauge            # 古老数据库大小
```

---

## 5. 性能优化实践

### 5.1 数据库调优

**LevelDB 缓存配置**:

```bash
geth \
  --cache 4096 \              # 总缓存大小 (MB)
  --cache.database 50 \       # 数据库缓存占比 (%)
  --cache.trie 25 \           # Trie 缓存占比 (%)
  --cache.gc 25               # GC 缓存占比 (%)
```

**推荐配置** (基于硬件):

| 内存大小 | --cache | --cache.database | --cache.trie |
|---------|---------|-----------------|--------------|
| 8 GB    | 2048    | 50%             | 25%          |
| 16 GB   | 4096    | 50%             | 25%          |
| 32 GB   | 8192    | 50%             | 25%          |
| 64 GB   | 16384   | 50%             | 25%          |

**Freezer 优化**:

```bash
# 将 Freezer 数据移到独立的 SSD
geth --datadir /path/to/data --datadir.ancient /path/to/ssd/ancient
```

### 5.2 同步模式优化

**Snap Sync (推荐)**:

```bash
geth --syncmode snap  # 最快的同步模式
```

**Full Sync**:

```bash
geth --syncmode full  # 完整验证,速度较慢
```

**Archive Node**:

```bash
geth --syncmode full --gcmode archive  # 保留所有历史状态
```

### 5.3 网络优化

**对等节点配置**:

```bash
geth \
  --maxpeers 50 \             # 最大对等节点数
  --maxpendpeers 10 \         # 最大待连接节点数
  --netrestrict "10.0.0.0/8"  # 限制网络范围
```

**带宽限制**:

```bash
# 限制入站/出站流量 (KB/s)
geth --metrics.expensive  # 启用详细网络指标
```

### 5.4 硬件配置建议

**最低配置** (Full Node):
- CPU: 4 核
- 内存: 16 GB
- 存储: 1 TB NVMe SSD
- 网络: 25 Mbps

**推荐配置** (Archive Node):
- CPU: 8 核+
- 内存: 32 GB+
- 存储: 4 TB+ NVMe SSD (RAID 0)
- 网络: 100 Mbps+

**存储性能要求**:
- **IOPS**: > 10,000 (随机读写)
- **吞吐量**: > 500 MB/s (顺序读写)
- **延迟**: < 1ms

---

## 总结

### 核心要点

1. **Geth 监控** 支持 Prometheus 和 InfluxDB 两种方案
2. **Prometheus** 适合快速搭建,InfluxDB 适合生产环境
3. **关键指标** 包括 CPU、内存、区块处理、P2P 网络和数据库性能
4. **性能优化** 需要从数据库、同步模式、网络和硬件多方面入手
5. **Grafana** 提供了强大的可视化和告警能力

### 下一步学习

- **构建自定义客户端**: 从源码定制 Geth 客户端
- **高可用架构**: 搭建 Geth 集群和负载均衡
- **安全加固**: 节点安全配置和防护策略

### 参考资源

- [Geth 官方文档 - Monitoring](https://geth.ethereum.org/docs/monitoring)
- [Prometheus 官方文档](https://prometheus.io/docs/)
- [Grafana 官方文档](https://grafana.com/docs/)
- [InfluxDB 官方文档](https://docs.influxdata.com/)

