# API接口文档

## 1. 基础约定

- Base URL：`/api/v1`
- 数据格式：JSON
- 时间格式：ISO 8601，例如 `2026-06-22T10:30:00Z`
- 认证：`Authorization: Bearer <access_token>`
- 分页参数：`page`、`page_size`

## 2. 通用响应

### 成功

```json
{
  "success": true,
  "data": {},
  "request_id": "req_xxx"
}
```

### 失败

```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_CASH",
    "message": "可用资金不足"
  },
  "request_id": "req_xxx"
}
```

## 3. 错误码

| code | HTTP | 说明 |
|---|---:|---|
| UNAUTHORIZED | 401 | 未登录或 token 无效 |
| FORBIDDEN | 403 | 无权限 |
| NOT_FOUND | 404 | 资源不存在 |
| VALIDATION_ERROR | 422 | 参数错误 |
| INSUFFICIENT_CASH | 400 | 虚拟资金不足 |
| INSUFFICIENT_POSITION | 400 | 持仓不足 |
| MARKET_DATA_UNAVAILABLE | 503 | 行情数据不可用 |
| AI_SAFETY_BLOCKED | 400 | AI请求触发安全边界 |

## 4. 认证接口

### 注册

`POST /auth/register`

Request:

```json
{
  "username": "xiaowang",
  "email": "xw@example.com",
  "password": "StrongPass123"
}
```

Response:

```json
{
  "success": true,
  "data": {
    "user": {"id": "uuid", "username": "xiaowang", "level": 1},
    "access_token": "jwt",
    "refresh_token": "jwt"
  }
}
```

### 登录

`POST /auth/login`

Request:

```json
{"account": "xw@example.com", "password": "StrongPass123"}
```

## 5. 学习接口

### 获取课程列表

`GET /lessons`

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "l1_compound_interest",
      "title": "复利的魔力",
      "level": 1,
      "status": "completed",
      "best_score": 100,
      "xp_reward": 100,
      "estimated_minutes": 8
    }
  ]
}
```

### 获取课程详情

`GET /lessons/{lesson_id}`

Response字段：

- `sections`：图文内容块。
- `interactions`：互动组件配置。
- `quiz`：测验题。

### 提交测验

`POST /lessons/{lesson_id}/quiz`

Request:

```json
{
  "answers": [
    {"question_id": "q1", "answer": "B"},
    {"question_id": "q2", "answer": "A"}
  ]
}
```

Response:

```json
{
  "success": true,
  "data": {
    "score": 80,
    "passed": true,
    "explanations": [
      {"question_id": "q1", "correct": true, "explanation": "72法则..."}
    ],
    "xp_gained": 100,
    "unlocked_achievements": []
  }
}
```

## 6. 模拟交易接口

### 获取组合概览

`GET /portfolio`

Response:

```json
{
  "success": true,
  "data": {
    "cash": 83600.00,
    "positions_value": 16400.00,
    "total_value": 100000.00,
    "total_return": 0,
    "return_rate": 0
  }
}
```

### 获取持仓

`GET /portfolio/positions`

Response:

```json
{
  "success": true,
  "data": [
    {
      "symbol": "510300",
      "name": "沪深300ETF",
      "quantity": 1000,
      "available_quantity": 1000,
      "avg_cost": 4.1,
      "current_price": 4.2,
      "market_value": 4200,
      "unrealized_pnl": 100,
      "unrealized_pnl_rate": 0.0244
    }
  ]
}
```

### 买入

`POST /orders/buy`

Request:

```json
{"symbol": "510300", "quantity": 1000, "order_type": "market"}
```

Response:

```json
{
  "success": true,
  "data": {
    "transaction_id": "uuid",
    "symbol": "510300",
    "name": "沪深300ETF",
    "side": "buy",
    "quantity": 1000,
    "price": 4.2,
    "gross_amount": 4200,
    "commission": 5,
    "tax": 0,
    "net_amount": 4205
  }
}
```

### 卖出

`POST /orders/sell`

Request:

```json
{"symbol": "510300", "quantity": 500, "order_type": "market"}
```

## 7. 行情接口

### 搜索标的

`GET /market/search?q=沪深300`

Response:

```json
{
  "success": true,
  "data": [
    {"symbol": "510300", "name": "沪深300ETF", "asset_type": "etf", "market": "SH"}
  ]
}
```

### 获取实时行情

`GET /market/quote/{symbol}`

Response:

```json
{
  "success": true,
  "data": {
    "symbol": "510300",
    "name": "沪深300ETF",
    "price": 4.2,
    "change": 0.03,
    "change_percent": 0.0072,
    "volume": 12345678,
    "timestamp": "2026-06-22T10:30:00Z",
    "data_stale": false
  }
}
```

### 获取K线

`GET /market/kline/{symbol}?period=daily&start=2024-01-01&end=2026-06-22`

## 8. AI导师接口

### 创建/继续对话

`POST /ai/chat`

Request:

```json
{
  "conversation_id": "uuid_or_null",
  "message": "为什么基金会亏钱？",
  "include_portfolio_context": true
}
```

Response:

```json
{
  "success": true,
  "data": {
    "conversation_id": "uuid",
    "message": "基金亏损通常来自市场波动、买入价格、持仓集中度...",
    "safety_label": "education",
    "suggested_actions": [
      {"type": "open_lesson", "label": "学习风险与收益", "lesson_id": "l3_risk_return"}
    ]
  }
}
```

### 获取对话历史

`GET /ai/conversations`

## 9. 工具接口

### 资产配置回测

`POST /tools/allocation-backtest`

Request:

```json
{
  "allocation": {"stock": 0.6, "bond": 0.3, "gold": 0.1},
  "start_date": "2016-01-01",
  "end_date": "2026-01-01",
  "rebalance": "quarterly"
}
```

Response:

```json
{
  "success": true,
  "data": {
    "annual_return": 0.078,
    "max_drawdown": -0.18,
    "sharpe_ratio": 0.71,
    "curve": [{"date": "2016-01-01", "nav": 1.0}]
  }
}
```

### 定投模拟

`POST /tools/dca-simulate`

Request:

```json
{
  "symbol": "510300",
  "amount": 1000,
  "frequency": "monthly",
  "start_date": "2020-01-01",
  "end_date": "2026-01-01"
}
```

### 估值分析

`GET /tools/valuation/{symbol}`

## 10. WebSocket（后期）

MVP 可以先轮询；后期支持：

- `/ws/market`：行情推送。
- `/ws/ai-chat`：AI流式输出。

## 11. 版本管理

- API版本从 `/api/v1` 开始。
- 破坏性变更必须新开 `/api/v2`。
- 响应字段只增不删，删除字段需至少提前一个版本标记 deprecated。
