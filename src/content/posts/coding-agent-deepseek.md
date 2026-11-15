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
