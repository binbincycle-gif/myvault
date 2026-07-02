---
title: "Agent 为什么会\"自己干活\"？一文讲透 Agent Loop"
source: "https://mp.weixin.qq.com/s/jw5RG3mN8pAYOsyQBYHYdg"
author:
  - "[[二曲线工程师]]"
published:
created: 2026-06-30
description:
tags:
  - "clippings"
---
二曲线工程师 工程师的第二曲线 *2026年6月3日 18:55*

二曲线工程师

读完需要

4

分钟

速读仅需 2 分钟

## /Agent 为什么会"自己干活"？一文讲透 Agent Loop/

你有没有注意到，当你把一个任务丢给 Cursor、Claude Code，或者国内的 Kimi Code CLI、Qwen Code、Trae，它不会只给你一段代码就完事。

它会先读文件，再改代码，然后跑测试，发现测试挂了，再去分析报错，再改，再跑……几分钟后，问题解决了。

你没有一步一步指挥它。它自己就干完了。

这背后到底发生了什么？

答案是： **Agent Loop** 。

1

**Agent 和 ChatGPT，差在哪里**

要理解 Agent Loop，先得明白 Agent 和普通 ChatGPT 有什么本质区别。

普通的 ChatGPT 是这样工作的：

```
1用户提问 → LLM → 输出回答 → 结束
```

在这种传统聊天模式下，每次对话都是独立的。LLM 模型只负责根据当前输入生成一段回答。它不会主动去读文件、跑命令，也不会根据外部执行结果继续调整下一步行为。

Agent 不一样：

```
1用户提问 
2→ LLM → 决定行动 → 调用工具 → 获取结果 
3→ LLM → 再决定 → …… → 任务完成
```

最关键的变化是多了一个 **循环** ：LLM 不只是输出文字，而是可以调用工具，看到工具返回的结果，然后再次思考，再次行动。

这个循环，就是 Agent Loop。

Agent 之所以能"自己干活"，靠的就是这个循环不断地转下去。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xmGIAYiaJQ29xErRdQ47sicRgVkkZOOKwCHX7w6Em1NWLMdgh7tXCBO3SHibrTbTmm3HJNEY7wH9ZlgrLtCWTA4uXUaYvbXSxRdzOtE0g1l6EY/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)

2

**Agent Loop 长什么样**

理解 Agent Loop，最经典的入口是一篇 2022 年的论文—— **ReAct** 。

> 关于作者: 姚顺雨，清华姚班出身，普林斯顿计算机博士。毕业后加入 OpenAI，参与了 Operator、Deep Research 的核心研发。2025 年底出任腾讯首席 AI 科学家，入选 MIT 科技评论"35 岁以下科技创新 35 人"中国区名单，是其中最年轻的一位。

ReAct 把 Agent Loop 拆解成三个步骤，循环往复：

**Reasoning（推理）** ：现在情况是什么？我下一步该做什么？

**Acting（行动）** ：执行一个具体的动作，比如搜索、读文件、调接口。

**Observation（观察）** ：动作执行完了，结果是什么？

然后回到 Reasoning，开始下一轮。

用人话说就是： **想清楚 → 动手干 → 看结果 → 再想清楚** 。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xmGIAYiaJQ28vMQLSMMQ7rOicdVC0piaeIcDhel0tWag1ckeXRD6CDreLe90XK6GwrbIncEobzG3KkWZIYmnTuW6BKCiakadias5ic2vIo6bylEAU/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1)

这个循环听起来简单，但它是今天很多 Agent 系统背后的共同逻辑：比如 LangGraph、OpenAI Agents SDK 这样的开发框架，以及 Claude Code 这样的编码 Agent 产品。

3

**一个真实的例子**

光说概念不够直观，来看一个具体的例子。

假设线上出现了报错，你把一段错误日志丢给 Agent，让它帮你排查原因。它不会一次性给你答案，而是一轮一轮地推进：

**第 1 轮**

- Reasoning：先看看报错信息是什么
- Action：读取错误日志
- Observation： `NullPointerException` ，出在 `UserService.getProfile()` 第 42 行

**第 2 轮**

- Reasoning：去看看这段代码
- Action：读取 `UserService.java` 第 42 行附近
- Observation：代码里没有判断 `user` 是否为 null，直接调用了 `user.getId()`

**第 3 轮**

- Reasoning：为什么 user 会是 null？去看调用方
- Action：搜索 `getProfile` 的调用位置
- Observation：发现调用方传入了一个不存在的 userId，数据库查询返回了 null

**第 4 轮**

- Reasoning：根因找到了，整理结论
- Action：生成根因分析报告
- Observation：输出完成

**结束。**

整个过程，Agent 自己决定每一步做什么，根据上一步的结果决定下一步的行动。你只是在最开始丢了一段报错日志。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xmGIAYiaJQ29GSLXK3zv32hKibiaVgrumVw6EteicdF2amdT3kHeX89m9nmAo7BVs89TEpuVnabqy0aEbB0Ch5g2cUX5MAUJQQuWe27Xicyic0zR8/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2)

4

**Loop 什么时候停**

Agent Loop 不会无限转下去。它有终止条件。

常见的有三种：

**1\. 模型主动宣布完成** ：LLM 判断任务已经达成，不再输出 Action，直接给出最终回答。这是最理想的情况。

**2\. 达到最大轮数** ：作为兜底机制，防止 Agent 陷入死循环。OpenAI Agents SDK 里这个参数叫 `max_turns` ，默认是 10。

**3\. 工具调用出错** ：某个工具抛出了异常，或者返回了无法处理的结果，Agent 中止并报告错误。

这三种终止逻辑在工程实现里都要显式处理，是 Agent 开发绕不过去的基础设计。

5

**Loop 本身不是难点**

等你真正动手实现一个 Agent Loop，会发现：核心逻辑其实很简单。

```
1while not done:
2    action = llm.think(context)
3    result = tool.run(action)
4    context.update(result)
```

一个下午就能写出来。

真正难的部分在别处： **每次调用 LLM 时，"context"里该放什么？**

第 1 轮的日志要不要传？第 3 轮的中间结果要不要保留？随着 Loop 不断执行，传给模型的上下文会越来越长，最终超出 Token 限制。

这就是所谓的 **Context Engineering** 问题——怎么在有限的窗口里，把最有用的信息喂给模型。

这是 Agent 工程里真正有技术含量的部分，也是下一篇要专门讲的内容。

6

**最后说一句**

理解了 Agent Loop，你就理解了所有 Agent 框架的核心抽象。

不管是 LangGraph 的节点流转，OpenAI Agents SDK 的 `Runner.run()` ，还是 Claude Code 在你的终端里自动跑命令——本质上都是同一件事： **一个 LLM 在一个循环里，不断思考、行动、观察，直到任务完成。**

Agent 工程师，本质上是在给 LLM 写 Runtime 的人。

这是系列的第一篇，后面会继续拆 Context Engineering、Tool Calling 和 Agent Runtime。感兴趣可以关注。

Agent 系列 · 目录