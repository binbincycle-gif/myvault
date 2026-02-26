# Obsidian 知识库搭建攻略（可直接落地版）

## 一、目标
用最小成本搭建一套可持续的知识系统，实现：
1. 本地高效记录  
2. 团队可检索复用  
3. 自动同步到远程仓库（避免手工遗漏）

---

## 二、目录结构（先标准化）
在你的 Vault 根目录建立：

- `00-Inbox/`（临时输入）
- `01-Projects/`（项目文档）
- `02-Areas/`（长期主题）
- `03-Resources/`（外部资料）
- `04-Archives/`（归档）
- `99-Templates/`（模板）

> 规则：先入 Inbox，每周归档一次。

---

## 三、Git 接入（一次配置，长期受益）

```bash
cd "<VAULT_PATH>"
git init
git branch -M main
git remote add origin <YOUR_REMOTE_GIT_URL>
```

创建 `.gitignore`：

```gitignore
.DS_Store
.obsidian/workspace.json
.obsidian/workspaces.json
.obsidian/cache
.obsidian/snapshots
.obsidian/plugins/*/data.json
*.tmp
*.log
```

首次提交：

```bash
git add .
git commit -m "init: knowledge base"
git push -u origin main
```

---

## 四、自动同步（核心）

创建脚本 `<SCRIPTS_PATH>/obsidian-auto-sync.sh`：

```bash
#!/bin/zsh
set -e
VAULT="<VAULT_PATH>"
BRANCH="main"

cd "$VAULT" || exit 1
git add .
if ! git diff --cached --quiet; then
  git commit -m "auto-sync: $(date '+%Y-%m-%d %H:%M:%S')"
  git push origin "$BRANCH"
fi
```

授权执行：

```bash
chmod +x <SCRIPTS_PATH>/obsidian-auto-sync.sh
```

---

## 五、定时任务（每小时 + 收工）
`crontab -e` 加入：

```cron
0 * * * * /bin/zsh <SCRIPTS_PATH>/obsidian-auto-sync.sh >> <SCRIPTS_PATH>/obsidian-sync.log 2>&1
30 22 * * * /bin/zsh <SCRIPTS_PATH>/obsidian-auto-sync.sh >> <SCRIPTS_PATH>/obsidian-sync.log 2>&1
```

---

## 六、Obsidian 设置建议
- 新笔记默认位置：`00-Inbox`
- 链接格式：相对路径
- 自动更新内部链接：开启
- 模板目录：`99-Templates`

---

## 七、模板（直接可用）

### 1）会议纪要模板
```md
---
type: meeting
date:
owner:
topic:
---

## 结论
-

## 动作
1.
2.
3.

## 风险
-
```

### 2）项目方案模板
```md
---
type: plan
project:
owner:
status:
---

## 背景
## 目标
## 方案
## 里程碑
## 风险
```

### 3）周复盘模板
```md
---
type: weekly-review
week:
owner:
---

## 本周结果
## 问题与原因
## 下周动作（<=3）
```

---

## 八、治理机制（避免“越用越乱”）
1. 每周固定 30 分钟清理 Inbox  
2. 文档必须有 owner/status/date  
3. 项目结束后 48 小时内归档  
4. 每月输出一次“知识库增量摘要”

---

## 九、常见坑
- 只写不归档 → 检索失效  
- 不自动同步 → 版本丢失  
- 无模板 → 文档无法复用  
- 无权限边界 → 敏感信息泄露风险

---

## 十、验收标准（部署完成）
- [ ] 能成功 push 到远程
- [ ] 自动同步脚本可运行
- [ ] 定时任务生效
- [ ] 模板可直接调用
- [ ] 团队知道“入库—整理—复用”流程
