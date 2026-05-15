# Day07｜参数设计与 Schema 基础

## 今日主题

今天继续第二阶段：**学会把 Skill 写对。**

Day06 讲的是「描述」：让 Agent 和用户知道这个 Skill 该不该用。

Day07 讲的是「参数」：让 Skill 在被调用时，拿到足够、清晰、可校验的输入。

一句话：

> 描述决定 Skill 会不会被正确选中，参数决定 Skill 能不能稳定干活。

很多业务 Skill 不稳定，不是模型不聪明，而是参数设计太随意：该必填的没必填，该限定的没限定，该有默认值的没有默认值，导致每次调用都像临时沟通。

---

## 技术原理（技术视角）

### 1. 参数是 Skill 的输入契约

Skill 参数不是表单字段的简单搬运，而是调用契约。

它至少要回答：

```text
调用这个 Skill 必须提供什么？
哪些信息可以不填？
每个字段是什么类型？
允许哪些取值？
缺失或错误时怎么处理？
```

如果描述是「门牌」，参数就是「进门规则」。

### 2. Schema 是让参数可校验的规则

Schema 可以理解成参数说明书 + 自动检查规则。

常见要素：

| 要素 | 作用 | 例子 |
|---|---|---|
| type | 字段类型 | string / number / boolean / array / object |
| required | 必填字段 | brandId、dateRange |
| enum | 限定可选值 | day / week / month |
| description | 字段解释 | 查询的品牌ID，不是品牌名称 |
| default | 默认值 | 最近7天 |
| additionalProperties | 是否允许额外字段 | false 表示不接受未定义字段 |

### 3. 好参数要少而准

不推荐：

```json
{
  "brand": "品牌",
  "data": "数据",
  "type": "类型",
  "remark": "备注"
}
```

问题：字段太泛，Agent 不知道该填什么，后端也不好校验。

推荐：

```json
{
  "brandId": "10086",
  "dateRange": {
    "startDate": "2026-05-01",
    "endDate": "2026-05-15"
  },
  "analysisType": "member_tag_overview"
}
```

这类参数更稳定，因为它明确了：查谁、查哪段时间、查什么分析。

---

## 业务翻译（业务视角）

业务侧不关心 Schema 这个词，但会感受到参数设计的好坏。

参数设计差，业务体验是：

- 每次都要补充背景；
- 结果口径忽左忽右；
- 系统经常反问或报错；
- 同一个问题不同人问出来结果不一致。

参数设计好，业务体验是：

- 只填关键条件；
- 输出口径稳定；
- 能自动补默认值；
- 出错时知道该补什么。

所以参数不是技术细节，而是业务稳定性的入口。

---

## 典型场景（EZR 业务版）

### 场景 1：会员标签洞察 Skill

目标：帮助运营判断某品牌会员标签覆盖、变化和异常。

建议参数：

```json
{
  "brandId": "必填，品牌ID",
  "dateRange": "必填，统计时间范围",
  "storeIds": "选填，门店范围；不填默认全品牌",
  "tagCategory": "选填，标签分类；不填默认全部",
  "compareMode": "选填，是否环比/同比"
}
```

管理重点：`brandId` 和 `dateRange` 必须强约束，否则所有分析都没有口径。

### 场景 2：导购跟进建议 Skill

目标：导购在跟进会员前，拿到今日建议。

建议参数：

```json
{
  "memberId": "必填，会员ID",
  "guideId": "必填，导购ID",
  "scenario": "必填，跟进场景：复购/沉默唤醒/试衣间未成交/生日关怀",
  "channel": "选填，触达渠道：企微/短信/电话/门店面访"
}
```

管理重点：`scenario` 要用枚举值，不要让用户自由写，否则话术和推荐逻辑会漂。

### 场景 3：资源预警 Skill

目标：技术管理者判断未来两周人员负载风险。

建议参数：

```json
{
  "startDate": "必填，排期开始日期",
  "endDate": "必填，排期结束日期",
  "teamScope": "选填，团队范围；不填默认技术中心全部",
  "includeUnassigned": "选填，是否包含未分配任务，默认 true"
}
```

管理重点：时间范围必须明确，团队范围可以默认，但不能模糊。

---

## 图示

```text
业务问题
  ↓
最小必要输入
  ↓
字段类型 / 必填 / 可选值
  ↓
Schema 校验
  ↓
稳定调用 Skill
  ↓
可复盘的业务输出
```

记住一句话：

> 参数设计不是把字段列全，而是把业务判断所需的最小上下文固定下来。

---

## 管理者关注点（成本 / 效率 / 风险）

### 1. 成本

参数越清楚，后续沟通和返工越少。尤其是跨产品、研发、业务团队时，Schema 可以减少口径争议。

### 2. 效率

稳定参数能让 Agent 自动调用 Skill，不需要每次临时追问。默认值和枚举值设计得好，业务使用门槛会明显下降。

### 3. 风险

参数不清会带来三类风险：

- **口径风险**：时间、品牌、门店范围没定义，结果不可比；
- **权限风险**：只传品牌名不传身份和权限，容易越权；
- **执行风险**：营销、发券、触达类 Skill 如果缺审批参数，可能误执行。

所以凡是涉及经营动作的 Skill，参数里必须考虑：身份、权限、范围、时间、动作级别。

---

## 官方资料（带看点）

### 本地文档（知识库镜像路径）
- `/Users/cycle/.local/share/fnm/node-versions/v24.13.1/installation/lib/node_modules/openclaw/docs/tools/skills.md`  
  看点：OpenClaw Skill 的加载、优先级和使用方式，理解 Skill 作为能力资产如何被系统识别。
- `/Users/cycle/.local/share/fnm/node-versions/v24.13.1/installation/lib/node_modules/openclaw/docs/tools/llm-task.md`  
  看点：JSON Schema 如何用于结构化输出和自动化校验，可迁移理解到业务 Skill 参数设计。
- `/Users/cycle/.local/share/fnm/node-versions/v24.13.1/installation/lib/node_modules/openclaw/docs/tools/plugin.md`  
  看点：插件与工具注册会涉及 manifest / schema，适合理解更底层的工具参数契约。

### 在线文档
- https://docs.openclaw.ai/tools/skills
- https://docs.openclaw.ai/tools/llm-task
- https://docs.openclaw.ai/tools/plugin

---

## 今日练习

选一个当前最想做成 Skill 的业务能力，按下面模板写参数：

```text
Skill 名称：
必须参数：
- 字段名：
- 类型：
- 为什么必填：

可选参数：
- 字段名：
- 默认值：
- 什么时候需要填：

枚举参数：
- 字段名：
- 可选值：
- 不允许自由输入的原因：

风险参数：
- 是否涉及权限：
- 是否涉及审批：
- 是否涉及外部触达/执行动作：
```

建议今天就拿「导购跟进建议 Skill」练一版：

- 必填：memberId、guideId、scenario；
- 可选：channel、storeId、productCategory；
- 枚举：scenario、channel；
- 风险：是否允许直接触达，还是只生成建议。

---

## 本日自检

1. 这个 Skill 的必填参数是否真的不可缺？
2. 有没有把自由文本改成枚举，减少结果漂移？
3. 是否明确了时间、范围、身份、权限这些关键口径？

## 一句话总结

参数设计的核心不是“字段越全越好”，而是用 Schema 把业务判断所需的最小上下文固定下来，让 Skill 可以稳定、可控、可复盘地被 Agent 调用。
