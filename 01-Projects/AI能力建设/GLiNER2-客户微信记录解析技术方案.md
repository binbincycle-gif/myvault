# GLiNER2 客户微信记录解析 - 技术方案

**场景**：从销售与客户的微信聊天记录中自动提取关键信息，转化为结构化数据存入CRM

**目标**：替代销售手工录入，将非结构化聊天记录转化为标准化客户画像

---

## 一、业务需求

### 输入
- 销售与客户的微信聊天记录（文本）
- 格式：纯文本或JSON（包含发送人、消息内容、时间）

### 输出
| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| 客户姓名 | str | 客户姓名或称呼 | 张总 |
| 公司名称 | str | 客户公司 | 某某科技 |
| 公司规模 | str | 人数/营收 | 200人/5000万 |
| 预算金额 | str | 预算范围 | 50万 |
| 产品需求 | str | 感兴趣的产品 | CRM系统/SCRM |
| 部署方式 | str | 私有化/云服务 | 私有化部署 |
| 决策人 | str | 决策人姓名 | 李总 |
| 期望上线 | str | 时间要求 | 3月份 |
| 痛点 | str | 客户提出的问题 | 现有系统不稳定 |
| 竞品信息 | str | 提及的竞品 | 销售易/纷享销客 |

### 业务价值
- 销售录入效率提升：预计从15分钟/条 → 2分钟/条
- 数据完整性：强制提取关键字段，减少漏填
- 后续可用性：结构化数据可用于线索评分、需求预测

---

## 二、技术架构

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  微信聊天记录     │ ──▶ │  GLiNER2 模型   │ ──▶ │  CRM API       │
│  (非结构化文本)   │     │  (本地推理)      │     │  (结构化写入)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### 组件

| 组件 | 技术选型 | 说明 |
|------|----------|------|
| 输入层 | Webhook / API | 接收微信聊天记录 |
| 处理层 | GLiNER2 (base-v1) | 本地推理，无需GPU |
| 输出层 | REST API | 写入CRM系统 |

---

## 三、Schema 定义

```python
schema = {
    "customer": [
        "name::str::客户姓名或称呼",
        "company::str::客户公司名称",
        "scale::str::公司规模，如'200人'或'5000万营收'",
        "budget::str::预算金额，如'50万'、'10-20万'",
        "product_interest::str::感兴趣的产品或服务",
        "deployment::str::部署方式偏好，如'私有化'、'SaaS'、'混合'",
        "decision_maker::str::决策人姓名或职位",
        "timeline::str::期望上线时间，如'3月份'、'Q2'",
        "pain_point::str::客户提及的问题或痛点",
        "competitor::str::提及的竞品或替代方案"
    ]
}
```

### 示例

**输入**：
> 客户张总通过了我的好友申请。他说他们公司叫华强科技，有280人左右，年营收大概8000万。现在用的系统不太好用，经常掉线，老板想换一套。他们预算大概60万左右，想了解一下CRM。之前也看过销售易，但觉得功能太复杂。最好能上半年上线。

**输出**：
```json
{
  "customer": [{
    "name": "张总",
    "company": "华强科技",
    "scale": "280人/8000万营收",
    "budget": "60万",
    "product_interest": "CRM",
    "deployment": "",
    "decision_maker": "老板",
    "timeline": "上半年",
    "pain_point": "现有系统不好用，经常掉线",
    "competitor": "销售易"
  }]
}
```

---

## 四、部署方案

### 环境要求

| 项目 | 规格 |
|------|------|
| 服务器 | 8核CPU + 16GB内存 |
| 存储 | 2GB（模型文件） |
| 网络 | 内网（数据不外出） |
| 操作系统 | Linux / macOS |

### 安装

```bash
pip install gliner2 torch
```

### 服务代码

```python
from flask import Flask, request, jsonify
from gliner2 import GLiNER2
import os

app = Flask(__name__)

# 全局加载模型（启动时加载一次）
print("Loading GLiNER2 model...")
extractor = GLiNER2.from_pretrained("fastino/gliner2-base-v1")
print("Model loaded successfully")

schema = {
    "customer": [
        "name::str::客户姓名或称呼",
        "company::str::客户公司名称",
        "scale::str::公司规模",
        "budget::str::预算金额",
        "product_interest::str::感兴趣的产品",
        "deployment::str::部署方式偏好",
        "decision_maker::str::决策人",
        "timeline::str::期望上线时间",
        "pain_point::str::客户痛点",
        "competitor::str::竞品信息"
    ]
}

@app.route('/extract', methods=['POST'])
def extract():
    data = request.json
    text = data.get('text', '')
    
    if not text:
        return jsonify({'error': 'No text provided'}), 400
    
    result = extractor.extract_json(text, schema)
    return jsonify(result)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### 启动

```bash
python gliner2_service.py
# 首次启动需下载模型（约800MB），后续启动约10秒
```

---

## 五、与CRM集成

### 方案A：Webhook触发

```javascript
// 微信企业版或其他支持Webhook的系统
fetch('http://<内网IP>:5000/extract', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
        text: "客户张总说他们公司..."
    })
})
.then(res => res.json())
.then(data => {
    // 调用CRM API写入结构化数据
    crmApi.createLead(data.customer[0]);
});
```

### 方案B：销售手动触发

- 在CRM页面添加"AI解析"按钮
- 销售粘贴聊天记录 → 点击解析 → 确认后写入

---

## 六、准确率评估

### 测试数据（50条）

| 字段 | 准确率（预估） | 需人工确认 |
|------|--------------|-----------|
| 客户姓名 | 90% | 是 |
| 公司名称 | 85% | 是 |
| 预算金额 | 80% | 是 |
| 产品需求 | 88% | 否 |
| 痛点 | 82% | 是 |

**建议**：所有提取结果需销售确认后写入CRM，作为"AI辅助+人工审核"模式

---

## 七、成本估算

| 项目 | 成本 |
|------|------|
| 服务器 | 现有服务器即可，无需额外采购 |
| 模型推理 | 0元（本地CPU推理） |
| 维护 | 模型更新约每季度1次 |
| 人力 | 开发和测试约3人天 |

---

## 八、推进计划

| 阶段 | 时间 | 内容 |
|------|------|------|
| PoC | 1周 | 本地部署+50条数据测试 |
| 试运行 | 2周 | 销售团队小范围试用 |
| 正式上线 | 1周 | 集成CRM+全量推广 |

---

## 九、风险与防护

| 风险 | 防护 |
|------|------|
| 隐私泄露 | 纯本地部署，数据不出网 |
| 误识别 | 人工审核机制兜底 |
| 响应速度 | 首条响应<3秒（预加载模型） |

---

**联系人**：Cycle
**日期**：2026-03-06
