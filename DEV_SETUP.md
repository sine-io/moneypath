# 开发环境搭建

## 1. 前置依赖

- Git
- Node.js 20+
- Python 3.11+
- Docker + Docker Compose
- uv（或 poetry）

## 2. 推荐目录

```text
~/work/finlearn-platform/
```

## 3. 前端初始化

```bash
cd frontend
npm install
npm run dev
```

## 4. 后端初始化

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
```

## 5. 数据库初始化

```bash
docker compose -f infra/docker-compose.yml up -d db redis
alembic upgrade head
```

## 6. 环境变量

复制 `.env.example` 为 `.env`，并填写：

- `DATABASE_URL`
- `REDIS_URL`
- `JWT_SECRET`
- `OPENAI_API_KEY`
- `AI_MODEL`

## 7. 本地开发顺序

1. 先跑前后端骨架。
2. 再接数据库。
3. 再接课程内容。
4. 再接交易逻辑。
5. 最后接 AI。

## 8. 调试建议

- 前端优先用假数据。
- 后端先写测试再实现。
- 行情接口先做 mock，再接真实数据。
- 所有大功能都拆成小任务。
