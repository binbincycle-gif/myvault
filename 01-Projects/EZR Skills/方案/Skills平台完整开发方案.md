---
title: Skills平台完整开发方案
tags:
  - project
  - EZR-Skills
  - Skill-Gateway
  - 技术方案
  - 数字员工
status: draft
source: source/Skills平台完整开发方案.pdf
---

# Skills平台完整开发方案

## 项目概述

在现有 SaaS CRM 基础上，新建一套 Skills 平台，将 CRM 业务能力封装为标准化 Skill，通过统一接口对外提供，支持任意 Agent 框架调用。

典型能力包括：

- 会员洞察
- 导购话术
- 沉睡召回
- 营销策略
- 会员概况查询

## 核心目标

1. 对外提供标准化 Skill 能力，任意 Agent 可通过 REST 接口调用。
2. 建立数字员工身份体系，scope 与 SaaS 权限服务单一数据源对齐。
3. 现有 SaaS 系统和业务微服务零改动。
4. 第一阶段交付：
   - Skill Gateway
   - 数字员工管理页
   - `member_summary` skill

## 改动范围

| 模块 | 改动类型 | 说明 |
|---|---|---|
| Skill Gateway | 全新建设 | 独立服务，完全新建，独立数据库 |
| 数字员工管理页 | 新增入口 | 嵌入 SaaS 导航，新增独立菜单 |
| SaaS 用户 / 权限 | 只读引用 | 不改代码，Gateway 实时 gRPC 查询 |
| 业务微服务 | 不改动 | Gateway 使用 service account 内网直连 |

## 技术架构

### 分层设计

| 层级 | 组件 | 职责 |
|---|---|---|
| 调用方 | 外部 Agent | OpenAI / Claude / Dify / 自研，持有 skill_token 调用 |
| 接入层 | Skill Gateway | 身份认证、scope 解析、skill 路由、调用审计 |
| 能力层 | Skills | `member_summary` 等具体 skill 实现 |
| 数据层 | 业务微服务 | 会员 / 订单 / 商品 / 指标 / 营销，gRPC 内网 |
| 权限层 | SaaS 权限服务 | 单一数据源，saas_user_id → store_ids[] |

### 技术选型

| 项目 | 选型 | 说明 |
|---|---|---|
| Skill Gateway 语言 | Go | 并发友好，gRPC 原生支持，启动快 |
| 对外协议 | HTTPS REST | 兼容所有 Agent 框架 |
| 内网通信 | gRPC | 复用业务服务现有 proto |
| 身份凭证 | JWT HS256 | 有效期 8h，含 de_id + saas_user_id |
| 数据库 | PostgreSQL | 独立实例 |
| scope 缓存 | 本地内存 TTL 5min | 降低权限服务压力 |

## 对外接口

### 1. POST `/auth/token`

数字员工凭 `client_id` 和 `client_secret` 换取 JWT，有效期 8 小时。Agent 启动时调用一次，token 过期后重新获取。

> 安全说明：原资料中的示例凭证已在本文档中脱敏，不写入可检索正文。

### 2. GET `/v1/skills`

返回平台所有可用 Skill 描述，格式兼容 OpenAI `tools[]`，Agent 注册后可由 LLM 自动决策调用时机。

示例 Skill：

- `member_summary`：查询门店会员概况，含新增、流失、RFM 分层。

### 3. POST `/v1/skills/{skill_id}/invoke`

执行 Skill 核心调用接口。Gateway 完成鉴权、scope 解析、路由执行后返回结构化数据。

返回字段包括：

- `run_id`
- `skill_id`
- `latency_ms`
- `data`
- `scope`

## 错误码设计

| HTTP | 场景 | error |
|---|---|---|
| 401 | token 无效 / 过期 | `unauthorized · token_expired` |
| 403 | scope 权限不足 | `forbidden · scope_not_allowed` |
| 404 | skill_id 不存在 | `not_found · skill_not_registered` |
| 502 | 业务服务超时 / 失败 | `upstream_error`，降级返回空数据 |
| 503 | 权限服务不可达 | `dependency_unavailable · scope_fetch_failed` |

## 数字员工管理原则

数字员工只存身份信息，不存权限数据。

scope 在每次调用时实时从 SaaS 权限服务获取，避免权限双写和权限漂移。

## 第一阶段建议验收

1. Skill Gateway 可部署、可认证、可审计。
2. 数字员工管理页可创建身份并生成凭证。
3. `/v1/skills` 可返回标准化 Skill 列表。
4. `member_summary` 可完成真实门店会员概况查询。
5. scope 与 SaaS 权限服务对齐，不能越权访问。
6. 调用日志完整记录：调用人、数字员工、skill、scope、耗时、错误码。

## 风险

- 权限服务不可达会影响调用链路，需要降级策略。
- service account 内网直连业务微服务必须有审计边界。
- Skill 参数定义如果过宽，会导致 Agent 调用不可控。
- 第一阶段不要同时做太多 Skill，建议先把 `member_summary` 做到可审计、可复用、可复制。
