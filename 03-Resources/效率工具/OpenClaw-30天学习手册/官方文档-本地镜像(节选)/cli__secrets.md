# secrets（学习节选）

> 来源：`/Users/cycle/.local/share/fnm/node-versions/v24.13.1/installation/lib/node_modules/openclaw/docs/cli/secrets.md`
> 用途：理解 OpenClaw 如何审计明文密钥、未解析引用和遗留敏感配置。

## 核心结论

`openclaw secrets` 不是单纯“存密钥”的命令，而是一套 **SecretRef 管理 + 运行时校验 + 明文残留审计** 的安全闭环。

## 关键子命令

### 1. `openclaw secrets audit`
用于只读扫描，重点检查：
- 明文 secret 是否残留在配置中
- SecretRef 是否未正确解析
- 是否存在配置优先级漂移
- `agents/*/agent/models.json` 等生成文件里是否仍有敏感字段
- 历史遗留配置是否未清理

常用命令：

```bash
openclaw secrets audit --check
openclaw secrets audit --json
```

返回语义：
- `status: clean | findings | unresolved`
- `summary`: 包含 `plaintextCount`、`unresolvedRefCount`、`shadowedRefCount`、`legacyResidueCount`

### 2. `openclaw secrets reload`
在不改配置文件的前提下，重新解析 SecretRef，并在全部成功时原子替换运行时快照。

### 3. `openclaw secrets apply`
执行保存好的变更计划，并清理目标明文残留。

## 学习看点

1. **审计先于上线**：先检查明文残留，再谈自动化运行。
2. **失败保持旧快照**：重新加载失败时，不会把半成品配置切到运行态。
3. **可用于日常巡检**：`audit --check` 很适合接入发布前检查、健康巡检和安全周检。

## 和 Day13 的关系

- 审计不是“出事后查日志”，而是“日常检查敏感配置是否已偏离规范”。
- Secrets 审计与日志脱敏配合，分别解决：
  - 配置层敏感信息是否安全
  - 运行层敏感信息是否被暴露
