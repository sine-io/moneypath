# 钱途 MoneyPath

> 让理财从零开始变简单 | 游戏化学习 + 无风险实践 + AI私人导师

[![GitHub Pages](https://github.com/sine-io/moneypath/actions/workflows/docs.yml/badge.svg)](https://github.com/sine-io/moneypath/actions/workflows/docs.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 🎯 项目简介

**钱途 (MoneyPath)** 是一个面向零基础投资者的综合性财商教育平台，通过"知识体系 + 模拟交易 + AI导师"三位一体的方式，帮助用户系统化学习投资理财知识。

### 核心特色

- 🎮 **游戏化学习**：20个闯关式课程，从零基础到建立个人投资框架
- 📈 **无风险实践**：虚拟账户 + 真实A股行情，模拟交易积累经验
- 🤖 **AI私人导师**：24/7智能答疑，个性化学习路径推荐
- 🛠️ **实战工具箱**：资产配置计算器、定投模拟器、估值仪表盘

## 📚 在线文档

**完整文档站点**：[https://sine-io.github.io/moneypath/](https://sine-io.github.io/moneypath/)

文档包含：
- 📖 完整课程大纲（4个阶段，20+关卡）
- 🎯 新手引导与学习系统
- 💻 技术架构与API文档
- 📊 运营规划与成本预算
- 🎨 设计规范与用户体验流程

## 🚀 快速开始

### 在线访问

访问在线文档站点：[https://sine-io.github.io/moneypath/](https://sine-io.github.io/moneypath/)

### 本地运行

```bash
# 1. 克隆仓库
git clone https://github.com/sine-io/moneypath.git
cd moneypath

# 2. 安装依赖
pip install mkdocs-material pymdown-extensions

# 3. 启动本地开发服务器
mkdocs serve

# 4. 浏览器访问 http://127.0.0.1:8000
```

### 构建静态站点

```bash
# 生成静态HTML文件到 site/ 目录
mkdocs build

# 使用 --strict 模式检查错误
mkdocs build --strict
```

## 📂 项目结构

```
moneypath/
├── docs/                      # 文档源文件
│   ├── index.md              # 首页
│   ├── curriculum/           # 课程大纲
│   ├── guide/                # 新手指南
│   ├── practice/             # 实践系统
│   ├── evaluation/           # 评审报告
│   ├── tech/                 # 技术文档
│   ├── operations/           # 运营规划
│   ├── design/               # 设计系统
│   ├── about/                # 关于
│   └── stylesheets/          # 自定义样式
├── .github/
│   └── workflows/
│       └── docs.yml          # GitHub Actions工作流
├── mkdocs.yml                # MkDocs配置文件
└── README.md                 # 本文件
```

## 🛠️ 技术栈

### 文档站点
- **生成器**：MkDocs
- **主题**：Material for MkDocs
- **部署**：GitHub Pages
- **CI/CD**：GitHub Actions

### 平台技术栈（规划）

**前端**
- Next.js 14 (App Router) + React 18 + TypeScript 5
- TailwindCSS 3 + Recharts / ECharts

**后端**
- Python 3.11+ + FastAPI 0.110+
- PostgreSQL 15 + Redis 7 + SQLAlchemy 2.0

**AI层**
- OpenAI GPT-4o-mini + LangChain
- 备选：DeepSeek / 通义千问

**数据源**
- akshare (A股数据) + Yahoo Finance (美股/港股)

## 📅 开发计划

### MVP时间线（3个月）

**第1个月：核心教育模块**
- Week 1-2：项目脚手架 + 用户认证
- Week 3-4：5个新手村关卡（含交互组件）
- Week 5：测验系统 + 经验值/成就

**第2个月：模拟交易**
- Week 6-7：市场数据集成（akshare）
- Week 8：虚拟账户 + 交易逻辑
- Week 9：持仓管理 + 盈亏计算

**第3个月：AI导师 + 工具**
- Week 10：GPT对话接口
- Week 11：资产配置计算器 + 定投模拟器
- Week 12：测试 + 部署上线

详见：[MVP开发计划](https://sine-io.github.io/moneypath/operations/mvp-plan/)

## 🤝 贡献指南

我们欢迎任何形式的贡献！

### 如何贡献

1. **Fork 本仓库**
2. **创建特性分支** (`git checkout -b feature/AmazingFeature`)
3. **提交更改** (`git commit -m 'Add some AmazingFeature'`)
4. **推送到分支** (`git push origin feature/AmazingFeature`)
5. **提交 Pull Request**

### 贡献方向

- 🐛 报告问题或建议新功能（[Issues](https://github.com/sine-io/moneypath/issues)）
- 📝 改进文档或修正错误
- 🎨 优化课程内容和互动设计
- 💻 提交代码（需遵循项目编码规范）

详见：[贡献指南](https://sine-io.github.io/moneypath/about/contributing/)

## 💰 成本估算

### 运营成本（月）

| 项目 | 费用 |
|------|------|
| 云服务器（2核4G） | ¥100 |
| PostgreSQL托管 | ¥0（自建） |
| Redis托管 | ¥0（自建） |
| AI调用（GPT-4o-mini） | ¥500 |
| 域名 + CDN | ¥50 |
| **月成本合计** | **¥650** |

**年成本**：¥7,800

详见：[成本预算](https://sine-io.github.io/moneypath/operations/budget/)

## ⚠️ 免责声明

本平台仅提供投资理财教育内容，所有信息仅供学习参考，不构成任何投资建议。投资有风险，决策需谨慎。用户需自行承担投资决策的风险和责任。

## 📄 许可证

本项目采用 [MIT License](LICENSE) 开源协议。

## 📞 联系我们

- **GitHub Organization**: [sine-io](https://github.com/sine-io)
- **项目仓库**: [moneypath](https://github.com/sine-io/moneypath)
- **问题反馈**: [Issues](https://github.com/sine-io/moneypath/issues)
- **在线文档**: [https://sine-io.github.io/moneypath/](https://sine-io.github.io/moneypath/)

---

**Slogan**: 你的钱途，明明白白 | Your Money, Your Path

**最后更新**：2026-06-22  
**文档版本**：v1.0

Made with ❤️ by Sine.io Team
