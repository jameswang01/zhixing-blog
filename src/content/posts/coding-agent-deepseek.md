---
title: "DeepSeek Harness：插件、执行循环与会话事件"
description: "理解 Cordis 组合方式和可持久化事件的职责。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 先看结论

DeepSeek Harness 使用 Cordis 组合能力，公开架构文档把模型适配、工具注册、会话和 Agent 循环都描述为可组合组件。本文阅读提交 `39163dd`，将架构文档和关键实现分开使用。[项目架构](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)

## 一次任务经过哪些组件

从阅读路径看，先建立会话，再由 Agent 驱动 turn 和 step；执行前解析工具集合、组装请求需要的上下文，之后进入模型和工具调用。`agent-loop/src/agent.ts` 中可以看到 `turn`、`preStep`、工具解析与请求准备等代码。[Agent 循环](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts)

同一任务中，turn 可以表示一轮推进，step 可以表示更细的执行边界。具体事件和生命周期应以对应版本的类型与实现为准，不要将这些名字直接套进所有框架。

## 事件和模型上下文有什么关系

项目文档区分需要持久化的 session 事件与运行时事件；文档还说明模型历史由事件投影构造。这个区分有实际意义：界面实时看到的状态、已经持久化的事实以及本次模型收到的消息，不一定是同一个集合。[架构中的事件和会话](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md) · [会话实现](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/session/src/index.ts)

例如，工具执行后可以保存事件，再由投影生成模型可读内容。设计恢复时仍要核对外部副作用是否已经发生；保存事件本身不自动让外部系统的写操作变成幂等。

## 插件组合的取舍

把能力放到插件边界，便于替换模型、工具或存储适配；与此同时，排障时需要确认实际加载了哪些插件、配置顺序以及事件处理链。代码中提供扩展点，并不意味着每套部署都启用了相同能力。[架构文档](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)

## 尚未验证

本文没有启动全部 profile，也没有验证某一线上环境的插件配置、性能或恢复效果；这些都需要结合实际部署和测试补充。


## 先认识三个层次：会话、轮次、步骤

项目的架构文档将会话事件作为重要的持久化事实来源；Agent 代码则在运行时维护 turn 和 step 边界。[架构中的事件](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md#L67-L78) · [Agent 实现](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts#L274-L390)

为了阅读方便，可以先把 session 理解成承载一段任务历史的边界，turn 理解成一次推进过程，step 理解成推进中的细粒度阶段。但这是帮助定位的抽象，具体状态及其生命周期仍以类型和代码为准。

## preStep：正式执行前做了哪些准备

[`preStep`](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts#L289-L322) 提供了一条具体线索：

1. 从 inbox 认领当前需要处理的消息，并取得当前模型选择。
2. 若没有配置 provider/model，抛出明确错误。
3. 针对当前 Agent 和模型解析可用工具集合。
4. 组装 system prompt 等上下文，并通过 `agent/pre-step` 扩展点形成继续或拒绝的决定。
5. 如果被拒绝或准备过程中出错，释放已解析的工具集合。

这里有个重要取舍：工具集合是在执行准备阶段解析的；因此分析“为什么模型看不到某个工具”时，应检查当前 Agent、模型选择、插件和配置，而不是只确认仓库里存在这个工具。

## turn 与 step：哪里记录了执行边界

[`turn`](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts#L332-L390) 开始时向 session 追加 `turn/start`，进入 step 前追加 `step/start`；它持续检查取消信号，并处理 pre-step 决定。代码中也有关于工具结果恢复的逻辑：当 step 抛出错误时，尝试将已经观察到、尚需补记的结果写入 session。

这能说明代码对“中断时如何保留部分执行事实”有具体处理；但不能因此声称所有外部副作用都有自动补偿或恰好一次语义。工具真正触达的数据库、支付或消息系统仍有各自边界。

## 事件记录与实时事件为什么要分开

[架构文档](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md#L67-L78) 区分持久 session 事件和运行中的 Agent/能力事件。界面显示一段流式文本时，用户看到的是实时过程；判断重启后能恢复什么，要核对哪些事件已经提交以及存储实现。

一个排障例子：工具已显示开始，随后进程突然退出。应该检查开始事件是否持久化、下游是否收到请求、结果是否提交、恢复代码怎样处理未完成调用，而不是仅凭界面最后一条消息判断状态。

## 设计取舍与证据限制

插件化使组件可通过配置组合，但也让排查依赖实际加载的组件：同一个工具名在不同 profile 下可能有不同配置。本文只读了指定提交的部分循环、会话与架构资料；没有执行全部 profile，也没有验证某一部署环境。


## 用测试核对工具顺序与失败边界

[工具顺序测试](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/tests/tool-order.spec.ts#L73-L115) 构造不同的工具注册顺序，检查会话记录中的请求头和实际发送给模拟适配器的工具列表。测试还覆盖显式配置顺序，以及配置引用不存在的工具时，不发送模型请求、关闭当前 turn 且不产生 step 的场景。

这一测试回答的是确定的边界问题：**准备阶段出错时，测试要求失败发生在模型请求之前。** 它不说明线上每个插件组合都经过同样测试，也不证明模型对工具顺序变化一定有相同表现。

## 用取消测试理解“停止”不是一个瞬间

仓库的[取消测试](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/tests/cancel.spec.ts#L151-L222) 区分了空闲时取消、保留 inbox 的取消等情形。阅读这类测试时，应把“已经认领的输入”“尚未认领的排队输入”和“运行中的步骤”分开看：它们在取消后可能面临不同处理。

如果把这些状态都合并成一个布尔值 `cancelled`，就很难回答用户的新输入是否会丢失、会否在恢复后再次运行。这里描述的是仓库测试的覆盖意图，本次没有在本地执行测试，也没有验证外部业务动作能否被撤销。

## 修改前应保留的几个边界

调整 `preStep` 或工具组装时，核对准备失败是否清理资源、是否发生模型调用、是否留下明确的 turn/step 记录；调整取消时，分别覆盖空闲、排队、准备中、运行中和收尾阶段。外部副作用仍需在下游系统核实。
