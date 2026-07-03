---
title: "【总结篇】13 篇 Agent 工程系列，我用一张图把它们串起来了"
source: "https://mp.weixin.qq.com/s/NT-EuEprv4ctcMIgXprZDA"
author:
  - "[[二曲线工程师]]"
published:
created: 2026-07-02
description: "Agent 工程化系列总结篇"
tags:
  - "clippings"
---
二曲线工程师 工程师的第二曲线 *2026年6月15日 11:23*

二曲线工程师

读完需要

6

分钟

速读仅需 2 分钟

## /【总结篇】13 篇 Agent 工程系列，我用一张图把它们串起来了/

前面 13 篇技术文章，已经从 Agent Loop 一路讲到了 Guardrail。每一篇都是一个独立的主题，但我一直有个担心：读者看完每篇都懂了，但把它们放在一起，未必清楚这些东西是怎么组织在一起的。

这篇就来做这件事——不重复每篇的内容，而是把 13 个主题串成一张地图。

**一、Agent 工程化，到底在解决什么问题**

在拆每个模块之前，先退一步看全局。

Agent 工程化要解决的问题，用一句话概括是：

**让 LLM 从"能对话"变成"能干活"，从"能干活"变成"敢用"。**

这三个阶段，对应三类不同的工程问题：

**1、"能干活"** ：

