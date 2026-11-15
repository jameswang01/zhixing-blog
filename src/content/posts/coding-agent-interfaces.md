---
title: "接入与可观测性：怎样追踪一次 Agent 任务"
description: "区分接入方式、实时事件和可回放的记录。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 先问使用者需要什么

终端界面适合交互调试，SDK 适合嵌入应用，服务接口便于远程连接；它们都是接入方式，并不自动定义相同的权限、持久化或故障恢复能力。

## 从源码找到可观察的边界

Pi 的循环在模型与工具推进过程中产生事件，宿主可以基于这些事件更新界面或记录过程；具体事件字段以对应代码版本为准。[Pi 循环](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts)

DeepSeek 文档区分可持久化的 session 事件与运行中的事件，二者服务于不同目的。实时界面看到的信息不应直接等同于已经落盘、可恢复的记录。[DeepSeek 事件说明](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)

Codex 的工具路由与执行编排为追查工具调用提供代码入口，但是否记录完整参数、怎样脱敏以及保存多久，应以具体产品和环境为准。[Codex 路由](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs) · [执行编排](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs)

Claude Code 的使用行为和扩展入口可以从官方文档了解；内部遥测实现未在本次资料中验证。[官方工作方式](https://code.claude.com/docs/en/how-claude-code-works)

## 设计一条可排障记录

至少关联任务、轮次、工具调用与尝试次数，并保存必要的状态、耗时、错误类别和版本信息。对用户输入、工具参数和返回值应先确定是否需要记录，再考虑脱敏、权限及保留时间。

例如一个结果错误的任务，可以依次核对用户输入、解析出的参数、工具返回的数据范围、最终回答引用。HTTP 请求成功只是其中一个信号，不足以证明结果正确。

## 验证建议

在安全的测试环境注入超时、空结果和取消，检查事件顺序是否清楚、界面显示与后端状态是否一致。本文没有运行这些产品的统一观测实验。


## 一次任务跨过哪些边界

```text
用户界面 / SDK → 会话与任务入口 → 模型请求 → 工具执行 → 外部服务
                         ↑             ↓
                     状态与事件 ← 结果和错误
```

这是通用分析图，不代表任何一个产品的实际部署图。UI 能显示进度，不等于后端一定保存该进度；接口返回请求已接收，也不等于外部动作完成。

## 从三个层次设计可观测性

| 层次 | 关注什么 | 一个例子 |
| --- | --- | --- |
| 任务 | 目标、状态、总耗时 | 用户取消后任务是否仍在运行 |
| 模型与工具 | 每轮调用、参数摘要、耗时、错误 | 选错工具还是工具本身失败 |
| 外部业务 | 业务请求号、下游状态、最终效果 | 提交申请超时后是否实际成功 |

应该通过合适的关联标识把这些层次连接起来，同时限制敏感数据进入日志。标识如何产生、跨服务如何透传，取决于实际系统。

## 从源码入口看可观察过程

Pi 的[流式响应处理](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L390-L460)会在消息生成过程中发出相应事件；界面可以消费增量，而最终完成消息和早期片段不能混为一谈。

DeepSeek 的[架构说明](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md#L67-L78)明确区分持久会话事件和运行时事件。某次 UI 出现更新，只能说明观察到了相应过程；是否已写入持久记录需要查对应存储和提交边界。

Codex 的[工具路由](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs)与[执行编排](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs)可作为追踪一次工具动作的代码入口，但这不意味着所有日志在每个部署环境都可查询，也不说明参数一定完整保留。

## 场景：用户点了停止

先区分四种状态：界面停止接收、服务端收到取消、正在运行的调用实际停止、外部业务动作是否已经发生。若只显示“已停止”，用户可能误以为前面所有操作都未发生。更好的产品提示应基于实际可核实的状态，并说明仍在确认的外部动作。

## 可落地的验证

在测试环境分别于模型输出、只读工具调用、写操作提交前后触发取消；检查界面、任务状态、工具记录和下游结果是否一致。还应覆盖断线重连、重复事件和隐私脱敏。以上是测试建议，本次没有运行跨产品的对比实验。


## 同一个任务，至少需要两种时间线

[Pi 并行工具测试](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/test/agent-loop.test.ts#L721-L813)显示：执行结束事件可以按完成时间产生，而结果消息按原调用顺序排列。因此一个排障界面若只按时间展示事件，会与模型后续看到的结果顺序不同。比较时应保留调用 ID 和批次顺序。

[DeepSeek 测试](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/tests/tool-order.spec.ts#L73-L100)对照会话请求头和模拟适配器接收的工具列表，示范了如何同时检查“记录了什么”和“实际发出了什么”。这些都是特定测试场景下的断言，不代表每个产品都提供相同的可观测接口。
