# 项目移交清单

## 1. 文档清单

- [x] README.md：项目总览。
- [x] PRD.md：产品需求。
- [x] TECH_SPEC.md：技术方案。
- [x] DATABASE_DESIGN.md：数据库设计。
- [x] API_SPEC.md：接口文档。
- [x] DESIGN_SYSTEM.md：UI/UX规范。
- [x] USER_FLOW.md：用户流程。
- [x] CURRICULUM.md：课程大纲。
- [x] CONTENT_GUIDE.md：内容创作规范。
- [x] DEV_SETUP.md：开发环境搭建。
- [x] DEPLOYMENT.md：部署指南。
- [x] TESTING.md：测试策略。
- [x] MVP_PLAN.md：开发计划。
- [x] BUDGET.md：预算。
- [x] GROWTH_STRATEGY.md：增长策略。

## 2. 接手人需要确认的问题

### 产品侧
- 目标市场先做中国用户，还是海外华人？
- 是否只做教育，还是后续做工具订阅？
- 第一版课程数量确认：10节还是15节？

### 技术侧
- AI模型使用 OpenAI 还是国产模型？
- 行情数据源是否需要购买 tushare 积分？
- 是否需要后台CMS，还是课程文件化即可？

### 合规侧
- 免责声明文案是否需要法务审核？
- AI回答是否需要记录审计日志？
- 是否完全避免实盘交易接入？

## 3. 开发优先级

P0：用户系统、课程系统、模拟交易核心、AI安全边界。  
P1：工具箱、成就、进度、基础回测。  
P2：社区、付费、高级报告、移动App。

## 4. 移交建议

新同事接手顺序：

1. 读 README.md。
2. 读 PRD.md 和 MVP_PLAN.md。
3. 读 TECH_SPEC.md 和 DATABASE_DESIGN.md。
4. 按 DEV_SETUP.md 建项目骨架。
5. 按 TESTING.md 先写核心测试。
6. 按 MVP_PLAN.md 周计划开发。

## 5. 当前状态

当前只完成规划文档，尚未创建实际代码仓库。可以基于这些文档新建项目，也可以先做 Figma/HTML 原型验证。
