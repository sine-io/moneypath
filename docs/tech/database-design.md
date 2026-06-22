# 数据库设计

## 1. 设计原则

- PostgreSQL 作为主库。
- UUID 作为主键，便于后期拆分服务。
- 交易、测验、AI对话等关键数据保留历史，不做硬删除。
- 金额统一使用 `DECIMAL`，禁止使用浮点数存储。
- 所有表保留 `created_at`，核心表保留 `updated_at`。

## 2. ER概览

```text
users 1---1 portfolios 1---* positions
  |             |
  |             +---* transactions
  |
  +---* lesson_progress
  +---* quiz_attempts
  +---* ai_conversations 1---* ai_messages
  +---* user_achievements *---1 achievements
  +---* risk_assessments
```

## 3. 表结构

### 3.1 users

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE,
  phone VARCHAR(30) UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  avatar_url TEXT,
  level INTEGER NOT NULL DEFAULT 1,
  experience INTEGER NOT NULL DEFAULT 0,
  risk_profile VARCHAR(30),
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_phone ON users(phone);
```

### 3.2 lessons

课程也可以文件化存储，但为了后台管理，建议同步一份索引到数据库。

```sql
CREATE TABLE lessons (
  id VARCHAR(80) PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  level INTEGER NOT NULL,
  category VARCHAR(50) NOT NULL,
  summary TEXT,
  xp_reward INTEGER NOT NULL DEFAULT 100,
  estimated_minutes INTEGER NOT NULL DEFAULT 8,
  content_path TEXT NOT NULL,
  order_index INTEGER NOT NULL,
  is_published BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 3.3 lesson_progress

```sql
CREATE TABLE lesson_progress (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  lesson_id VARCHAR(80) NOT NULL REFERENCES lessons(id),
  status VARCHAR(30) NOT NULL DEFAULT 'not_started',
  best_score INTEGER,
  attempts INTEGER NOT NULL DEFAULT 0,
  time_spent_seconds INTEGER NOT NULL DEFAULT 0,
  completed_at TIMESTAMP,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now(),
  UNIQUE(user_id, lesson_id)
);

CREATE INDEX idx_lesson_progress_user ON lesson_progress(user_id);
```

### 3.4 quiz_attempts

```sql
CREATE TABLE quiz_attempts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  lesson_id VARCHAR(80) NOT NULL REFERENCES lessons(id),
  score INTEGER NOT NULL,
  total INTEGER NOT NULL,
  answers JSONB NOT NULL,
  passed BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 3.5 portfolios

```sql
CREATE TABLE portfolios (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID UNIQUE NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  cash DECIMAL(14,2) NOT NULL DEFAULT 100000.00,
  total_value DECIMAL(14,2) NOT NULL DEFAULT 100000.00,
  total_return DECIMAL(14,2) NOT NULL DEFAULT 0.00,
  return_rate DECIMAL(10,6) NOT NULL DEFAULT 0,
  reset_count INTEGER NOT NULL DEFAULT 0,
  last_reset_at TIMESTAMP,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 3.6 positions

```sql
CREATE TABLE positions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  portfolio_id UUID NOT NULL REFERENCES portfolios(id) ON DELETE CASCADE,
  symbol VARCHAR(30) NOT NULL,
  name VARCHAR(100) NOT NULL,
  asset_type VARCHAR(30) NOT NULL DEFAULT 'stock',
  quantity INTEGER NOT NULL,
  available_quantity INTEGER NOT NULL,
  avg_cost DECIMAL(12,4) NOT NULL,
  current_price DECIMAL(12,4),
  market_value DECIMAL(14,2),
  unrealized_pnl DECIMAL(14,2),
  unrealized_pnl_rate DECIMAL(10,6),
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now(),
  UNIQUE(portfolio_id, symbol)
);
```

### 3.7 transactions

```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  portfolio_id UUID NOT NULL REFERENCES portfolios(id) ON DELETE CASCADE,
  symbol VARCHAR(30) NOT NULL,
  name VARCHAR(100) NOT NULL,
  side VARCHAR(10) NOT NULL,
  quantity INTEGER NOT NULL,
  price DECIMAL(12,4) NOT NULL,
  gross_amount DECIMAL(14,2) NOT NULL,
  commission DECIMAL(12,2) NOT NULL DEFAULT 0,
  tax DECIMAL(12,2) NOT NULL DEFAULT 0,
  net_amount DECIMAL(14,2) NOT NULL,
  executed_at TIMESTAMP NOT NULL DEFAULT now(),
  created_at TIMESTAMP NOT NULL DEFAULT now()
);

CREATE INDEX idx_transactions_portfolio ON transactions(portfolio_id);
CREATE INDEX idx_transactions_executed ON transactions(executed_at);
```

### 3.8 ai_conversations / ai_messages

```sql
CREATE TABLE ai_conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(200),
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP NOT NULL DEFAULT now()
);

CREATE TABLE ai_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES ai_conversations(id) ON DELETE CASCADE,
  role VARCHAR(20) NOT NULL,
  content TEXT NOT NULL,
  safety_label VARCHAR(50),
  token_count INTEGER,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

### 3.9 achievements / user_achievements

```sql
CREATE TABLE achievements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(80) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NOT NULL,
  icon VARCHAR(100),
  xp_reward INTEGER NOT NULL DEFAULT 0,
  condition JSONB NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);

CREATE TABLE user_achievements (
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  achievement_id UUID NOT NULL REFERENCES achievements(id),
  unlocked_at TIMESTAMP NOT NULL DEFAULT now(),
  PRIMARY KEY(user_id, achievement_id)
);
```

### 3.10 risk_assessments

```sql
CREATE TABLE risk_assessments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  answers JSONB NOT NULL,
  score INTEGER NOT NULL,
  profile VARCHAR(30) NOT NULL,
  recommendation JSONB NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

## 4. 初始化数据

需要预置：

- 15个 MVP 课程索引。
- 20个常见成就。
- 风险测评题库。
- 股票/ETF白名单：沪深300ETF、中证500ETF、创业板ETF、黄金ETF、债券ETF等。

## 5. 迁移规范

- 使用 Alembic 管理迁移。
- 每次 schema 变更必须生成迁移文件。
- 禁止直接在生产数据库手改表结构。
- 迁移文件命名：`YYYYMMDD_HHMM_short_description.py`。
