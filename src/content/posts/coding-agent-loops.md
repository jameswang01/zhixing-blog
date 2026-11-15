---
title: "Agent Loop 与工具执行：从三份源码看控制流"
description: "比较模型调用、工具派发与循环退出的工程边界。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 先确定比较范围

这里对照 Pi、Codex 和 DeepSeek Harness 的公开代码，Claude Code 只采用官方行为文档。它们的代码基线分别是 `4f95fbe`、`ee344ca`、`39163dd`；不同模块的名称与能力并非一一对应。

## 相同问题：模型提出动作之后怎么办

| 环节 | 需要回答的问题 |
| --- | --- |
| 模型请求 | 当前请求带哪些消息和工具定义？ |
| 结果解析 | 响应是普通文本、工具调用、错误还是被截断？ |
| 工具执行 | 参数是否可用，动作是否允许，并行是否安全？ |
| 结果回填 | 工具输出如何变成下一步可用的上下文？ |
| 停止与恢复 | 用户取消、预算耗尽、错误或完成如何表达？ |

## 三种可核对的代码入口

- Pi 的 `runLoop`、`executeToolCalls` 展现消息推进及工具批次分支，包含顺序和并行路径。[Pi 循环](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts)
- Codex 的 `ToolRouter` 与 `ToolRegistry` 将工具路由、注册和并行能力查询分开；具体循环还需继续沿调用链阅读。[路由](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs) · [注册](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/registry.rs)
- DeepSeek Harness 的 Agent 循环包含 turn、step 与工具集合解析等步骤，并与会话事件协作。[循环代码](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts) · [架构说明](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)
- Claude Code 官方资料描述收集上下文、行动和验证的反复过程，但这不足以证明其内部函数调用顺序。[官方工作方式](https://code.claude.com/docs/en/how-claude-code-works)

## 如何用于面试

画一个最小执行图，再补三个失败路径：模型参数不完整、工具请求超时、用户中途取消。每条路径说明谁发现问题、状态记录在哪里、是否允许重试，以及用户看到什么。对外部写入，恢复前还要核实副作用是否已经发生。

## 未验证范围

这里没有做跨产品基准测试，也不据此比较成功率、速度或成本。实际行为还取决于模型、工具、版本与配置。
