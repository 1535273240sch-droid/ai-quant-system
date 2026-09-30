# Enterprise AI Quant System (企业级 AI 量化交易平台)

<div align="center">

![Version](https://img.shields.io/badge/Version-v0.2.0-blue?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-Framework-009688?style=flat-square&logo=fastapi)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)
![Redis](https://img.shields.io/badge/Redis-7.0-DC382D?style=flat-square&logo=redis)
![Docker](https://img.shields.io/badge/Docker-Compose%20Ready-2496ED?style=flat-square&logo=docker)
![Maturity](https://img.shields.io/badge/Maturity-Enterprise%20Grade%20~90%25-brightgreen?style=flat-square)

<p align="center">
  <b>面向加密货币与数字资产的企业级全自动化 AI 量化交易系统</b><br>
  涵盖「因子挖掘 · 策略回测 · 模拟试运行 (Paper Trading) · 实盘交易 · 全链路多层风控 · 审批与对账治理」的完整闭环。
</p>

</div>

---

## 🌟 核心体系与模块说明

系统严格按金融级标准设计，提供高可靠的自动化交易闭环：

| 核心模块 | 功能与机制说明 |
|:---|:---|
| **订单全生命周期管理** | 严格遵循 `订单预检 → 路由下单 → 撮合成交回报 → 实时持仓统计 → 每日一致性对账` |
| **多层风控体系 (Risk Guard)** | `max_notional` 单笔/全局限额、高频重复单防刷校验、最小下单量限制、价格偏离安全带保护、**紧急 Kill Switch (一键熔断)** |
| **可插拔策略引擎** | 因子研究引擎、向量化/事件驱动回测、支持动态参数与注册工厂模式 |
| **多级审批治理机制** | 关键指令与审批单生命周期管理（创建/时效过期/审计投影）+ 触发熔断后的阶梯式复核恢复审批流 |
| **自动化持仓对账** | 系统内部记账与交易所远程账单自动比对，差异自动标记并归档审计记录 |
| **全交易所适配层** | 统一数据源与执行抽象接口（支持 Binance、Coinbase、OKX）+ 故障自动降级切换通道 |
| **模拟交易环境** | 内置高仿真的 Paper Trading 虚拟撮合网关，与实盘运行 100% 相同的数据流水线 |

---

## 🛠️ 全栈技术架构

- **后端核心**：Python 3.12 · FastAPI · SQLAlchemy 2.0 (异步 ORM) · Alembic (数据库迁移) · Celery 异步任务 · Pydantic v2
- **前端控制台**：TypeScript · Vite · Vue/React SPA（生产环境由 Nginx 静态托管）
- **数据基础设施**：PostgreSQL 16 (持久化关系存储) · Redis 7 (实时事件流与分布式缓存) · Docker Compose · Nginx · systemd
- **企业级可观测性**：`structlog` JSON 结构化日志 · OpenTelemetry 分布式追踪 · Prometheus `/metrics` 指标暴露 · Sentry 异常捕获

---

## 📂 仓库目录结构

```
.
├── backend/                      # FastAPI 高性能核心后端
│   ├── app/
│   │   ├── api/                  # RESTful API 路由层与 WebSocket 端点
│   │   ├── services/             # 核心领域服务（订单、风控、对账、多级审批等）
│   │   ├── adapters/             # 交易所标准化适配器（Binance / Coinbase / OKX / Paper）
│   │   ├── strategies/           # 策略研究与执行引擎
│   │   ├── db/                   # SQLAlchemy ORM 实体模型与数据库连接池
│   │   ├── events/               # 领域事件流处理 (基于 Redis Stream)
│   │   ├── observability/        # 监控指标、结构化日志与 APM 追踪
│   │   └── main.py               # 应用启动总入口
│   ├── alembic/                  # 数据库表结构版本迁移脚本
│   ├── tests/                    # pytest 单元测试与集成测试套件
│   ├── Dockerfile                # 多阶段安全构建 Dockerfile (非 root 运行)
│   ├── docker-compose.yml        # 后端全容器化编排方案
│   ├── requirements.txt          # Python 核心依赖定义
│   └── .env.example              # 生产环境变量模板
├── frontend/                     # Vite + TypeScript 企业级运营控制台
├── deploy/
│   ├── nginx.conf                # Nginx 高性能代理与静态托管配置
│   ├── quant.service             # Linux systemd 常驻守护进程单元
│   └── docker-compose.infra.yml  # 基础设施服务（PostgreSQL 16 + Redis 7）
└── PROJECT_STATUS.md             # 详细功能成熟度矩阵与交付备忘
```

---

## 🚀 生产部署手册

生产环境推荐采用 **「Docker 运行数据底座 + systemd 守护核心进程 + Nginx 反向代理前端」** 的高稳定混合架构。

### 1. 架构拓扑
```
公网客户端请求 → Nginx (:80 / :443)
                  ├─ /                                → /var/www/enterprise-ai-quant（前端静态资源）
                  └─ /api, /health, /ready, /metrics   → 127.0.0.1:8000
                                                             ↑
                                                      uvicorn 进程（systemd: quant.service）
                                                             ↓
                                        PostgreSQL (:5432) · Redis (:6379)
                                        (Docker 容器，内网安全隔离，仅绑定 127.0.0.1)
```

### 2. 环境先决条件
- **操作系统**：Ubuntu 22.04 LTS / 24.04 LTS+
- **基础依赖**：Python 3.12+、Docker、Docker Compose、Nginx
- **推荐硬件**：>= 2 核 CPU / 4GB 内存 / 40GB SSD

### 3. 启动数据底座（PostgreSQL + Redis）
```bash
cd deploy
docker compose -f docker-compose.infra.yml up -d
docker compose -f docker-compose.infra.yml ps   # 检查并确认状态为 healthy
```
> [!NOTE]
> 容器默认仅绑定 `127.0.0.1` 本地回环接口，不向公网暴露端口，确保数据层物理隔离。

### 4. 部署后端服务
```bash
cd backend

# 创建专用虚拟环境
python3.12 -m venv .venv
.venv/bin/pip install --upgrade pip
.venv/bin/pip install -r requirements.txt

# 配置生产环境变量
cp .env.example .env
vim .env   # 务必更新 QUANT_AUTH_SECRET、QUANT_POSTGRES_DSN 与管理员初始密码

# 执行数据库版本迁移
.venv/bin/alembic upgrade head

# 注册并启动 systemd 守护进程
sudo cp ../deploy/quant.service /etc/systemd/system/quant.service
sudo systemctl daemon-reload
sudo systemctl enable --now quant

# 检查服务健康状态
sudo systemctl status quant
curl http://127.0.0.1:8000/health
```

### 5. 编译与部署前端控制台
```bash
cd ../frontend
npm install
npm run build

# 部署至生产托管目录
sudo mkdir -p /var/www/enterprise-ai-quant
sudo rsync -a dist/ /var/www/enterprise-ai-quant/

# 配置 Nginx 站点
sudo cp ../deploy/nginx.conf /etc/nginx/conf.d/enterprise-ai-quant.conf
sudo nginx -t && sudo systemctl reload nginx
```

---

## 🛡️ 核心环境变量规范 (`.env`)

| 环境变量参数 | 默认 / 推荐值 | 作用与安全说明 |
|:---|:---|:---|
| `QUANT_ENVIRONMENT` | `production` | 运行环境标识 (`development` / `production`) |
| `QUANT_MODE` | `paper` | 盘口模式：`paper` (模拟盘) / `live` (实盘，开发环境强制阻断) |
| `QUANT_AUTH_SECRET` | *(32位随机密钥)* | JWT 签名密钥，**生产环境严禁使用默认值** |
| `QUANT_POSTGRES_DSN` | `postgresql+asyncpg://...` | PostgreSQL 异步连接池 URL |
| `QUANT_REDIS_URL` | `redis://127.0.0.1:6379/0` | 事件总线、异步队列与缓存地址 |
| `QUANT_LOG_FORMAT` | `json` | 生产环境推荐 JSON 格式输出，便于日志系统采集中继 |

---

## 🔒 金融级安全保障准则

1. **凭证隔离**：`.env`、交易私钥、数据库快照已被 `.gitignore` 全面排除，**严禁提交至版本仓库**；
2. **强制加密传输**：生产环境务必配置 TLS 证书并开启 `QUANT_FORCE_HTTPS=true`；
3. **熔断与恢复审批**：紧急情况下触发 `Kill Switch` 会立刻中断所有未成交流水，恢复实盘交易必须经由多角色授权审批流水，防止误操作。

---

## 📊 交付与路线图
关于已完成功能细项、各周期里程碑与回测基准结果，请参阅 [PROJECT_STATUS.md](PROJECT_STATUS.md)。
