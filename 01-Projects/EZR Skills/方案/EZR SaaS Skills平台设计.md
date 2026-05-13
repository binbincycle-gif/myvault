---
title: EZR SaaS Skills平台设计
tags:
  - project
  - EZR-Skills
  - Skill-Gateway
  - SaaS能力释放
  - 数字员工
status: draft
source: source/EZR_SaaS_Skills平台设计.pdf
---

# EZR SaaS Skills平台设计

## 项目背景

1. **业务变化**：从“卖 SaaS 功能”转向“卖 AI 能力接口（Skill）”。
2. **客户变化**：客户未来不是只点页面，而是要让 Agent 调能力。
3. **技术基础已具备**：已有业务微服务体系 + 指标平台，不是从 0 起步。
4. **组织目标明确**：团队要从人力交付转向能力交付，形成可复用资产。

## 平台定位

对外提供三样东西：

1. **Skill 发现**：告诉 Agent 有哪些能力。
2. **Skill 调用**：执行能力并返回数据。
3. **数字员工开通**：分配身份和权限。

Agent 方不需要了解 EZR 内部实现。

## 统一设计目标

### 1. 统一认知

明确我们搭建的是 **Skills 平台**，不是功能堆叠。

### 2. 统一边界

明确哪些能力复用现有体系，哪些必须新增。

### 3. 统一路径

从当前 CRM / OpenAPI / 指标服务，演进到可对外的 Agent 能力平台。

## 核心模块

### Skill Gateway

负责：

- Skill 发现
- Skill 调用
- 身份认证
- 权限解析
- 路由与聚合
- 调用审计

### 数字员工管理

负责：

- 数字员工身份创建
- 技能绑定
- 权限范围分配
- client_id / client_secret 管理
- 使用审计

## 管理判断

EZR Skills 的本质不是“给现有接口套一层 AI”，而是把 CRM 能力封装成 Agent 能理解、能调用、能审计、能复用的业务能力单元。

最重要的边界：

> 业务微服务继续作为事实源，Skill Gateway 只做身份、权限、路由、聚合、结构化返回和审计，不重写 CRM 业务逻辑。
