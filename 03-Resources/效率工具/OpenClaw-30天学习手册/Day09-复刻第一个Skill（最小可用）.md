# Day 09｜复刻第一个Skill（最小可用）

## 今日主题

通过一个最小可用 Skill 的完整开发流程，理解 Skill 的实际构建方式——从定义文件到执行逻辑，掌握从"不会"到"能跑"的闭环。

## 技术原理（技术视角）

### 最小 Skill 的构成

一个可运行的 Skill 只需要三个核心文件：

```
my-skill/
├── skill.yaml        # 定义文件（名称、描述、输入输出 Schema）
├── handler.js        # 执行逻辑（实际做什么）
└── config.json       # 可选配置（超时、 retries 等）
```

### skill.yaml 结构解析

```yaml
name: hello_skill
description: "向指定人发送问候消息，适用于新成员加入场景"
version: "1.0.0"

input_schema:
  type: object
  properties:
    name:
      type: string
      description: "被问候人的名字"
    time_of_day:
      type: string
      enum: ["morning", "afternoon", "evening"]
      description: "问候时段"
  required: ["name"]

output_schema:
  type: object
  properties:
    message:
      type: string
      description: "生成的问候消息"
    sent_at:
      type: string
      format: date-time

handler: ./handler.js
config:
  timeout_ms: 5000
  retries: 2
```

### handler.js 编写规范

```javascript
// handler.js 示例
module.exports = async function({ name, time_of_day }, context) {
  // 1. 参数校验（可选，Schema 已做第一层校验）
  if (!name || name.trim() === "") {
    throw new Error("name 不能为空");
  }

  // 2. 业务逻辑
  const greetings = {
    morning: "早上好",
    afternoon: "下午好",
    evening: "晚上好"
  };
  const message = `${greetings[time_of_day]}，${name}！欢迎加入团队 🎉`;

  // 3. 返回结构化结果（必须符合 output_schema）
  return {
    message,
    sent_at: new Date().toISOString()
  };
};
```

### Skill 注册与调试

1. **放置到正确目录**
   - 默认 Skill 目录：`~/.openclaw/skills/`
   - 可在配置中自定义：`skills.path` 

2. **验证加载**
   - 启动 OpenClaw 时自动扫描目录
   - 或使用命令手动刷新：`openclaw skills reload`

3. **本地测试**
   ```bash
   openclaw skills test hello_skill --input '{"name": "张三", "time_of_day": "morning"}'
   ```

### 常见失败点

| 失败场景 | 原因 | 解决思路 |
|---------|------|---------|
| Skill 加载失败 | YAML 语法错误 / 文件路径错误 | 用 `yaml lint` 验证语法；检查 handler 路径 |
| 参数校验不通过 | input_schema 定义与实际不匹配 | 严格定义 types 和 required |
| 返回结果格式错误 | output_schema 与实际返回不一致 | 确保返回对象字段与 schema 完全一致 |
| 权限被拒绝 | 敏感操作未配置白名单 | 检查 skills.allowlist 配置 |

## 业务翻译（业务视角）

### 为什么需要"最小可用"思路

**先跑通，再优化——是 Skill 开发的核心原则。**

- **降低试错成本**：最小 Skill 只关注一件事，不求完美，只求可调用
- **快速验证价值**：跑通后才能判断这个 Skill 是否有业务价值
- **建立信心**：完成第一个 Skill 是后续复杂封装的基础

### 这个能力解决的核心问题

**从"想用 AI"到"能用 AI"的跨越**

- 没有 Skill 之前：需要写代码、配置环境、对接 API
- 有 Skill 机制后：声明式定义 + 简单脚本 = 可复用能力

### 对效率/质量/速度/可控的影响

- **效率**：最小可用版本可在 30 分钟内完成从 0 到 1
- **质量**：Schema 约束保证输入输出一致性，减少沟通损耗
- **速度**：一次封装，后续调用无需重复开发
- **可控**：每个 Skill 独立版本管理，可单独审批和监控

## 典型场景（个人版）

### 场景一：团队值班提醒

- **输入**：日期、人员名单
- **输出**：排班表、提醒消息
- **价值**：每周重复的值班通知自动化

### 场景二：代码片段检索

- **输入**：关键词（如 "promise all"）
- **输出**：相关代码片段、所在文件路径
- **价值**：团队知识沉淀复用

### 场景三：会议纪要生成

- **输入**：会议记录文本
- **输出**：结构化纪要（待办、决策、讨论点）
- **价值**：快速输出可执行会议结论

## 官方资料（带看点）

### 本地文档

- [[官方文档-本地镜像(节选)/03-Skill机制]] - 看点：Skill 定义文件的完整字段说明
- [[官方文档-本地镜像(节选)/04-Skill开发实战]] - 看点：从零创建到调试上线的完整流程
- [[本地文档索引]] - 查看所有本地镜像文档目录

### 在线文档

- [Skill 定义参考](https://docs.openclaw.ai/reference/skill-definition) - 完整的 YAML 字段说明
- [Skill 调试指南](https://docs.openclaw.ai/guide/skill-debug) - 本地测试和日志分析方法
- [Skill 示例库](https://github.com/openclaw/openclaw/tree/main/examples/skills) - 5+ 开箱即用的 Skill 示例

## 图示

### 最小 Skill 开发流程

![最小Skill开发流程](./assets/day09-minimal-skill-flow.svg)

## 管理者关注点（成本/效率/风险）

| 维度 | 关注点 |
|------|-------|
| **成本** | 第一个 Skill 开发成本约 30-60 分钟，后续复用零边际成本；建议从高频重复任务入手 |
| **效率** | 最小可用版本快速验证业务价值，确认有用后再投入完善；避免过度设计 |
| **风险** | 初期建议在非生产环境调试；敏感操作配置审批流；建立 Skill 上线检查清单 |

## 本日自检

1. 我是否能独立完成一个最小 Skill 的创建（定义 + handler + 测试）？
2. 我是否理解 skill.yaml 中每个字段的作用？
3. 我是否能说出一个适合封装为 Skill 的个人/团队高频场景？

## 一句话总结

**最小可用 Skill 的核心是先跑通再完善——用 30 分钟验证价值，用标准化机制实现复用。**
