---
title: "Agent 每次调 LLM，发的是什么 HTTP 请求"
source: "https://mp.weixin.qq.com/s/LQf6Z5SXAkOkL03UBZzwrw"
author:
  - "[[二曲线工程师]]"
published:
created: 2026-06-30
description:
tags:
  - "clippings"
---
二曲线工程师 工程师的第二曲线 *2026年6月9日 09:00*

二曲线工程师

读完需要

6

分钟

速读仅需 2 分钟

## /Agent 每次调 LLM，发的是什么 HTTP 请求？/

最近在深入做 Agent 工程，翻了很多框架源码。

有一天突然意识到一件事：我用了好几个月 SDK，写了上千行 Agent 代码，但我说不清楚每次调 LLM 的时候，底层实际发的是什么 HTTP 请求，请求体长什么样，响应体里哪个字段决定了 Agent 的下一步动作。

这个盲区让我有点不安。作为后端工程师，如果我不清楚一个系统最核心的 IO 边界，我就不算真正理解这个系统。

这篇文章就是我补完这块认知之后想记录的东西： **Agent 框架帮你封装了什么，LLM API 本身只做了什么，边界在哪里。**

**先说清楚：LLM API 规范是谁定的**

现在 Agent 开发里，最常见的 **LLM API** 规范来自两家： **OpenAI** 和 **Anthropic** 。

这里说的「规范」，不是某个标准组织发布的 RFC，而是事实上的请求/响应格式：大家都按这个形态设计 SDK、适配模型、封装 Agent 框架。

为什么是这两家？最直接的原因是，它们长期处在头部模型梯队，模型能力强，开发者用得多，所以 API 设计自然会反过来影响整个 LLM 应用开发生态。

OpenAI-compatible 格式扩散最广，很多模型服务、SDK、Agent 框架都会优先适配；Anthropic Messages API 则在 Claude 和工具调用场景里很有代表性。

OpenAI 这边有两套接口：Chat Completions 在大模型应用开发里很早普及，很多商业模型和本地推理服务都提供兼容这套格式的接口，比如 DeepSeek、Qwen、Kimi 以及不少 Ollama / vLLM 部署出来的本地模型服务。你写的代码换个 base\_url 就能切换模型，它也就逐渐成了生态里的事实标准；官方现在重点推的是 Responses API，更适合 agentic 场景，内置工具和连续上下文支持更完善。

Anthropic 则有自己的 Messages API，结构和 OpenAI 不同，尤其在 `content block` 、 `tool_use` 、 `tool_result` 这些工具调用细节上有一套独立约定。

所以作为 Agent 工程师，这两套规范都得懂： **OpenAI-compatible 格式** 决定了你能不能复用生态里大多数模型， **Anthropic Messages API** 决定了你能不能用好 Claude 的原生能力，也让你真正理解「规范」本身是什么，而不是只会调一家的 SDK。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xmGIAYiaJQ2ib5IFnZ7GNOoyuBMzpVFaxEkt02Xtkzz3gf6okSNCCuUoVDKIwrv1KNSwPGqIu3nzqTl42ZJm833o7bgDGnIR1IZ8sFEia5jk48/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)

**LLM API 只做一件事**

说得直白一点，LLM API 最底层的接口约定只有一条：

> 给我这次需要的上下文，我还你这次的响应。

至少在 Chat Completions 和 Anthropic Messages 这类基础调用里，它默认不替你维护业务会话状态。多轮对话、工具调用、记忆——这些听起来像是模型的能力，但背后的工作量其实是这样分的： **多轮状态由你维护** ； **工具调用的结构化输出能力由模型提供** ，但客户端工具真正怎么执行、结果怎么回填，是你的 Runtime 完成的。

拿 Anthropic 的 API 举例，一次最简单的请求长这样：

```
1POST https://api.anthropic.com/v1/messages
2x-api-key: sk-ant-...
3anthropic-version: 2023-06-01
4Content-Type: application/json
5
6{
7  "model": "claude-...",
8  "max_tokens": 1024,
9  "system": "你是一个 SRE 故障排查助手。",
10  "messages": [
11    { "role": "user", "content": "api-gateway CPU 飙到 95%，帮我排查" }
12  ]
13}
```

就这些。模型处理完，还你一个响应。对于这类请求来说，如果你没有把上一轮历史放进来，它就不知道上一轮发生了什么，也不知道你下一步打算怎么办。

**那"多轮对话"是谁做的？**

多数情况下，是你做的。

Chat Completions 和 Anthropic Messages 默认不替你维护会话状态。HTTP 本身就是无状态协议，这是底层基础；但更关键的是，这类 LLM API 的接口设计也没有替你维护业务会话。所谓「多轮」，就是每次把模型需要的历史重新组织后发过去——可能是完整历史，也可能是裁剪后的最近消息加上工具调用结果。

OpenAI Responses API 里有 `previous_response_id` 这类能力，可以减少你手动拼接历史的工作量。但这不改变一个工程事实：业务状态、工具结果、权限边界、重试策略，仍然不能默认交给模型或 API 平台替你管理。

用户说「刚才那个服务重启之后还是报错」，模型能理解「刚才」指什么，不是因为它记住了，而是因为你把前面的对话塞进了 `messages` 数组：

