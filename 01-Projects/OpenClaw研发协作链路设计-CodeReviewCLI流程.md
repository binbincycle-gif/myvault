# OpenClaw Code Review CLI 流程设计

> 场景：CLI 主动触发式代码审查
> 创建时间：2026-03-05

---

## 一、流程设计

### 1.1 整体流程

```
研发完成编码
     │
     ▼
$ openclaw review .    （本地CLI命令）
     │
     ▼
OpenClaw 接收请求
     │
     ▼
代码解析 + 审查分析
     │
     ├── 代码规范
     ├── 逻辑正确性
     ├── 性能隐患
     ├── 安全漏洞
     └── 测试覆盖
     │
     ▼
终端输出审查报告
     │
     ▼
研发确认/修改
     │
     ▼
git push
```

### 1.2 命令设计

```bash
# 基础用法
openclaw review .                    # 审查当前目录
openclaw review ./src --staged       # 仅审查暂存区
openclaw review main --compare dev   # 比较分支

# 输出选项
openclaw review . --format terminal  # 终端输出（默认）
openclaw review . --format json     # JSON输出
openclaw review . --output report.md # 输出到文件
```

---

## 二、审查维度

### 2.1 必检项

| 维度 | 检查内容 | 严重级别 |
|------|----------|----------|
| 代码规范 | 命名、格式、注释 | P2 |
| 逻辑正确性 | 边界条件、空指针等 | P0 |
| 性能隐患 | 循环、N+1查询等 | P1 |
| 安全漏洞 | SQL注入、XSS等 | P0 |
| 错误处理 | 异常捕获、返回值 | P1 |

### 2.2 可选项

```bash
openclaw review . --check all        # 全部检查（默认）
openclaw review . --check security  # 仅安全
openclaw review . --check performance # 仅性能
```

---

## 三、输出示例

### 3.1 终端输出

```
╔══════════════════════════════════════════════════════════════╗
║                    OpenClaw Code Review                      ║
╠══════════════════════════════════════════════════════════════╣
║ 文件: src/user.service.ts                                    ║
║ 耗时: 2.3s                                                  ║
╠══════════════════════════════════════════════════════════════╣

[P0] ⚠️ 安全漏洞
  位置: src/user.service.ts:45
  问题: SQL拼接存在注入风险
  建议: 使用参数化查询
  
  ```typescript
  // ❌ 错误
  db.query(`SELECT * FROM users WHERE id = ${userId}`);
  
  // ✅ 正确
  db.query('SELECT * FROM users WHERE id = ?', [userId]);
  ```

──────────────────────────────────────────────────────────────

[P1] ⚠️ 性能隐患
  位置: src/user.service.ts:78
  问题: N+1 查询问题
  建议: 使用批量查询或JOIN
  
  ```typescript
  // ❌ 错误
  for (const user of users) {
    const orders = await db.getOrders(user.id);
  }
  
  // ✅ 正确
  const orders = await db.getOrdersByUserIds(users.map(u => u.id));
  ```

──────────────────────────────────────────────────────────────

[P2] 💡 代码规范
  位置: src/user.service.ts:102
  问题: 变量命名不够清晰
  建议: `userList` → `activeUsers`

──────────────────────────────────────────────────────────────

╠══════════════════════════════════════════════════════════════╣
║ 总结: 1个P0, 2个P1, 3个P2                                   ║
║ 建议: 修复P0后合并                                           ║
╚══════════════════════════════════════════════════════════════╝
```

### 3.2 JSON 输出

```json
{
  "review": {
    "timestamp": "2026-03-05T15:00:00Z",
    "files": ["src/user.service.ts"],
    "duration_ms": 2300
  },
  "issues": [
    {
      "severity": "P0",
      "type": "security",
      "file": "src/user.service.ts",
      "line": 45,
      "message": "SQL拼接存在注入风险",
      "suggestion": "使用参数化查询"
    },
    {
      "severity": "P1", 
      "type": "performance",
      "file": "src/user.service.ts",
      "line": 78,
      "message": "N+1 查询问题",
      "suggestion": "使用批量查询"
    }
  ],
  "summary": {
    "P0": 1,
    "P1": 2,
    "P2": 3
  }
}
```