需要 [Agent Loop](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483749&idx=1&sn=b94484ecf1bbad2b028998cba76f59dd&scene=21#wechat_redirect) 、 [Tool Calling](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483782&idx=1&sn=10d24a8eedff443f368d905f1781ad6a&scene=21#wechat_redirect) 、 [Agent Runtime](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483823&idx=1&sn=784a88223c98a2e38f43b3314a80dd67&scene=21#wechat_redirect) 、 [Planning](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483911&idx=1&sn=3ef4644f900c11e40355af8aa66513b7&scene=21#wechat_redirect) 、 [Multi-Agent](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483909&idx=1&sn=f167a0eb993611147a07d91ab85dcc78&scene=21#wechat_redirect) ——这些是让 Agent 具备行动和执行能力的基础设施。

**2、"干得好"** ：

需要 [Context Engineering](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483758&idx=1&sn=0d10b5cf5b3c4b659b92a86586ae1ac7&scene=21#wechat_redirect) 、 [Memory](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483828&idx=1&sn=40a279c4de35708256f5869e1c4f8f50&scene=21#wechat_redirect) 、 [RAG](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483912&idx=1&sn=dba8f6e0fc1388d73b599311551f9c5b&scene=21#wechat_redirect) 、 [Prompt Engineering](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483913&idx=1&sn=374ef3ff8b490af927fc69bcdc3b0681&scene=21#wechat_redirect) ——这些决定 Agent 每一步推理的信息质量和行为质量。

**3、"敢用"** ：

需要 [Trace](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483829&idx=1&sn=f8e45ac056a14f24d28389b6adcb7933&scene=21#wechat_redirect) 、 [Eval](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483862&idx=1&sn=7439bf0b7151a21ee6441e0712bab583&scene=21#wechat_redirect) 、 [HITL](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483858&idx=1&sn=745f3df5514e0fe0a87a40034f0522d5&scene=21#wechat_redirect) 、 [Guardrail](https://mp.weixin.qq.com/s?__biz=MzU1NTM5MjQ0NQ==&mid=2247483915&idx=1&sn=06b87c32a22a8b52b737d00243f67151&scene=21#wechat_redirect) ——这些让 Agent 的行为可见、可测、可控。

13 个主题，本质上都在回答这三个问题的某一个。

**二、一张全景图**

**![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/xmGIAYiaJQ2icM8Snfrj5rwO0hW0mAeEuTxictIGYkVXTS5CmibZBLOvZZs81rEOFFfYd9WwQFRd0W0MO8VLTCI9hsWV37KAlWBeCp1biapRQ9UA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)**

把 13 个主题按照功能归类，可以得到五个层次：

**1、执行层** ： Agent Loop + Planning

Agent 怎么动起来、怎么想清楚再动手。

Loop 是基本运行机制，Planning 是面对复杂任务时的全局规划能力。没有这一层，Agent 只是一个问答系统。

**2、信息层** ：Context Engineering + Memory + RAG

Agent 每次推理能看到什么。

Context Engineering 决定窗口里放什么；Memory 决定历史信息怎么沉淀和调取；RAG 决定外部知识怎么按需注入。这一层的质量，决定了 Agent 推理的上限。

**3、能力层** ：Tool Calling + Multi-Agent

Agent 能做什么。

Tool Calling 是单个 Agent 伸向外部世界的手；

Multi-Agent 是多个 Agent 协作，解决单个 Agent 承载不了的复杂任务。

**4、运行与控制层** ：Agent Runtime + Prompt Engineering

Runtime 负责调度和执行——驱动 Loop、管理状态、协调工具、记录 Trace；

Prompt Engineering 负责向模型传递行为指令——塑造角色、约定格式、划定边界。前者提供硬执行机制，后者提供软行为引导。

两者性质不同，但共同决定了 Agent 怎么跑、怎么被约束。

**5、保障层** ：Trace + Eval + HITL + Guardrail

Agent 出了问题怎么查、质量怎么量化、高风险操作怎么拦截。

这一层决定了 Agent 能不能真正上生产。

五个层次不是独立的，而是相互依赖的：

- 执行层需要信息层提供高质量输入
- 能力层扩展了 Agent 的边界
- 运行层把所有东西串起来
- 保障层让整个系统值得信任

**三、最值得带走的三个认知**

**认知一：模型能力很重要，但信息质量往往才是 Agent 的实际瓶颈**

这是整个系列最值得反复确认的结论。

更强的模型不一定能解决错误 Context、低质量检索和混乱 Memory 带来的问题。Agent 的最终表现，是模型能力、信息质量和工具能力共同作用的结果。但在实践中，很多人第一反应是"换个更强的模型"——而真正的瓶颈往往是信息层没做好。

把 Context 管好、把检索质量提上去、把 Memory 设计合理，同样的模型效果可能有质的提升。模型是引擎，信息是燃料。引擎再好，烧的是劣质油，也跑不快。

**认知二：Prompt 是软约束，权限、策略和 Guardrail 才是硬边界**

这是很多人搭 Agent 时会踩的坑。

在 Prompt 里写"不要做 X"，模型通常会遵守——但不保证一定遵守。

真正的边界必须在代码层面强制执行：工具白名单、参数校验、HITL 审批、Guardrail 拦截。

Prompt 告诉 Agent 应该做什么，权限系统、Runtime 策略和 Guardrail 决定 Agent 能做什么。

**认知三：Trace 和 Eval 不是锦上添花，是能不能持续迭代的基础**

没有 Trace，出了问题是黑盒，改了东西不知道是不是真的改好了。没有 Eval，优化靠感觉，上线没信心。

Agent 从"跑起来"到"跑得好"，Trace 和 Eval 是必经之路，不是可选项。

**四、一个经常被问到的问题**

**「这些东西，我从哪里开始？」**

如果你是第一次搭 Agent，建议按这个顺序：

**第一步，把 Loop 跑起来。** 不需要完美，先让 Agent 能动起来，调用一两个工具，完成一个简单任务。

**第二步，把 Trace 加上。** 在 Agent 能跑之后，立刻加 Trace。这件事越晚做，后面越痛苦。有了 Trace，你才知道 Agent 在做什么。

**第三步，把 Context 管好。** 当你发现 Agent 跑着跑着开始"忘事"、结论开始跑偏，这时候去看 Context Engineering 和 Memory，是最有效的投入。

**第四步，把 Eval 建起来。** 当你开始迭代 Prompt、换模型、调工具——你需要一个客观的标准来判断是变好了还是变差了。

**第五步，完善 Guardrail。** 从接入第一个工具开始，就要设置最小权限、参数校验和预算限制；上生产前，再补齐 HITL、敏感数据保护和完整安全策略。安全不是最后才做的事，而是从第一行工具代码就要考虑的事。

其他的——Planning、RAG、Multi-Agent——在基础打好之后，按需引入。

**五、这个系列没有讲的**

诚实地说，有几件事这个系列没有覆盖：

**1、Agent 的成本优化** ：

Token 消耗、API 费用、缓存策略——这些在生产环境里是真实的工程问题，值得单独成篇。

**2、Agent 的部署与扩容** ：

单机跑和多实例部署是完全不同的复杂度，状态管理、并发控制、任务队列——这些是 Agent Infra 层的话题。

**3、具体领域的 Agent 实践** ：

代码 Agent、客服 Agent、数据分析 Agent——每个领域有自己特有的设计模式和踩坑记录。

这些是后续可以继续写的方向。这个系列覆盖的是通用的 Agent 工程化基础，打好底座，上面的东西才能搭起来。

**最后说一句**

Agent 工程化不是一个新技术栈，而是软件工程在 AI 时代的延伸。

Loop 类似控制循环

Runtime 类似调度和执行系统

Tool Calling 类似受模型驱动的接口调用

Trace 延续了链路追踪思想

Eval 类似测试与质量体系

Guardrail 则承担策略校验和安全拦截

——如果你有后端系统的经验，这些东西不陌生，只是换了一批名词，解决了一批新问题。

这个系列到这里，算是把 Agent 工程化的基础框架讲完了。

下一篇，也是最后一篇，我想聊一件更私人的事： **一个后端工程师，为什么要、以及怎么转向 Agent 开发。**

那是另一种角度的总结。

Agent 系列 · 目录