```
1{
2  "messages": [
3    { "role": "user",      "content": "api-gateway CPU 飙到 95%" },
4    { "role": "assistant", "content": "我先查一下错误日志。" },
5    { "role": "user",      "content": "刚才那个服务重启之后还是报错" }
6  ]
7}
8
```

这就是为什么 Agent 框架里总有一个 `memory` 或者 `history` 的概念—— **它是你的代码在维护这个数组，不是模型在记忆** 。

而且这里有个很多人忽略的细节： **工具调用的历史也要存** 。

不能只存文字对话，还要存「模型上次请求调哪个工具」「工具返回了什么结果」。否则模型下一轮完全不知道自己刚才做了什么。

**那"工具调用"是谁执行的？**

也是你。

模型本身不能直接执行你系统里的函数。它能做的是——在某个时刻，返回一个结构化的 tool call 对象，你可以把它理解成一张「工具调用申请单」：我想调这个工具，参数是这些，请你去执行。

Anthropic 的响应长这样：

```
1{
2  "stop_reason": "tool_use",
3  "content": [
4    { "type": "text",     "text": "我来查一下日志。" },
5    { "type": "tool_use", "id": "toolu_abc", "name": "query_logs",
6      "input": { "service": "api-gateway", "level": "ERROR" } }
7  ]
8}
```

`stop_reason` 是 `tool_use` ，意思是：我停在这了，等你执行完工具把结果给我。

然后你的代码去真正调 `query_logs` ，拿到结果，再把结果拼回 `messages` ，再发一次请求：

```
1{
2  "messages": [
3    "...",
4    { "role": "assistant", "content": [ "...上面那条 tool_use..." ] },
5    { "role": "user", "content": [
6      { "type": "tool_result", "tool_use_id": "toolu_abc",
7        "content": "{\"logs\": [...]}" }
8    ]}
9  ]
10}
```

这一圈走下来，才算完成了一次客户端工具调用。 **模型负责生成调用申请，真正干活的是你的代码。**

HITL 审批、guardrails 校验、工具执行超时重试——这些都是卡在「模型发出申请」和「工具真正被执行」之间的工程逻辑，全部是你的 Runtime 在做，不是一次 LLM API 请求自动替你完成的。

**那 LLM API 这一层，你需要直接打交道的有哪些？**

梳理下来，生产 Agent 常见会遇到七件事：

**① 系统指令** —— 生产 Agent 通常会在请求中带上稳定的系统/开发者指令，告诉模型它是谁、能做什么。Anthropic 放在顶层 `system` 字段，OpenAI 在不同接口里有不同写法，比如 Chat Completions 里的 `role: "system"` / `"developer"` ，Responses API 里的 `instructions` 或 `input` 。格式不同，写错直接报错。

**② Messages 历史** —— 每次请求把模型需要的历史组织好发过去，包括工具调用记录。这是「多轮对话」的真相。

**③ Tools Schema** —— 在允许模型调用工具的请求中，需要提供当前可用工具的 JSON Schema 定义。字段名两家不同：OpenAI 叫 `parameters` ，Anthropic 叫 `input_schema` 。

**④ Response 解析** —— 收到响应后先判断模型想干什么：Anthropic 看 `stop_reason` 和 `content` block 类型；OpenAI Chat Completions 看 `finish_reason` 和 `tool_calls` 。这是 Agent Loop 里最核心的判断点。

**⑤ 工具结果回填** —— 工具跑完，结果必须按规范格式塞回 `messages` 再发请求。OpenAI 用 `role: "tool"` 的消息，Anthropic 用 `role: "user"` 里的 `tool_result` block， **两家完全不同，最容易写错。**

**⑥ Streaming 解析** —— 开启 `stream: true` 后，响应变成 SSE 事件流，工具调用的参数是分片推送的，必须拼接完整再 `JSON.parse` ，不能拿到一片就解析。

**⑦ Structured Output** —— 需要模型输出 JSON 时，OpenAI Chat Completions 常用 `response_format.json_schema` 约束，Responses API 则是 `text.format` 里的 `json_schema` 。Anthropic 的常见做法是定义一个输出工具，用 `tool_choice` 强制模型调用，从而得到结构化结果。

这七件事，框架帮你包了一层，但包裹不代表消失。出问题的时候，你必须知道底下在发生什么。

**Agent Runtime 在 API 之上加了什么**

理解了基础 LLM API 主要负责「一次推理」，那 Agent Runtime 做的事就清晰了：

**上下文工程** ——历史长度裁剪、摘要压缩、敏感信息脱敏、RAG 检索结果注入。这些都是在组装入参之前发生的。

**执行控制** ——工具白名单校验、guardrails 检查、HITL 暂停恢复、Trace 记录、错误重试、fallback 模型。这些都是在解析出参之后、工具执行前后发生的。

它们不是基础推理请求天然自带的能力，而是你的工程代码在 API 调用前后包裹出来的控制逻辑。

**一句话总结**

对 Chat Completions 和 Anthropic Messages 这类基础调用来说，LLM API 更像一个无状态的接口：你发一个请求，它还你一个响应；下次再来时，你需要自己把必要上下文带上。

Agent 所有的「智能」——记忆、工具、多轮、状态、安全——都是你（或者框架）在这个接口之上堆起来的工程。

**理解这条边界，你才真正算是在做 Agent 工程，而不是在调包。**

#Agent开发 #大模型应用 #AI工程师 #后端开发 #LLM #HTTP

Agent 系列 · 目录