---

## 四、Prompt 模板

### 4.1 主审查 Prompt

```markdown
你是一个资深代码审查专家。请对以下代码进行全面审查：

审查维度：
1. **代码规范**：命名、格式、注释、代码风格
2. **逻辑正确性**：边界条件、空指针、线程安全
3. **性能隐患**：循环优化、缓存使用、N+1查询
4. **安全漏洞**：SQL注入、XSS、CSRF、敏感信息
5. **错误处理**：异常捕获、日志记录、返回值

代码：
```{language}
{code}
```

输出要求：
- 对每个问题标注严重级别：P0(阻塞)/P1(严重)/P2(建议)
- 给出具体位置：文件名:行号
- 提供修复建议和示例代码
- 用Markdown格式输出
```

### 4.2 安全专项 Prompt

```markdown
请对以下代码进行安全审查，重点检测：

1. SQL注入
2. XSS跨站脚本
3. 命令注入
4. 敏感信息泄露
5. 权限校验缺失
6. 加密算法使用

代码：
```{language}
{code}
```

对于每个安全问题，请说明：
- 漏洞类型
- 风险等级
- 修复方案
```

---

## 五、技术实现

### 5.1 CLI 命令结构

```
openclaw
├── review          # 代码审查
│   ├── --path      # 审查路径
│   ├── --staged   # 仅暂存区
│   ├── --compare  # 分支比较
│   ├── --format   # 输出格式
│   ├── --check   # 检查类型
│   └── --output  # 输出文件
├── chat            # 对话模式
├── generate        # 代码生成
└── docs            # 文档生成
```

### 5.2 集成 Git Hook（可选）

```bash
# 安装 pre-commit hook
openclaw install hook

# 每次 git commit 前自动审查
```

```yaml
# .openclaw/hooks/pre-commit.yaml
enabled: true
commands:
  - openclaw review . --staged
blocking: true
severity_threshold: P0  # P0问题阻止提交
```

---

## 六、使用场景

### 6.1 日常开发

```bash
# 1. 写完代码
$ vim src/user.service.ts

# 2. 本地测试通过
$ npm test

# 3. 提交前审查
$ openclaw review .
# 输出审查结果...

# 4. 确认无误
$ git add .
$ git commit -m "feat: add user service"
$ git push
```

### 6.2 分支对比

```bash
# 审查新功能分支 vs main
$ openclaw review feature/new-api --compare main

# 仅看新增代码
$ openclaw review . --staged
```

---

## 七、与现有流程的结合

### 7.1 当前流程

```
Claude写代码 → 本地测试 → git commit → git push
```

### 7.2 改进后流程

```
Claude写代码 → 本地测试 → openclaw review . → 确认 → git commit → git push
                              ↑
                          新增环节
```

### 7.3 增量成本

| 步骤 | 耗时 | 增量 |
|------|------|------|
| Claude编码 | 30min | - |
| 本地测试 | 5min | - |
| **openclaw review** | **30s** | **+10%** |
| git push | 10s | - |

**增量成本**：约30秒，增加10%时间，换取代码质量提升

---

## 八、试点建议

### 8.1 试点范围
- 1个后端团队（5-8人）
- 2个前端团队（可选）

### 8.2 推行节奏

| 周 | 内容 |
|----|------|
| 1 | 安装CLI工具，熟悉命令 |
| 2 | 每日开发使用，收集反馈 |
| 3 | 优化Prompt，反馈迭代 |
| 4 | 总结效果，推广 |

### 8.3 成功指标

- 使用率：60% → 80%
- P0问题发现率：提升50%
- 合并后返工率：降低30%

---

## 九、下一步

- [ ] CLI工具安装配置
- [ ] 试点团队
- [ ] 收集反馈优化Prompt

---

*待执行*
