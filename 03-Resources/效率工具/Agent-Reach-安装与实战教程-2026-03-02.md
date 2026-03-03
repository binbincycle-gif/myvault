# Agent-Reach 安装与实战教程（可对外分享）

> 更新时间：2026-03-02  
> 适用对象：想让 AI 助手具备“可读网页/搜平台/抓视频/读社媒”的团队与个人  
> 项目地址：https://github.com/Panniantong/Agent-Reach

---

## 1. 这是什么（1分钟看懂）

**Agent-Reach** 不是新的大模型，也不是替代你的 Agent。  
它是一个“互联网能力脚手架”：帮你把常见渠道能力（网页、YouTube、GitHub、X/Twitter、小红书等）快速接好，让 AI 助手能直接调用。

一句话：**给你的 Agent 装上“看网、搜网、读网”的眼睛。**

---

## 2. 能做什么（按实战价值）

### 开箱即用（无需复杂配置）
- 网页阅读（Jina Reader）
- YouTube/B站信息提取（yt-dlp）
- RSS 读取
- 全网语义搜索（Exa via MCP）

### 配置后可用（常用于运营/情报）
- GitHub（仓库/Issue/搜索）
- X/Twitter（读取/搜索，Cookie 模式）
- 小红书（搜索、详情读取，MCP）
- 抖音、LinkedIn、Boss 直聘（按对应 MCP/工具）

---

## 3. 设计理念（为什么它适合团队）

- **脚手架而不是封装平台**：底层工具可替换，不被单一方案绑死。
- **免费优先**：尽量走开源工具和免 Key 能力。
- **本地凭据**：Cookie/Token 存本地配置，不走平台托管。
- **可诊断**：`agent-reach doctor` 一眼看哪些通、哪些不通。

---

## 4. 标准安装流程（对外分享版）

### Step 1：准备环境
- Python 3.10+（推荐 3.11）
- Node.js（用于部分 CLI）
- Git、Docker（涉及 MCP 服务时）

### Step 2：安装
```bash
pip install agent-reach
agent-reach install --env=auto
```

### Step 3：诊断
```bash
agent-reach doctor
```

### Step 4：按需配置渠道
- GitHub：登录 gh CLI
- X/Twitter：导出 Cookie 后配置
- 小红书：部署 xiaohongshu-mcp 并登录

---

## 5. OpenClaw 场景推荐接法

如果你已经在用 OpenClaw，建议把 Agent-Reach 作为“外部情报层”：

- 主会话负责“决策与输出”
- Agent-Reach 负责“抓取与搜索”
- 定时任务（cron）负责“日报/周报”

**推荐链路：**
1) 定时触发
2) 调用 Agent-Reach 抓取平台信息
3) 汇总成管理层可读版本（结论→动作→风险）
4) 自动发送到指定渠道

---

## 6. 我们这次实战踩坑记录（可复用）

### 坑1：Python 版本不满足
- 现象：`requires Python >=3.10`
- 处理：升级到 Python 3.11，并用独立虚拟环境运行
- 建议：团队统一 Python 版本，避免环境碎片化

### 坑2：小红书“扫码后仍未登录”
- 根因：容器未正确持久化 cookies
- 处理：本地登录工具生成 `cookies.json`，挂载进容器并显式指定 `COOKIES_PATH`
- 结果：登录状态稳定为 true

### 坑3：参数不符合 MCP Schema
- 现象：`invalid params / additional properties`
- 处理：先查 schema，再严格按字段调用（如 `keyword + filters`）
- 建议：先“最小合法参数”跑通，再加筛选条件

---

## 7. 安全与风控建议（务必加在对外分享里）

1. **Cookie 类平台使用专用小号**，避免主号风险。  
2. 凭据文件权限最小化（仅当前用户可读写）。  
3. 服务器环境使用 `--safe` / `--dry-run` 先预演。  
4. 把“抓取能力”与“发送能力”分层，避免误发。  
5. 对外内容增加来源校验，防止错误信息扩散。

---

## 8. 给团队的落地建议（两周）

### 第1周：打通能力
- 完成安装与 doctor 全绿（至少关键渠道）
- 跑通 2 个高价值场景（如：竞品情报、内容选题）

### 第2周：形成产出
- 固化日报模板
- 增加定时任务
- 明确 owner 与异常处理SOP

验收标准：
- 每天稳定产出
- 出错可追踪
- 一线能直接使用

---

## 9. 一句话结论

**Agent-Reach 适合做“AI互联网能力底座”：低成本、可替换、可扩展。先跑通关键场景，再标准化扩面。**
