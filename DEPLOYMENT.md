# 部署指南

## 1. 部署目标

MVP 优先使用 **Docker Compose** 一键部署，降低交接和运维门槛。

## 2. 环境要求

- Ubuntu 22.04+
- Docker 24+
- Docker Compose v2
- 2核4G 起步
- 50GB SSD 起步

## 3. 仓库准备

建议目录：

```text
finlearn-platform/
├── frontend/
├── backend/
├── content/
├── infra/
└── docs/
```

## 4. 环境变量

`.env.example` 需要包含：

```bash
DATABASE_URL=postgresql://finlearn:password@db:5432/finlearn
REDIS_URL=redis://redis:6379/0
JWT_SECRET=change_me
OPENAI_API_KEY=change_me
AI_MODEL=gpt-4o-mini
AKSHARE_ENABLED=true
TSUARE_ENABLED=false
```

## 5. 本地启动

### 5.1 启动依赖

```bash
docker compose -f infra/docker-compose.yml up -d db redis
```

### 5.2 启动后端

```bash
cd backend
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### 5.3 启动前端

```bash
cd frontend
npm install
npm run dev
```

## 6. 生产部署

### 6.1 构建镜像

```bash
docker compose -f infra/docker-compose.yml build
```

### 6.2 启动服务

```bash
docker compose -f infra/docker-compose.yml up -d
```

### 6.3 反向代理

Nginx 负责：
- 前端静态资源
- API 反代
- HTTPS 证书
- 限流

## 7. 数据库初始化

1. 创建数据库。
2. 运行 Alembic 迁移。
3. 导入课程索引和成就种子数据。
4. 创建初始管理员账号。

## 8. 备份策略

- PostgreSQL 每日自动备份。
- 备份保留 7 天。
- 每周做一次恢复演练。

## 9. 监控

建议监控：

- CPU / 内存 / 磁盘
- API 响应时间
- 任务失败率
- 行情源失败率
- AI 请求失败率

## 10. 上线检查清单

- [ ] `.env` 已替换为真实配置
- [ ] 数据库迁移已执行
- [ ] 管理员账号已创建
- [ ] 课程内容已导入
- [ ] AI key 已配置
- [ ] 域名和证书已绑定
- [ ] 健康检查通过
