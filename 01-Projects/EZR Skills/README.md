---
title: EZR Skills
tags:
  - project
  - AI项目
  - EZR-Skills
  - Skill-Gateway
  - SaaS能力释放
status: active
owner: 待定
priority: high
review_date: 2026-06-16
---

# EZR Skills

## 项目定位

EZR Skills 是把 EZR CRM 的业务能力，从“页面功能 / 内部接口”升级为 **Agent 可发现、可调用、可审计、可复用的 Skill 能力平台**。

一句话：

> 过去 SaaS 卖页面和流程，未来 Skills 平台卖可被 Agent 调用的业务能力。

## 背景判断

1. 业务变化：从“卖 SaaS 功能”转向“卖 AI 能力接口（Skill）”。
2. 客户变化：客户未来不是只点页面，而是让 Agent 调能力。
3. 技术基础：EZR 已有业务微服务体系 + 指标平台，不是从 0 起步。
4. 组织目标：从人力交付转向能力交付，形成可复用资产。

## 平台对外提供三类能力

1. **Skill 发现**：告诉 Agent 有哪些能力。
2. **Skill 调用**：执行并返回结构化数据。
3. **数字员工开通**：分配身份和权限。

Agent 方不需要了解 EZR 内部实现。

## 核心架构

| 层级 | 组件 | 职责 |
|---|---|---|
| 调用方 | 外部 Agent | OpenAI / Claude / Dify / 自研，持有 skill_token 调用 |
| 接入层 | Skill Gateway | 身份认证、scope 解析、skill 路由、调用审计 |
| 能力层 | Skills | `member_summary`、销售日报汇总、试衣记录等 Skill 实现 |
| 数据层 | 业务微服务 | 会员 / 订单 / 商品 / 指标 / 营销等内网服务 |
| 权限层 | SaaS 权限服务 | 单一数据源，saas_user_id → store_ids[] |

## 第一阶段目标

交付：

- Skill Gateway
- 数字员工管理页
- `member_summary` skill
- 经营分析类 Skill 样例，如销售日报汇总

原则：

- 现有 SaaS 系统和业务微服务零改动。
- SaaS 权限服务只读引用，不重复维护权限。
- 业务微服务作为事实源，Gateway 做身份、权限、路由、聚合、结构化返回和审计。

## 当前已沉淀资料

### 方案

- [[方案/EZR SaaS Skills平台设计]]
- [[方案/Skills平台完整开发方案]]

### 技能样例

- [[技能样例/EZR Skill建设：销售日报汇总]]
- [[技能样例/试衣记录]]

### 原始文件

- [source/EZR_SaaS_Skills平台设计.pdf](source/EZR_SaaS_Skills平台设计.pdf)
- [source/EZR_SKill_建设.pdf](source/EZR_SKill_建设.pdf)
- [source/Skills平台完整开发方案.pdf](source/Skills平台完整开发方案.pdf)
- [source/试衣记录.pdf](source/试衣记录.pdf)

## 当前推进动作

- [ ] 明确第一阶段 P0 Skill：`member_summary`、销售日报汇总、试衣记录是否都纳入。
- [ ] 确认 Skill Gateway 技术栈与部署方式：Go + REST + gRPC + PostgreSQL。
- [ ] 定义数字员工管理页最小功能：创建身份、生成凭证、绑定 Skill、审计查看。
- [ ] 对接 SaaS 权限服务，确认 scope 获取方式和缓存策略。
- [ ] 输出第一版 Skill 标准模板：`skill_id`、输入、输出、权限、上游服务、错误策略、示例。
- [ ] 将示例凭证、测试地址从正式知识库正文中脱敏，仅保留在原始 source 文件中。

## 风险与边界

1. **不要重写 CRM 业务逻辑**：业务微服务继续作为事实源。
2. **权限不能双写**：scope 必须与 SaaS 权限服务单一数据源对齐。
3. **不要首期塞太多 Skill**：先用 1-2 个高频 Skill 验证 Gateway、权限、审计、返回结构。
4. **凭证必须脱敏**：测试地址和 api_key 不写入可检索正文。
5. **Agent 调用必须可审计**：记录调用人、数字员工、Skill、scope、耗时、错误码。

## 下次复盘问题

1. 第一阶段到底交付几个 Skill？是否只保留 `member_summary` + 销售日报汇总？
2. 数字员工身份与 SaaS 用户 / 员工的映射关系是否确认？
3. scope 是按品牌、门店、指标还是组合授权？
4. Skill 返回结构是否统一，是否兼容 OpenAI tools[]？
5. 调用日志和错误码是否能支撑企业级审计？
6. 是否需要把 Skills 平台从“项目”提升为 EZR AI 能力底座？
