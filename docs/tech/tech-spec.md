# 技术方案文档

## 1. 架构目标

FinLearn Platform 采用 **Next.js 前端 + FastAPI 后端 + PostgreSQL + Redis + AI 服务 + 财经数据服务** 的分层架构，目标是：

- 支持零基础用户的课程学习、模拟交易、AI问答、工具计算。
- 初期单机 Docker Compose 可部署，后期可平滑拆分服务。
- 财经数据源优先使用免费/低成本方案，保留替换能力。
- 所有“投资建议”能力必须以教育、模拟、风险提示为边界，不做明确荐股。

## 2. 总体架构

```text
Browser / Mobile H5
        |
        v
Next.js 14 App Router
        |
        v
FastAPI API Gateway
  |        |          |
  v        v          v
PostgreSQL Redis   Market Data Service
                    | akshare / tushare / yfinance
  |
  v
AI Tutor Service -> OpenAI / DeepSeek / Qwen
```

## 3. 技术栈

| 层级 | 选型 | 说明 |
|---|---|---|
| 前端 | Next.js 14 + React + TypeScript | 现代 Web，支持 SSR/SEO |
| UI | TailwindCSS + shadcn/ui | 快速搭建、风格统一 |
| 图表 | ECharts / Recharts | 收益曲线、资产饼图、K线展示 |
| 状态 | Zustand + TanStack Query | 客户端状态与服务端缓存 |
| 后端 | Python 3.11 + FastAPI | 高性能、易集成 AI/财经数据 |
| ORM | SQLAlchemy 2 + Alembic | 数据模型与迁移 |
| 数据库 | PostgreSQL 15 | 主业务库 |
| 缓存 | Redis 7 | 行情缓存、AI限流、任务状态 |
| 异步任务 | Celery / RQ（二选一） | 同步行情、生成报告、批量回测 |
| AI | OpenAI GPT-4o-mini，备选 DeepSeek/Qwen | 成本优先，支持替换 |
| 数据 | akshare，tushare，yfinance | A股为主，兼顾海外资产 |
| 部署 | Docker Compose + Nginx | MVP成本低、交付简单 |

## 4. 仓库结构建议

```text
finlearn-platform/
├── frontend/
│   ├── app/
│   │   ├── (auth)/
│   │   ├── (dashboard)/learn/
│   │   ├── (dashboard)/trade/
│   │   ├── (dashboard)/ai-tutor/
│   │   └── (dashboard)/tools/
│   ├── components/
│   ├── lib/
│   ├── store/
│   └── types/
├── backend/
│   ├── api/
│   ├── core/
│   ├── db/
│   ├── models/
│   ├── schemas/
│   ├── services/
│   ├── tests/
│   └── main.py
├── content/
│   ├── lessons/
│   └── quizzes/
├── infra/
│   ├── docker-compose.yml
│   ├── nginx.conf
│   └── .env.example
└── docs/
```

## 5. 核心服务

### 5.1 Lesson Service

职责：课程读取、进度更新、测验判分、经验值发放。

关键方法：

```python
class LessonService:
    async def list_lessons(user_id: str) -> list[LessonSummary]: ...
    async def get_lesson(lesson_id: str, user_id: str) -> LessonDetail: ...
    async def submit_quiz(user_id: str, lesson_id: str, answers: list[Answer]) -> QuizResult: ...
    async def complete_lesson(user_id: str, lesson_id: str) -> ProgressResult: ...
```

### 5.2 Market Data Service

职责：统一封装行情数据源，屏蔽 akshare/tushare/yfinance 差异。

缓存策略：

| 数据 | 缓存 |
|---|---|
| A股实时行情 | 15分钟 |
| 历史K线 | 1天 |
| 财务指标 | 1天 |
| 股票搜索列表 | 1天 |

备用策略：

1. 优先 akshare。
2. akshare 失败时尝试 tushare。
3. 仍失败则返回缓存旧数据，并标记 `data_stale=true`。

### 5.3 Virtual Trading Service

职责：虚拟账户、买卖、持仓、交易流水、挑战任务。

交易边界：

- 初始资金 100000 虚拟币。
- 模拟 A股 T+1，可配置是否严格启用。
- 交易费率：佣金万 2.5，最低 5 元；卖出印花税千 1。
- 禁止超额买入、禁止卖空。

### 5.4 AI Tutor Service

职责：AI问答、学习建议、持仓教育性分析、风险提示。

Prompt原则：

- 只做教育和模拟分析。
- 不提供“买入/卖出某股票”的直接指令。
- 必须提示风险和不确定性。
- 遇到用户要求荐股、内幕消息、保本收益，必须拒绝并转为风险教育。

## 6. 前端页面

| 页面 | 路径 | 说明 |
|---|---|---|
| 首页 | `/` | 产品介绍、注册入口 |
| 登录 | `/login` | 手机/邮箱登录 |
| 学习首页 | `/learn` | 关卡地图、进度 |
| 课程详情 | `/learn/[lessonId]` | 图文、互动、测验 |
| 模拟交易 | `/trade` | 账户、持仓、交易入口 |
| 行情搜索 | `/trade/market` | 搜索股票/ETF |
| AI导师 | `/ai-tutor` | 对话式问答 |
| 工具箱 | `/tools` | 各类计算器入口 |
| 配置计算器 | `/tools/allocator` | 资产比例与回测 |
| 定投模拟器 | `/tools/dca` | 定投回测 |
| 估值仪表盘 | `/tools/valuation` | 股票指标解释 |

## 7. 非功能要求

| 指标 | MVP目标 |
|---|---|
| 首屏加载 | < 2.5s |
| API P95 | < 500ms，不含AI |
| AI首字响应 | < 3s |
| 同时在线 | 1000用户 |
| 可用性 | 99.5% |
| 数据备份 | 每日一次 |

## 8. 关键风险与缓解

| 风险 | 表现 | 缓解 |
|---|---|---|
| 行情源不稳定 | akshare接口变化 | 封装数据源，加入 tushare 备用 |
| AI费用高 | 问答量增长 | 缓存高频问答，低价模型优先 |
| 合规风险 | 用户误解为荐股 | 强制免责声明，Prompt限制，敏感拦截 |
| 教育内容质量 | 概念错误 | 内容评审流程，引用来源，版本管理 |
| 新手流失 | 学习负担重 | 每关5-10分钟，游戏化反馈 |

## 9. 开发原则

- 先MVP，不做复杂社区和付费系统。
- 所有财经计算必须有单元测试。
- AI输出必须经过安全边界检查。
- 所有课程内容独立放在 `content/`，方便非技术同事维护。
- 数据源服务必须可以替换，不要把 akshare 调用散落在业务代码中。
